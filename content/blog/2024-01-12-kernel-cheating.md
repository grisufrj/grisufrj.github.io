---
title: "Entendendo o Papel do Kernel na Segurança de Jogos Online"
date: 2024-01-12
published: 2024-01-12
description: "Uma abstração em alto nível do funcionamento de um cheat kernel level utilizando IOCTL"
images: ["/images/kernel-1.png"]
author: ruhptura
categories:
  - "Easy"
tags:
  - "Game Hacking"
---

# O que é o kernel

Quando falamos em sistemas operacionais, em geral pensamos em recursos visuais e interativos como suas aplicações e interface gráfica. No entanto, além de fornecer a interface com o usuário, o sistema também precisa se comunicar com a parte física do computador: o *hardware*. Por conta disso, em geral definimos duas categorias de atuação de um sistema operacional:

- User level
- Kernel level

A primeira delas, que seria o nível de usuário ou espaço de usuário, consiste em todo o espaço reservado para executar aplicações comuns instaladas em um sistema, como o navegador, o editor de textos ou até mesmo binários em linha de comando. Quando esses programas são executados, o kernel aloca os recursos necessários para as mesmas, criando uma espécie de *sandbox*, com uma quantidade qualquer de memória virtual.

Já o kernel em si, atua como um nível intermediário entre o *user space* e o *hardware*, recebendo interações geradas no nível de usuário e gerenciando recursos da memória, do processador e de periféricos diretamente. Os “programas” que rodam aqui são chamados de *drivers*, que precisam funcionar perfeitamente, pois qualquer erro nesse nível pode causar o *crash* do sistema operacional, ocasionando a tela azul da morte no caso do Windows.

A comunicação entre os níveis é feita através das chamadas *syscalls*.

![Figura 1](/images/kernel-1.png)

## Importância para cheats e anti-cheats

Entendendo mais agora sobre o kernel, podemos questionar o porque ele é importante quando pensamos em segurança de jogos. De maneira simples, as ações realizadas em *kernel space* possuem um poder de controle maior, visto que é possível se comunicar com os recursos do computador diretamente, sem a necessidade de pedir permissão ao sistema operacional.

Por conta disso, muitos *cheats* e *anti-cheats* se propõem a atuar no kernel, para adquirir um controle quase que absoluto de manipulação, principalmente de memória, e assim atingir seus respectivos objetivos.

Pense por exemplo em um *cheat* externo que precise chamar funções como **ReadProcessMemory** e **WriteProcessMemory**. Um *anticheat* com permissões de kernel pode simplesmente fazer um *hook* dessas funções ao ser iniciado com o sistema operacional, e assim toda vez que elas forem chamadas o *software* defensivo terá conhecimento, podendo até mesmo bloquear ações suspeitas.

Esse foi apenas um exemplo do que é possível fazer, mas existem diversas outras técnicas de detecção e anti-detecção que só são possíveis de serem executadas com a permissão de *kernel level*

# IOCTL

Muito bem, tendo em mente a motivação de contruir *cheats* em *kernel mode*, vamos entender agora em alto nível como eles são feitos. O foco aqui será entender o método de comunicação por IOCTL.

## Driver

Primeiro de tudo, precisamos do driver que atuará de fato no nível de kernel. Esse driver precisa de três mecanismos principais:

- Operações básicas de memória como ler e escrever
- Criar um hook em alguma função do sistema para servir de entrada de dados pelo *user mode*
- Precisa ter uma assinatura válida

Para o primeiro objetivo, precisamos usar estruturas de dados e funções não documentadas do Windows, sendo uma delas a **MmCopyVirtualMemory**, que possui o papel de copiar *bytes* para uma região de memória qualquer de um processo.

Para criar o hook em uma função de outro *driver* (por exemplo o dxgkrnl.sys), simplesmente criamos um *payload* com a instrução **jmp** para um endereço controlado por nós, e sobreescrevemos diretamente esse *payload* na memória do *driver*, visto que temos permissão de kernel.

Para conseguirmos executar o nosso *driver*, ele precisa ter uma assinatura válida reconhecida pelo sistema, no entanto, não temos uma maneira fácil de conseguir isso. Em geral o que é feito é a exploração de outros drivers vulneráveis do sistema, para mapear o conteúdo do nosso driver em *kernel space*. Uma ferramenta muito conhecida para fazer esse *exploit* é o [KDMapper](https://github.com/TheCruZ/kdmapper).

## Aplicação

Após o mapeamento das funções desejadas do nosso driver, precisamos também de uma aplicação em *user mode* capaz de interagir com essas funcionalidades criadas em kernel. Por esse motivo que é criado um *hook handler* através do nosso *driver*. Esse “handler” funciona como um receptor de instruções, que podemos enviar através da nossa aplicação, de tal maneira a ordenar manipulação de memória em kernel.

O nome [IOCTL](https://learn.microsoft.com/pt-br/windows/win32/devio/device-input-and-output-control-ioctl-) vem justamente dessa funcionalidade, pois é a partir desse *hook* que implementamos uma interface de entrada e saída de comunicação com o *driver*. A partir daí, lógicas específicas de cada *cheat* e cada *game* podem ser implementadas pelo próprio *user mode*.

![Figura 2](/images/kernel-2.png)