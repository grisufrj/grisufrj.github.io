---
title: Tutorial de Game Hacking utilizando Cheat Engine 
date: 2024-08-12
description: "Aprender como utilizar o Cheat Engine para encontrar e manipular endereços de memória de interesse, injeção/remoção de código para mudar o funcionamento do jogo"
tags: [Game Hacking, Engenharia Reversa]
categories: [easy]
author: asw4ng
published: 2024-08-12
---

Abordaremos neste artigo sobre os princípios fundamentais sobre Game Hacking, ensinando como utilizar o Cheat Engine, um programa clássico para mudar aspectos de um jogo a seu favor. 

### Sobre Processos
Antes de tudo, é bom deixar bem definido sobre o que é um jogo para o computador. Ao abrirmos qualquer programa executável, o sistema operacional cria um novo processo para rodar este programa. Um **Processo** é, portanto, uma tarefa em execução no computador. Podem ter múltiplos processos rodando ao mesmo tempo, sendo em janelas ou em plano de fundo, e pode se criar vários processos de um mesmo programa. 

### Sobre Cheat Engine
O Cheat Engine é um Memory Scanner. Dado um processo, ele busca na memória RAM que este processo utiliza por determinados valores ou padrões em tempo real. Com isso podemos procurar por endereços de memória interessantes, como a posição do jogador ou a vida de um inimigo. Além disso, o Cheat Engine também possui funcionalidades de Debugger e Auto Assembler, muito úteis para análise e modificação da lógica do jogo.

Neste post, vamos resolver como exemplo o tutorial que vem junto ao baixar o Game Engine, e desta forma mostrando na prática como usar as ferramentas para hackear.

## Solucionando o Tutorial Padrão (Tutorial-x86_64.exe)
Começaremos solucionando todas as 9 etapas do tutorial padrão do Cheat Engine. Este tutorial pode ser aberto tanto pelo executável, que se encontra na pasta do executável do Cheat Engine, quanto no menu help dele aberto:

![Figura](/images/CheatEngine_Tutorial/Pasta.jpg) ![Figura 2](/images/CheatEngine_Tutorial/Help.jpg)
### Etapa 1: Anexando um Processo 

![Figura](/images/CheatEngine_Tutorial/Step1.jpg)

Depois de abrir o tutorial, temos que comunicar ao Cheat Engine qual processo ele deve analisar. Para anexar o processo do tutorial, basta clicar no ícone de busca abaixo, ir na aba de processos e escolher aquele com o nome "Tutorial-x86_64.exe":

![Figura](/images/CheatEngine_Tutorial/Welcome.jpg)

Em geral, podem existir múltiplos processos com o mesmo nome. Isso ocorre com frequência em browsers (cada aba é um processo distinto), mas podem haver processos que rodam em plano de fundo que abrem junto com o processo principal. O que distingue cada um é pelo PID, o número ao lado do nome na tela acima. 

No caso deste tutorial, somente existe um processo com este nome, portanto podemos selecionar ele e seguir em frente.

### Etapa 2: Escaneando por Valores Conhecidos

![Figura](/images/CheatEngine_Tutorial/Step2.jpg)

Nesta Etapa, desejamos mudar o valor da vida que aparece na tela. Para isso, temos que encontrar aonde na memória o valor está sendo guardado. Temos o valor inicial exato da vida, 100. O tutorial sugere que o tipo da variável é de 4-bytes, mas em casos gerais é bem possível não sabermos exatamente o tipo, então vou progredir sem essa informação.

Com isso, os campos para o primeiro escaneamento ficam:

	Value: 100
	Scan Type: Exact Value
	Value Type: All ou 4-bytes
	
Realizando então o primeiro escaneamento (clickando no botão 'First Scan' acima do campo *Value*), encontramos vários endereços de memória com o valor 100:

![Figura](/images/CheatEngine_Tutorial/Step2.1.jpg)

Para distinguir entre estes inúmeros valores, temos que ver como cada um se comporta e ver diferenças. Vamos utilizar escaneamentos em sequência usando informações que sabemos da vida. Temos no tutorial um botão para reduzir a vida. No caso, foi reduzido 3 pontos de vida, resultando em 97 de vida total. Temos estas duas informações para filtrar a lista de endereços, sabemos que se algum endereço não mudou desta forma não pode ser a variável que queremos.

No caso, usamos a informação da vida total e usamos outro escaneamento do tipo 'Exact Value' e clickando em *Next Scan*: 

![Figura](/images/CheatEngine_Tutorial/Step2.2.jpg)

Somente um sobrou! Muitas vezes não conseguimos filtrar totalmente os endereços de uma vez, para estes casos devemos realizar vários escaneamentos para reduzir a um número aceitável de candidatos. 

Clickando duas vezes neste endereço irá mandar ele para a lista de endereços guardados abaixo. Clique duas vezes no valor e o mude para 1000. O botão *Next* do tutorial deve liberar.

> *O texto que se vê e seu valor real são duas variáveis distintas, de tipos diferentes. O texto pode atualizar com o novo valor ou não ao modificar o valor para 1000. Para verificar, force um update do valor pelo programa original, neste caso pelo botão 'Hit me' e veja se o valor está levemente menor que o valor que você botou.*

### Etapa 3: Escaneando por Valores Desconhecidos

![Figura](/images/CheatEngine_Tutorial/Step3.jpg)

Para esta etapa, temos que mudar o valor de uma variável que não sabemos o seu valor. As únicas informações que temos dela são de que seu valor está entre 0 e 500 e que seu valor decresce quando apertado o botão *Hit me*. O programa também mostra por 1 segundo o quanto decresceu ao apertar o botão.

Para começar, podemos realizar o primeiro escaneamento de duas formas, uma geral e outra se utilizando das informações dadas:

Geral:

	Value: ---
	Scan Type: Unknown Value
	Value Type: All

Entre 0 e 500:

	Value: 0 and 500
	Scan Type: Value between...
	Value Type: All

Para escaneamentos seguintes, utilizamos *Decreased Value/Decreased Value by...* se apertado o botão ou *Unchanged Value* se feito nada. Achado o endereço correto, mude seu valor para 5000.

### Etapa 4: Floats e Doubles

![Figura](/images/CheatEngine_Tutorial/Step4.jpg)

Temos agora que achar dois valores conhecidos, sabendo que eles são dos tipos float e double.

### Sobre Números em Ponto Flutuante
Ponto Flutuante é uma maneira de se representar números racionais digitalmente, capaz de suportar números decimais e números muito grandes. Estes seguem o formato da notação científica, como $$ 12345 = 1,2345 \times 10^{4}. $$ Na variável em si, sendo float (4-bytes) ou double (8-bytes), é guardado 3 informações: 1 bit de sinal (S), a mantissa (M) e o expoente (E), de forma que o valor seja dado da forma:

$$
valor = (-1)^S\times M\times 2^E 
$$

Configuração de uma variável float (4-bytes):

![Figura](/images/CheatEngine_Tutorial/Step4.1.jpg)


Para achar estes valores, deve-se seguir o passo-a-passo da Etapa 2, com o campo *Value Type* correspondente.

### Etapa 5: Achando e Removendo Instruções

![Figura](/images/CheatEngine_Tutorial/Step5.jpg)

Algo que não foi comentado até agora é que o local onde uma variável é guardada pode mudar ao reiniciar o programa ou até mesmo durante sua execução, fazendo com que tenha que se repetir o processo de procura. Existem duas formas de fazer mudanças no funcionamento do programa que possam ser repetidos caso isto aconteça: mudanças no código e ponteiros com base um endereço estático. Nesta etapa, vamos procurar e modificar uma instrução que reduz o valor desejado. 

Após encontrar o endereço do valor, desejamos encontrar o código que muda o seu valor. Para tal, clique no endereço com o botão direito do mouse e depois na opção '*Find out what writes to this address*'

![Figura](/images/CheatEngine_Tutorial/Step5.1.jpg)

Isso vai anexar o Debugger ao processo sendo analisado. Toda vez que alguma instrução acessar a variável ela será listada nesta janela, portanto para achar a instrução que muda o valor só precisamos apertar novamente o botão *Change Value*. Se tudo estiver correto, irá aparecer uma ou mais instruções como ilustrado a seguir:

![Figura](/images/CheatEngine_Tutorial/Step5.2.jpg)

Como só precisamos impedir do botão de mudar o valor, podemos só substituir esta instrução por NOP (No Operation). A forma mais simples é pelo botão *Replace*. Selecione a instrução desejada, clique no *Replace* e confirme a ação. Pare o Debugger clicando em *Stop* no canto inferior direito e veja se o valor não mude ao clicar em *Change Value*.

### Etapa 6: Ponteiros

![Figura](/images/CheatEngine_Tutorial/Step6.jpg)

Na programação, muitas vezes é utilizado variáveis que guardam endereços de outras variáveis. Estas variáveis são chamadas de **Ponteiros**. Podemos acessar o valor da variável a qual temos o endereço em um ponteiro, acessando de forma indireta. Para achar um ponteiro para uma certa variável, primeiro temos que descobrir o endereço desta. Como temos acesso ao valor exato, podemos seguir o passo-a-passo da Etapa 2.

Com o endereço correto em mão, clique com o botão direito nele e selecione a opção *Find out what access this address* para abrir o Debugger:

![Figura](/images/CheatEngine_Tutorial/Step6.0.jpg)

Isso significa que as próximas vezes que uma instrução tente acessar este endereço será listado. Mudando o valor da variável, temos estas instruções que aparece listadadas:

![Figura](/images/CheatEngine_Tutorial/Step6.1.jpg)

Veja que todas as instruções possuem um termo entre colchetes. Em assembly, ter um registrador como rax significa uma operação sobre ele, mas ter um termo [rax] a instrução opera sobre o endereço com o valor atual deste registrador. Como podemos observar na imagem abaixo, **depois da execução** da segunda instrução o registrador rdx tem o endereço em Hexadecimal da variável desejada. Poderia ter na instrução um termo do tipo:

```s
	mov [rcx+8], eax
```

Neste caso, significa que tem um offset de 8 sendo adicionado ao conteúdo do ponteiro, que devemos levar em conta para achá-lo. No caso do tutorial, o offset é zero, então podemos usar diretamente o endereço da variável. Fazendo um escaneamento de valor exato só que com valor em Hexadecimal (só marcar o quadrado *Hex* ao lado do campo *Value*):

![Figura](/images/CheatEngine_Tutorial/Step6.2.jpg)

Achamos nosso ponteiro! Mas o Cheat Engine está tratando ele como uma variável de 4-bytes e não como um ponteiro, então devemos consertar isso. Existem duas formas: mudar o tipo do endereço salvo na lista ou adicionar manualmente. Foi escolhido para este tutorial adicionar manualmente. Copie o endereço do ponteiro (No caso temos um endereço estático 'Tutorial-x86_64.exe+325AD0') e clique no botão *Add Address Manualy*, no canto inferior direito acima da lista de endereços salvos. Na nova janela, cole o endereço e selecione a opção *Pointer*. Se tudo ocorrer corretamente, deve estar mostrando o valor atual da variável ao lado do campo Address, como na imagem a seguir:

![Figura](/images/CheatEngine_Tutorial/Step6.3.jpg)

Salve e veja que conseguimos modificar o valor diretamente pelo ponteiro. Para finalizar, mude o valor para 5000, congele o ponteiro selecionando a caixa *Active*  como a imagem abaixo e clique no botão *Change Pointer* do tutorial.

![Figura](/images/CheatEngine_Tutorial/Step6.4.jpg)

### Etapa 7: Injeção de Código

![Figura](/images/CheatEngine_Tutorial/Step7.jpg)

Injeção de código é uma técnica de forçar o programa a executar seu código em vez do original, geralmente feito substituindo uma instrução por uma instrução *jmp* para parte de seu código. Neste caso, desejamos que uma parte do código que retira 1 de vida ao apertar o botão aumente 2 de vida ao invés disso. O primeiro passo é encontrar a instrução que faz esta redução.

Após encontrar o endereço da variável pelo escaneamento *Exact Value*, clique na opção *Find out what writes to this Address* e clique novamente no botão *Hit me*. Com isso achamos a instrução:

![Figura](/images/CheatEngine_Tutorial/Step7.1.jpg)

Com a instrução selecionada, clique em *Show disassembler* e, mantendo a instrução selecionada nesta janela, precione as teclas Ctrl+A para abrir o Auto-Assembler. Utilize o template de Injeção de Código no Auto-Assembler:

![Figura](/images/CheatEngine_Tutorial/Step7.2.jpg)

Temos o seguinte código na janela agora:

````s
alloc(newmem,2048,"Tutorial-x86_64.exe"+2DB57) 
label(returnhere)
label(originalcode)
label(exit)

newmem: //this is allocated memory, you have read,write,execute access
//place your code here

originalcode:
sub dword ptr [rsi+000007E0],01

exit:
jmp returnhere

"Tutorial-x86_64.exe"+2DB57:
jmp newmem
nop 2
returnhere:
````

O fluxo do código segue: 
	
	"Tutorial-x86_64.exe"+2DB57 (Posição original do código anterior) -> newmem -> originalcode -> exit.
	
Chamo a atenção para o fato de que o código original ainda será executado se nada fizermos. Podemos somar 3 para que +3 -1 = +2, mas também podemos só pular a execução do código original:

```s
newmem: //this is allocated memory, you have read,write,execute access
//place your code here
add dword ptr [rsi+000007E0],02
jmp exit

originalcode:
sub dword ptr [rsi+000007E0],01

exit:
jmp returnhere
```

Agora só falta aplicar a injeção. Clique no botão *Execute* e teste se funcionou apertando o botão *Hit me*.

### Etapa 8: Ponteiros para Ponteiros para Ponteiros para...

![Figura](/images/CheatEngine_Tutorial/Step8.jpg)

Nada impede que o endereço apontado por um Ponteiro seja de outro Ponteiro. Na verdade, isso é bem comum. Para chegarmos à base da sequência, teremos que repetir o passo-a-passo da etapa 6 até não conseguirmos mais, possivelmente chegando em um endereço estático.

Depois de achar o endereço do valor pelo escaneamento de valor exato, analizamos quais instruções acessam tal valor:

![Figura](/images/CheatEngine_Tutorial/Step8.1.jpg)

Diferentemente da etapa 6, vemos agora que a instrução acessa nosso valor com um offset de 18 (hexadecimal) no endereço do ponteiro. Portanto, para achar o endereço do ponteiro, temos entender que o valor no ponteiro vai ser 18 menos que o endereço da variável, ou o valor em rsi (**LEMBRANDO:** os valores vistos dentro dos registradores são os DEPOIS da instrução ser executada. Se a instrução modificar o registrador de interesse, o valor dentro NÃO é confiável).

Procurando pela memória o valor contido no ponteiro, encontramos seu endereço:

![Figura](/images/CheatEngine_Tutorial/Step8.2.jpg)

Vamos adicionar o ponteiro na lista de endereços salvos, pelo método manual. Copiando o endereço do ponteiro e marcando a caixa *Pointer*, temos que adicionar o offset da instrução. Se tudo estiver correto, o valor ao lado do campo *Address* deve ser igual ao valor da variável:

![Figura](/images/CheatEngine_Tutorial/Step8.3.jpg)

Salvando este ponteiro, deve ficar da forma apresentada na imagem abaixo. Se modificar pelo ponteiro, deve modificar a variável também.

![Figura](/images/CheatEngine_Tutorial/Step8.4.jpg)

O tutorial nos dá a informação de que a cadeia de ponteiros tem 4 ponteiros, então temos que repetir mais 3 vezes o passo-a-passo. Para não ficar repetindo, será mostrado as janelas de adicionar endereço para cada um dos ponteiros seguintes:

![Figura](/images/CheatEngine_Tutorial/Step8.5.jpg)

Veja que os offsets de cada etapa são diferentes. Aperte *Add Offset* para adicionar mais campos de offset, informando que este ponteiro aponta para mais um ponteiro, a fim de chegar no valor desejado.

![Figura](/images/CheatEngine_Tutorial/Step8.6.jpg)

Vemos a seguir que o endereço do quarto ponteiro está em verde e tem outro formato. Significa que este endereço é estático e permanecerá igual mesmo reiniciando o programa. 

![Figura](/images/CheatEngine_Tutorial/Step8.7.jpg)

Esta é o último endereço que precisamos salvar. Se tudo estiver certo, ele ainda deve apontar para o valor desejado. Se sim, mude-o para 5000, congele o ponteiro e clique em *Change Pointer*.

![Figura](/images/CheatEngine_Tutorial/Step8.8.jpg)

### Etapa 9: Código compartilhado e Structures

![Figura](/images/CheatEngine_Tutorial/Step9.jpg)

Estamos chegando no fim deste tutorial. Falamos na etapa 5 e 7 sobre manipular código, mas uma instrução pode mudar vários endereços além do desejado. Por exemplo, uma mesma função pode ser chamada para reduzir a vida tando do jogador quanto para inimigos. Logo, se retirarmos esta funcionalidade os inimigos se tornam imortais! 

O que queremos, então, é uma forma de direfenciar quando a instrução trata da vida dos aliados e quando trata dos inimigos. É esperado que as variáveis de cada jogador sigam um molde na memória, aliados ou inimigos, que é chamado de estruturas. Nesta etapa, vamos analisar a existência e conteúdos das estruturas de cada jogador, a fim de analisar as diferenças entre times.

Começamos achando o endereço da vida de Dave, por escaneamento de valor exato (tipo Float). Descobrindo quais instruções modificam este endereço, descobrimos que existe um ponteiro que com offset 8 acessa a vida de Dave. Para achar a vida dos outros jogadores, vamos tomar proveito que este código é compartilhado e achar todos os endereços que esta intrução acessa:

![Figura](/images/CheatEngine_Tutorial/Step9.1.jpg)

Reduza a vida de todos os jogadores. Os endereços listados serão as vidas de todos os jogadores, na ordem que a vida foi reduzida:

![Figura](/images/CheatEngine_Tutorial/Step9.2.jpg)

Adicione todos estes endereços na lista de endereços salvos e nomeie cada um para diferenciar. Agora vamos analizar a área da memória da estrutura. Sabemos o endereço base das estruturas, sendo os endereços das vidas menos o Offset, 8 em hexadecimal. Clique em *Memory View*, no lado esquerdo acima da lista de endereços salvos. Na nova janela, vá em *Tools->Dissect data/structures*:

![Figura](/images/CheatEngine_Tutorial/Step9.3.jpg)

Antes de determinar a estrutura, recomendo separar os esdereços base em dois grupos de dois endereços, para facilitar a comparação. Clique em *File->Add extra address* para adicionar novos endereços em um grupo e *File->Add new group* para adicionar um novo grupo.

![Figura](/images/CheatEngine_Tutorial/Step9.4.jpg)

Clique em *Structures->Define new structure* e, com o campo *Guess Field Types* marcado, aperte Ok. O Cheat Engine vai fazer o possível para entender qual o tipo de cada endereço de memória, ele pode errar. Por exemplo, o Cheat Engine nos deu esta estrutura:

![Figura](/images/CheatEngine_Tutorial/Step9.5.1.jpg)

Vemos alguns endereços que sabemos de cara o que é. No offset 08, temos a vida, como esperado. Além disso, temos os nomes de cada jogador no offset 19. Podemos até supor que o offset 18 tem relação com o tamanho dos nomes, já que bate com os quatro casos. Mas não tem nada muito óbvio que nos diz qual "Time" cada jogador faz parte.

Vemos no offset 58 um ponteiro, que abrindo vemos que aponta para a estrutura do aliado. Temos um loop de ponteiros, então. Por curiosidade, fui abrindo cada ponteiro deste offset e vemos algo interessante:

![Figura](/images/CheatEngine_Tutorial/Step9.5.2.jpg)

Uma nova interpretação da estrutura! E com um offset 14 que para aliados é 1 e para inimigos 2. É bem possível que você tenha recebido esta versão ao fazer em casa, mas isso exempla o fato de que o Cheat Engine pode errar ao assumir tipos à endereços, e deve ser levado em conta. É a mesma estrutura, podemos ver este valor na primeira interpretação no ponteiro no offset 10, no formato P->1XXXXXXX para aliados e P->2XXXXXXX para inimigos.

Irei tomar a segunda interpretação como a mais válida. No final, a estrutura fica desta forma:

![Figura](/images/CheatEngine_Tutorial/Step9.5.jpg)

Com um offset que determina times, agora só precisamos fazer uma injeção de código que compara este offset para determinar se é para reduzir a vida ou não. Novamente descubra qual instrução escreve em uma das vidas e, com ela selecionada, aperte *Show Dissasembler*. Com a instrução ainda selecionada na nova janela, pressione Ctrl+A para abrir o Auto-Assemble, use o template de Code Injection e escreva o código a seguir:

![Figura](/images/CheatEngine_Tutorial/Step9.6.jpg)

A etapa está feita! Teste se o código está fazendo efeito e clique em *Restart game and autoplay*.

Com isso, terminamos o tutorial.

## Fechamento

Agradeço muito pela sua atenção e espero que lhe tenha inspirado a se aprofundar mais na área de Game Hacking e Engenharia Reversa. Recomendo ver mais posts no Blog do GRIS sobre as áreas, eles devem te esninar ainda mais sobre, só clicar nas Tags.

> Vale a pena lembrar que foi ensinado o básico de Cheat Engine, para jogos **SEM ANTI-CHEAT**. Seja responsável e use estas ferramentas por sua conta e risco.

## Referências
	
	https://wiki.cheatengine.org/index.php?title=Tutorials:Cheat_Engine_Tutorial_Guide_x64#Pointer_Scan
	
	https://wiki.cheatengine.org/index.php?title=Tutorials:Videos
	
	https://www.youtube.com/watch?v=Nib69uZJCaA&t=31s
	
	https://www.youtube.com/watch?v=yjdSxL2DWfE
