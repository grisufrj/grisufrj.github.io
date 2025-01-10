---
title: "pwn 101: Desafios de Introdução a Buffer Overflow"
date: 2024-01-12
published: 2024-01-12
description: "Introdução a exploit de binários"
categories:
  - "Easy"
tags:
  - "C"
  - "CTF"
  - "Pwn"
images: ["/images/pwningimage.jpg"]
author: 0xmr
---

Essa postagem tem como objetivo iniciar a jornada no mundo de pwning de maneira didática, ao demonstrar um 
Buffer Over Flow, um exploit moderno que obtem vantagem sobre a Stack e buffers de memória.


# Conceitos Básicos de Programas

Antes de tudo, é importante ter em mente alguns conceitos importantes antes de começar a jornada em pwning. Certos conceitos de low-level e como os programas são compostos vão ser extremamente importantes para entender 

Todo programa em baixo nível possui **Heap**, **Registradores**, a **Stack** e instruções em assembly. Segue abaixo uma descrição do que cada um significa.

**Heap:** Lugar onde a memória é alocada dinamicamente, através de funções

como malloc(), realloc(), calloc()

**Registradores:** São locais que servem como armazenamento temporário para sequências de bits, que estão dentro da CPU do computador. Possuem x bytes de tamanho dependendo da arquitetura. Eles servem como receptáculo de valores e endereços para realizarem operações.

**Stack:** Lugar na memória onde as variáveis declaradas no programa são armazenadas temporariamente em uma estrutura de pilha LIFO (Last in First out).

Existem apenas duas instruções em uma pilha, a de push (que bota alguma coisa dentro da stack) e da pop (que tira algo da stack) . Toda função tem sua própria stack, onde suas variáveis ficam armazenadas. A stack consiste na região entre o Base Pointer (ponteiro que aponta para a base da Stack) e o Stack Pointer (ponteiro que aponta para o topo da stack). Na stack as variáveis que são declaradas primeiro possuem um endereço **MAIOR** do que as variáveis que são declaradas depois.


# Pwntools

Pwntools é um framework em python utilizado para CTF´s e desenvolvimento de exploits. Ele serve para agilizar e facilitar desenvolvimento de exploits.

Aqui iremos usá-lo para facilitar escrever certos inputs em nosso programa.

Abaixo segue alguns conceitos sobre o pwntools e suas funções:

-> from pwn import *  = Linha responsável por importar o pwntools para o programa

p = process('.programa') = Cria uma conexão com o programa/processo a ser executado

p.sendline() = Envia uma linha para o programa 

p.interactive() = Cria uma secção interativa com o terminal

p32(numero) = Transforma um número em bytearray de 32 bits (ótimo para passar um número para **little endian**)

p64(numero) = Transforma um número em bytearray de 64 bits (ótimo para passar um número para **little endian**)


# **[Csaw 2018 Quals Boi](https://guyinatuxedo.github.io/04-bof_variable/csaw18_boi/index.html)**

Neste desafio podemos reparar que estamos lidando com um binário de 64 bits (que basicamente significa que ele foi compilado com uma arquitetura de 64 bits).

!["bof1"](/images/bof1.png)


O programa printa a string **“Are you a big boiiiii??”**, nos dá uma opção de input e nos printa a data e o dia.

Vamos começar dando uma olhada na função main desse programa fazendo uso novamente do **Ghidra:**

!["bof2"](/images/bof2.png)


Podemos notar o seguinte:

- Nosso input é lido com a função read() e sabemos que ele lê 18 bytes hexadecimais e depois os armazena em input.
- Temos um condicional que estabelece: caso a variável inteira target possua o valor de 0x-350c4512 (aparentemente) iremos poppar um shell, a partir do comando run_cmd(/bin/bash), caso contrário iremos apenas obter a data a partir do shell.
- Próximo do comando de return temos um if statement à variável long stackCanary, que é basicamente uma variável que é posta na stack que possui o objetivo de indicar quando houve uma tentativa de Buffer OverFlow.

Ok, antes de começarmos a discorrer sobre como vamos exploitar esse programa e conseguir a flag, vamos clarificar o valor da variável target no if statement.

Ao usar um debugger como o Ghidra, podemos ter acesso ao código de linguagem de montagem (Assembly), que gerou o binário. Clicando na linha de código correspondente:

!["bof3"](/images/bof3.png)


Vemos que o valor que está sendo comparado com o de target na instrução CMP é:

**0xcaf3baee .**

Ok, agora sabemos o valor que a variável target precisa ter para podermos poppar o shell e conseguir a flag para esse problema. Porém, a pergunta que devemos nos fazer é:

**Como vamos mudar o valor da variável target, se o input do programa não altera seu valor, e sim o valor da variável “input”?**

## **Buffer OverFlow**

É aqui que vamos apresentar a técnica de exploit chamada **Stack Buffer OverFlow**, que precisaremos usar para resolver esse challenge.

- **O que exatamente é um buffer?**

R: Um buffer é basicamente um espaço na memória que guarda temporariamente um dado.

- **Por que do OverFlow?**

R: Quando temos um comando de input, o dado que foi inserido será guardado temporariamente em um buffer de tamanho x, porém caso os dados passados possuam um tamanho maior do que x, ele irá subscrever endereços

!["bof4"](/images/bof4.png)


A imagem acima basicamente diz que o Buffer cresce em direção ao **Base Pointer** (Registrador que indica a base da Stack), portanto conseguimos corromper os dados que estão armazenados nos endereços que são **maiores** do que os endereços de onde está nosso input.

Com esse raciocínio em mente, encontramos um jeito de alterar o valor da variável **target** “através” de **input**, visto que o endereço de input é:

(A imagem abaixo mostra o layout da **stack** no Ghidra, é possível acessá-lo clicando em qualquer uma das variáveis onde elas foram declaradas.)

!["bof5"](/images/bof5.png)


Endereço da variável input = **Stack[-0x38]**

Endereço da variável target = **Stack[-0x24]**

Distância entre os dois endereços = **0x14**

O endereço da variável target, é **maior** que o endereço da variável input. Também precisamos lembrar que estamos escrevendo **0x18** bytes de dados, por conta da função **read**.

Portanto, podemos escrever esses 0x14 bytes de diferença e também subscrever 0x4 bytes da variável target. Como números inteiros possuem 4 bytes de tamanho em C, podemos reescrever tranquilamente o valor de target, visto que ele é uma variável do tipo inteiro.

Como queremos que o valor de **target** seja **0xcaf3baee** precisaremos então inputar 0x14 bytes aleatórios + **0xcaf3baee** porém em little endian, pois é como binário irá ler o valor em hexa.

## **O que é little endian?**

Explicando rapidamente, endianness tem a ver com a ordem que os bytes se apresentam

Por exemplo, o número 0x7025

Em big endian ele será = 0x7025 enquanto em little endian será = 0x2570

Os bytes do número se inverteram, lembrando sempre que **UM** dígito em hexadecimal precisa de 4 bits para ser representado, portanto **DOIS** dígitos em hexa precisam de 1 byte (8 bits) para ser representado.

Ok, com tudo isso em mente podemos escrever nosso exploit para conseguir a flag.

O exploit consiste em um script em python usando o pwntools


!["bof6"](/images/bof6.png)


Fazendo uso desse código no programa um shell irá poppar!



!["bof7"](/images/bof7.png)


## Referências:

Desafio realizado: [https://guyinatuxedo.github.io/04-bof_variable/csaw18_boi/index.html](https://guyinatuxedo.github.io/04-bof_variable/csaw18_boi/index.html)

Lista de instruções de Assembly: [https://www.cs.virginia.edu/~evans/cs216/guides/x86.html](https://www.cs.virginia.edu/~evans/cs216/guides/x86.html)

BufferOverFlow: [https://www.rapid7.com/blog/post/2019/02/19/stack-based-buffer-overflow-attacks-what-you-need-to-know/](https://www.rapid7.com/blog/post/2019/02/19/stack-based-buffer-overflow-attacks-what-you-need-to-know/)
