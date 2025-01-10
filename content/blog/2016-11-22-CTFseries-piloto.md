---
layout: post
title: CTF series piloto -- IO netgarage
description: "Episódio piloto da série de guias para CTFs/wargames"
tags:
  - "CTF"
  - "Engenharia Reversa"
author: lgribeiro
date: 2016-11-22
published: 2016-11-22
---

h3ll0, fr13nd. 
Esse post dá início a série voltada para guias de CTFs e wargames que foram resolvidos por membros do GRIS.

### Mas o que é um CTF ou wargame?
Os capture the flags (CTFs) e wargames são ótimas formas de testar conhecimento na área da segurança. Nesses desafios temos aplicações projetadas com vulnerabilidades disponíveis para que a comunidade coloque em prática aquilo que foi estudado.

O intuito das postagens sobre esses desafios não é fornecer respostas, mas guiar o leitor através da linha de raciocínio utilizada para resolvê-los . Estamos fazendo isso pois é difícil encontrar esse tipo de material em português e, além disso, é uma forma de fornecer a comunidade um pouco daquilo que aprendemos no GRIS. 

### O primeiro desafio:

Como primeiro desafio teremos o level01 do [IO netgarage](https://io.netgarage.org/). A temática dele é engenharia reversa (RE) em uma máquina IA-32.

O primeiro passo é se conectar ao servidor fornecido pelo wargame. para isso, utilizamos o protocolo [SSH](https://pt.wikipedia.org/wiki/Secure_Shell).

```bash
$ ssh level1@io.netgarage.org
password: level1
```

conforme a descrição do wargame, temos acesso somente ao programa correspondente ao usuário utilizado. E, além disso, os programas se encontram no diretório /levels. Portanto, acessamos esse diretório `$ cd /levels` 

feito isso, executamos o level01 `$./level01` e com isso, recebemos a output:

```bash
Enter the 3 digit passcode to enter:
```

Então, precisamos descobrir a input necessária para que seja possível ler a senha do o level2 do wargame. Para descobrir a chave, utilizaremos o [GDB](https://www.gnu.org/software/gdb/) (GNU debugger) da seguinte maneira  `$ gdb -q level01` O parâmetro -q é utilizado para o gdb não exibir sua "propaganda" habitual.

Feito isso, disassemblamos a função main do binário:
`(gdb) disassemble main`
e teremos como saída:

```bash
Dump of assembler code for function main:
   0x08048080 <+0>:	push   $0x8049128 //endereço colocado na pilha
   0x08048085 <+5>:	call   0x804810f //chamada de função
   0x0804808a <+10>:	call   0x804809f //chamada de função
   0x0804808f <+15>:	cmp    $0x10f,%eax //um valor é comparado com o retorno
   0x08048094 <+20>:	je     0x80480dc //se igual, pula para 0x80480dc
   0x0804809a <+26>:	call   0x8048103 //se não, chama 0x8048103
End of assembler dump.
```
 
O código acima está comentado, o que facilita enxergar a solução do problema. Aposto que o número em hexa que está sendo comparado com o registrador eax tem algo de especial.

![Bruh](/images/elliot_hackerman.jpg)
