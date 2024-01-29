---
title: "Assault Cube Hack: Parte 1"
date: 2024-01-07
published: true
description: "POC de hack interno para Assault Cube"
categories:
  - "Easy"
tags:
  - "C++"
  - "Game Hacking"
author: jurbas
---

# Análise do Jogo

## Introdução

Este material visa fornecer um olhar detalhado sobre o processo de desenvolvimento de uma POC, _proof of concept_, para cheats internos usando o jogo Assault Cube, um jogo de tiro em primeira pessoa de código aberto, perfeito para o escopo desse material.

> **Nosso objetivo aqui não é encorajar o uso desonesto de cheats, mas sim explorar as complexidades e habilidades técnicas necessárias para criar e entender tais modificações.**

## **Encontrando _Offsets_**

_Offsets_ são endereços na memória onde dados específicos, como saúde, munição, ou posição do jogador, são armazenados. Há dois métodos para encontra-los.

- **Método Estático**: Alguns _offsets_ são estáticos, sempre serão carregados na mesma posição de memória relativa ao endereço base do processo, e podem ser encontrados no código do jogo, com isso podemos fazer uma análise estática com ferramentas como IDA e Ghidra.
- **Método Dinâmico**: Será o método abordado nesse material utilizando _scanners_ de memória como Cheat Engine para encontrar _offsets_ dinâmicos. Para isso teremos que abrir o jogo e realizar sucessivos _scans_ até encontrarmos o o valor que queremos (por exemplo vida, munição etc.).

Primeiro vamos abrir o Assault Cube e o Cheat Engine, selecione o ícone de computador para _attach_ ao processo do jogo. Escolha _AssaultCube.exe_ na lista de processos.

No jogo, observe quantas balas você tem. Por padrão, você deve começar com 20 balas. Volte ao Cheat Engine e no campo _Value_, digite 20.

Clique em _First Scan_. O Cheat Engine listará todos os endereços de memória que atualmente têm o valor 20.

![Untitled](/images/ac-hack-1.png)

Veja quantos valores candidatos temos na lista, 2.349 candidatos, isso é muita coisa vamos filtrar mais. Voltando ao jogo e gaste algumas balas para alterar o valor e vamos scannear novamente e os resultados serão filtrados, mostrando apenas os endereços cujos valores mudaram, para 19 no caso da imagem.

![Untitled](/images/ac-hack-2.png)

Obtivemos um número muito menor de resultados, ou seja, uma desses dois resultados é o valor real da munição, com duplo clique adicionamos cada um dos valores em nossa lista e alteramos o campo _Value_ no Cheat Engine, o resultado que alterar o valor da munição do jogo será o correto.

![Untitled](/images/ac-hack-3.png)

Agora que identificamos o endereço de memória da munição, precisamos entender um conceito crucial em jogos e programas modernos: a alocação dinâmica de memória.

**Por que o endereço muda?**
Quando reabrimos o jogo e repetimos o processo de _scan_ pela munição, notamos que ela está em um novo endereço de memória. Isso ocorre devido à Alocação Dinâmica de Memória.

Os jogos, especialmente os mais complexos, frequentemente gerenciam sua memória de forma dinâmica, reorganizando e realocando dados conforme necessário para otimizar o desempenho e a experiência do jogador.

Para navegar por essa estrutura de memória em constante mudança, utilizamos uma abordagem baseada em **ponteiros**. Estes ponteiros atuam como marcadores, guiando-nos através de várias camadas de dados até chegarmos ao valor desejado.

Por exemplo, ao carregar um jogo, podemos seguir uma cadeia de ponteiros que passa por:

<aside>
📂 Mundo > Entidades > Jogador > Munição

</aside>

Cada passo nesta cadeia nos aproxima do endereço específico onde a quantidade de munição do jogador é armazenada. Ao identificar e seguir esses ponteiros, podemos localizar de forma confiável e eficiente os dados relevantes, mesmo quando seu endereço exato na memória muda a cada sessão do jogo.

Voltando à prática, no Cheat Engine, clicamos com o botão direito e escolha a opção _Find out what writes to this address_ [F6]. Em jogo vamos dar alguns tiros para atualizar o valor da munição e registrar quais instruções modificam o valor da munição.

![Untitled](/images/ac-hack-4.png)

Analisando o _disassembly_, podemos perceber a instrução **`mov esi, [esi + 14]`** aparece imediatamente antes do decremento. Isso sugere que o jogo calcula o endereço da munição ajustando o valor atual de **`esi`** com um offset de **`14`**. O valor em **`esi`**, após este ajuste, aponta para onde a munição é armazenada. A execução de **`dec [esi]`** após o ajuste de endereço significa que **`esi`** agora aponta diretamente para a quantidade de munição, e o jogo está pronto para subtrair uma unidade dessa quantidade.

```nasm
mov [edx],ecx
mov esi,[esi+14]
dec [esi]
push edi
mov edi,[esp+14]

;EAX=004FC9FC
;EBX=00000000
;ECX=00000078
;EDX=00DCA6A0
;ESI=00DCA678 (munição)
;EDI=016CD5D7
;EBP=0019FAEC
;ESP=0019FAB8
;EIP=004637EB
```

Portanto temos até agora:

<aside>
📂 `**DCA664 + 14`** > 120 (Munição)

</aside>

Agora vamos achar o ponteiro para **`DCA664`** e encontraremos o seguinte resultado.

![Untitled](/images/ac-hack-5.png)

E repetimos o processo de Engenharia Reversa até achar o ponteiro estático, resultando no seguinte esquema.

<aside>
📂 Endereço base > Jogador > Arma atual > Ponteiro munição atual > Munição

</aside>

![Untitled](/images/ac-hack-6.png)

<aside>
💡 **Dica**: No Cheat Engine, **ponteiros estáticos** são frequentemente destacados em **verde**.

</aside>

Concluindo, vamos anotar os offsets em um arquivo offsets.h.

```cpp
// Offsets for Assault Cube 1.2.0.2
const unsigned int baseAddr            = 0x400000;
const unsigned int offsetPlayer        = 0x109B74;
const unsigned int offsetCurrWeapon    = 0x378;
const unsigned int offsetAmmunitionPtr = 0x14;
```

## Próximos Passos

Confira a [Parte 2](/blog/2024-01-14-assault-cube-hack-parte-2) da nossa série sobre hacking no Assault Cube.
