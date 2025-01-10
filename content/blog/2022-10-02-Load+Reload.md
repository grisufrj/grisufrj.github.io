---
title: "Load+Reload: Uma prova de conceito de um side-channel que explora a cache associativa de processadores AMD"
date: 2022-10-02
published: 2022-10-02
author: ottoboni
---

# Introdução

Em busca da maior performance possível, os processadores atuais tiram
vantagem de diversos artifícios, muitas vezes sem grande consideração
pela segurança. Nesse contexto, ataques como Spectre (Kocher et al.
2019) e Meltdown (Lipp et al. 2018) mostram que a execução especulativa
é um tópico muito interessante a ser explorado. Nesse artigo,
construiremos uma prova de conceito do ataque Load+Reload (Lipp et al.
2020), que tira vantagem do preditor de *way* presente na cache dos
processadores atuais da AMD. Durante esse processo, veremos uma forma
diferente de medir o tempo de leitura da memória, a thread contadora,
mais precisa do que a instrução `rdtsc`, usada em ataques como
Flush+Reload (Yarom and Falkner 2014).

# Organização interna da cache

A cache de um processador é uma memória que armazena páginas de memória
recentemente acessadas. Por na maior parte das vezes ser composta por
células de memória estáticas (SRAM), a cache é múltiplas vezes mais
rápida que a memória principal de um computador, que usa células
dinâmicas (DRAM).

A menor unidade dentro da cache é chamada de **linha** e geralmente tem
o mesmo tamanho de uma página de memória. Uma certa página, no entanto,
não necessariamente pode ser armazenada em qualquer linha. O grupo de
linhas nas quais uma página pode ser amazenada é chamada de **set**.
Além disso, cada linha de um set onde uma certa página pode ser
armazenada é chamada de **way**.

No caso mais simples, cada set tem apenas 1 way e a cache é considerada
**diretamente mapeada**, pois cada página só pode ser armazenada em
exatamente uma linha. Se, pelo contrário, uma página pode ser armazenada
em *qualquer* linha, a cache é considerada **totalmente associativa**,
como se tivesse apenas 1 grande set. Por fim, se uma página pode ser
armazenada em *n* linhas, a cache é considerada **associativa por
conjunto *n*-way**.

Outro ponto importante é a forma como a cache será indexada e marcada. O
**index** se refere ao endereço utilizado para acessar um certo set,
enquanto que a **marca**[1] se refere a como cada way dentro de um set
será identificado. Nos processadores Zen da AMD, a cache é
**virtualmente indexada e fisicamente marcada (VIPT)**[2](AMD 2019).
Isso significa que a partir do endereço virtual de uma página é possível
encontrar o set onde ela está armazenada na cache. Analogamente, a
partir do endereço físico de uma página é possível encontrar em qual way
do set página está.

# Predição de way

Como explicado no tópico anterior, quando uma página de memória é
requisitada, primeiro o endereço virtual dela é utilizado para
determinar o set da cache onde ela pode estar armazenada. Após
determinar o set, o endereço físico é utilizado para encontrar o way. A
grande vantagem desse método é que o endereço físico pode ser calculado
enquanto o set está sendo determinado.

Na arquitetura Zen da AMD, a cache L1 de dados é associativa por
conjunto 8-way(AMD 2019), ou seja, pode ser necessário verificar a marca
de 8 ways para saber em qual deles uma página pode estar armazenada. A
fim de acelerar esse processo, o processador usa uma *μ*marca para
determinar se a página está ou não em um set(AMD 2019). A *μ*marca é uma
função do endereço virtual e, analogamente à marca normal, fica
armazenada em cada way. Como ela não depende do endereço físico, o
processador pode rapidamente verificar se algum way tem a *μ*marca
correspondente e, se nenhum tiver, imediamente desistir da cache L1 e
começar a procurar a página na cache L2, sem nem mesmo esperar a
tradução do endereço virtual para o físico.

O problema ocorre quando dois endereços virtuais *distintos* referentes
ao *mesmo* endereço físico são acessados em sucessão. Cada acesso vai
alterar a *μ*marca, fazendo com que o próximo acesso caia para a cache
L2, já que o preditor de way não encontrará a *μ*marca que foi
sobrescrita(AMD 2019). Nesse caso, todos os acessos serão feitos na
cache L2, aumentando o tempo de acesso.

# Ataques de cache

Um side-channel se baseia em extrair informação por meio de algum rastro
deixado pela implementação de um sistema. A cache de um processador, por
exemplo, implicitamente informa um processo se uma certa página de
memória foi acessada recentemente.

Se dois processos *A* e *B* compartilham a mesma página de memória, o
processo *B* pode medir o tempo gasto para ler a página e determinar se
a mesma está ou não na cache. Caso a leitura seja rápida, a página
estava na cache; caso contrário, não estava. Dessa forma, se o processo
*B* sabe que a página não estava na cache em um momento *t*<sub>1</sub>
e em um momento posterior *t*<sub>2</sub> for determinado que a página
estava na cache, pode-se concluir que o processo *A* a acessou.

# Medindo o tempo de acesso

## `rdtsc`

No artigo original do ataque Flush+Reload (Yarom and Falkner 2014), a
instrução `rdtsc` é utilizada para pedir o tempo de acesso à memória e
determinar se uma certa página está ou não na cache. Essa instrução
carrega o valor do *time-stamp counter* nos registradores `edx:eax`.
Assim, em conjunto com instruções de ordenação de execução, é possível
calcular o tempo decorrido entre o início e fim da leitura.

```.c
//Função que calcula o tempo de acesso a uma página usando a instrução
uint64_t load_count(uint64_t *addr)
{
	uint64_t volatile time;
	asm volatile (
		"mfence              \n\t"
		"lfence              \n\t"
		"rdtsc               \n\t"
		"lfence              \n\t"
		"movl %%eax, %%ebx   \n\t"
		"movq (%%rcx), %%rcx \n\t"
		"lfence              \n\t"
		"rdtsc               \n\t"
		"subl %%ebx, %%eax   \n\t"
		: "=a" (time)
		: "c" (addr)
		: "rbx"
	);
	return time;
}
```
Função que calcula o tempo de acesso a uma página usando a instrução \texttt{rdtsc}

Um problema ao usar a instrução `rdtsc` é que nos processadores mais
recentes da AMD ela não é mais tão precisa, pois tem seu contador
incrementado a cada 30 ciclos aproximadamente (Lipp et al. 2020). Esse
fato diminui muito a resolução da medição, principalmente em ataques nos
quais a diferença do tempo de acesso é mais sutil.

## A thread contadora


```.c
//Função da thread contadora
uint64_t volatile count = 0;
void *counting_thread(void *args) {
	set_affinity(COUNTING_CORE);
	asm volatile (
		"xorq %%rax, %%rax   \n\t"
		"loop%=:             \n\t"
		"incq %%rax          \n\t"
		"movq %%rax, (%%rbx) \n\t"
		"jmp loop%=          \n\t"
		:
		: "b" (&count)
		: "rax"
	);

	pthread_exit(NULL);
}
```

O artigo ARMageddon (Lipp et al. 2016) mostrou que threads contadoras
podem ter uma resolução tão boa ou melhor do que a instrução `rdtsc` em
processadores ARM. A ideia é ter uma thread cujo único objetivo é
incrementar uma variável global continuamente o mais rápido possível.

A fim de ser o mais otimizada possível, a thread contadora armazena o
valor atual de `count` no registrador `rax` e o copia para a memória
após incrementá-la. Dessa forma, apenas uma escrita à memória é
realizada e o valor atual é acessado exclusivamente pelo registrador.

Um ponto importante a ser notado no código da thread contadora é o uso
das instruções de ordenação `mfence` e `lfence`. Analogamente à medição
do tempo usando a instrução `rdtsc`, é necessário garantir que as duas
leituras à variável `count` aconteçam antes e depois da leitura da
página. Além disso, para o caso da thread contadora, é importante que a
thread principal do atacante obtenha um valor minimamente atualizado de
`count`. A seguir está reproduzido um trecho do manual da Intel sobre a
instrução `mfence`.

> This serializing operation guarantees that every load and store
> instruction that precedes the MFENCE instruction in program order
> becomes globally visible before any load or store instruction that
> follows the MFENCE instruction.
>
> (Intel 2021)

Dessa forma, segundo o manual, os incrementos realizados pela thread
contadora na variável `count` durante a leitura da página se tornarão
visíveis globalmente antes da segunda leitura de `count` que segue a
instrução `mfence`. O uso dessa instrução é fundamental, pois sem ela as
duas leituras de `count` retornariam o mesmo valor na maior parte das
vezes, o que resultaria em um tempo de leitura nulo.

```.c
//Função que calcula o tempo de acesso a uma página usando a thread contadora.
uint64_t load_count(uint64_t *addr)
{
	uint64_t volatile time;
	asm volatile (
		"mfence              \n\t"
		"lfence              \n\t"
		"movq (%%rbx), %%rcx \n\t"
		"lfence              \n\t"
		"movq (%%rax), %%rdx \n\t"
		"lfence              \n\t"
		"mfence              \n\t"
		"movq (%%rbx), %%rax \n\t"
		"subq %%rcx, %%rax   \n\t"
		: "=a" (time)
		: "a" (addr), "b" (&count)
		: "rcx", "rdx"
	);
	return time;
}
```
# Load+Reload

Load+Reload é um dos dois ataques propostos no artigo Take a Way(Lipp et
al. 2020) e tira vantagem do fato que quando dois endereços virtuais
*distintos* referentes ao *mesmo* endereço físico são acessados em
sucessão, a leitura ocorrerá na cache L2. Esse comportamento,
entretanto, só ocorre quando as threads sendo executadas estão no mesmo
núcleo físico(Lipp et al. 2020).

Foram então criados dois programas, uma vítima e um atacante, que serão
executados em núcleos lógicos distintos, mas no mesmo núcleo físico.
Para isso, foi utilizada a biblioteca `pthread` para criar a thread e
prendê-la a um núcleo específico.

A memória que será compartilhada entre a vítima e o atacante é um
arquivo de 8MiB composto de bytes aleatórios mapeados na memória com a
syscall `nmap`. O ataque consiste em uma vítima lendo continuamente
partes do arquivo `data` usando caracteres de uma string `secret`. O
caractere é multiplicado por 4096, de modo que cada byte do arquivo lido
está em uma página distinta.

```.c
//Leitura do arquivo a partir da string secret.
read_byte(&data[secret[i] * 4096]);
```

Enquanto isso, o atacante também lê continuamente o arquivo. Diferente
da vítima, no entanto, o atacante selectiona todos os bytes de `0` até
`0xff` como índice para a leitura do arquivo.

```.c
uint64_t time =
    load_count(&data[byte * 4096]);
```
O tempo levado para a leitura é então medido usando uma thread contadora
e o byte que levou mais tempo para ser lido é escolhido como o que a
vítima estava lendo naquele momento.

A ideia é que se o byte sendo lido pelo atacante for o mesmo que está
sendo lido pela vítima, a leitura do atacante cairá para a L2 por conta
do preditor de way. Essa leitura na L2 levará consideravelmente mais
tempo do que uma leitura na L1, e essa diferença pode ser medida usando
a thread contadora.


[1] *Tag* em inglês.

[2] *Virtually indexed and physically tagged (VIPT)* em inglês.


## Código fonte

<https://github.com/grisufrj/articles/tree/main/load%2Breload>

## Bibliografia

AMD. 2019. *Software Optimization Guide for AMD Family 17h Models 30h
and Greater Processors*.
<https://developer.amd.com/wp-content/resources/56305_SOG_3.00_PUB.pdf>.

Intel. 2021. *Intel® 64 and IA-32 Architectures Software Developer’s
Manual*. <https://cdrdv2.intel.com/v1/dl/getContent/671200>.

Kocher, Paul, Jann Horn, Anders Fogh, Daniel Genkin, Daniel Gruss,
Werner Haas, Mike Hamburg, et al. 2019. “Spectre Attacks: Exploiting
Speculative Execution.” In *2019 IEEE Symposium on Security and Privacy
(SP)*, 1–19. <https://doi.org/10.1109/SP.2019.00002>.

Lipp, Moritz, Daniel Gruss, Raphael Spreitzer, Clémentine Maurice, and
Stefan Mangard. 2016. “ARMageddon: Cache Attacks on Mobile Devices.” In
*25th USENIX Security Symposium (USENIX Security 16)*, 549–64. Austin,
TX: USENIX Association.
<https://www.usenix.org/conference/usenixsecurity16/technical-sessions/presentation/lipp>.

Lipp, Moritz, Vedad Hadžić, Michael Schwarz, Arthur Perais, Clémentine
Maurice, and Daniel Gruss. 2020. “<span class="nocase">Take A Way:
Exploring the Security Implications of AMD’s Cache Way
Predictors</span>.” In *<span class="nocase">15th ACM ASIA Conference on
Computer and Communications Security (ACM ASIACCS 2020)</span>*. Taipei,
Taiwan. <https://doi.org/10.1145/3320269.3384746>.

Lipp, Moritz, Michael Schwarz, Daniel Gruss, Thomas Prescher, Werner
Haas, Anders Fogh, Jann Horn, et al. 2018. “Meltdown: Reading Kernel
Memory from User Space.” In *27th USENIX Security Symposium (USENIX
Security 18)*, 973–90. Baltimore, MD: USENIX Association.
<https://www.usenix.org/conference/usenixsecurity18/presentation/lipp>.

Yarom, Yuval, and Katrina Falkner. 2014. “FLUSH+RELOAD: A High
Resolution, Low Noise, L3 Cache Side-Channel Attack.” In *23rd USENIX
Security Symposium (USENIX Security 14)*, 719–32. San Diego, CA: USENIX
Association.
<https://www.usenix.org/conference/usenixsecurity14/technical-sessions/presentation/yarom>.


