---
title: "Modern Binary Exploitation: Laboratório de Engenharia Reversa"
date: 2024-08-11
description: "Nesse write-up, resolvo e explico o primeiro laboratório do [Modern Binary Exploitation](https://github.com/RPISEC/MBE) focado em **Engenharia Reversa**"
published: 2024-08-11 
categories:
  - easy
tags:
  - "Engenharia Reversa"
author: kitagawa
---

Olá! Nesse post, resolveremos o primeiro laboratório do [Modern Binary Exploitation](https://github.com/RPISEC/MBE) da [RPISEC](https://rpis.ec/) que aborda a **Engenharia Reversa**. Caso tenha interesse no assunto, temos diversos posts no blog que abordam o tema, basta acessar a tag!

# Sobre os laboratórios do MBE
Todos os laboratórios do curso residem dentro de uma máquina virtual disponibilizada no material através de uma imagem de disco para Ubuntu 14.04, que possui toda a configuração necessária para o Wargame. Os desafios são separados por laboratório e dificuldade, sendo C o mais fácil e A o mais difícil. Além disso, você acessa o challenge mediante ao usuário do respectivo desafio. Portanto, começando no C, o seu objetivo é exploitar o desafio para spawnar o terminal logado no usuário da próxima challenge e pegar a senha dele (que está em `/home/labXX/.pass`).


# Laboratório 01

O laboratório 01 aborda a **Engenharia Reversa**, tópico, esse, que é trabalhado durante as três primeiras lectures do material. Para uma melhor compreensão do que está ocorrendo, é necessário saber um pouco sobre:
- Programação em C
- Assembly x86
- Stack

Todas as ferramentas utilizadas estarão listadas ao final do write-up. 

## Lab1C

Conferindo algumas informações do arquivo:

```sh
lab1C@warzone:~$ file /levels/lab01/lab1C
/levels/lab01/lab1C: setuid ELF 32-bit LSB  executable, Intel 80386, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.24, BuildID[sha1]=1522352e3de50fd6d180831ba18e2bca16be4204, not stripped
```

Nosso código está rodando em 32 bits. Isso é importante para entendermos como são passados os parâmetros das funções, dadas as características da arquitetura. Nesse caso, os parâmetros são passados através da Stack.

Acessando com a senha padrão `lab01start`, o primeiro problema nos prompta com uma interface para inserir uma senha: 

```
-----------------------------
--- RPISEC - CrackMe v1.0 ---
-----------------------------

Password: 
```

Realizando o disassembly da função `main()`:

```sh
Dump of assembler code for function main:
   0x080486ad <+0>:	push   ebp
   0x080486ae <+1>:	mov    ebp,esp
   0x080486b0 <+3>:	and    esp,0xfffffff0
   0x080486b3 <+6>:	sub    esp,0x20
   0x080486b6 <+9>:	mov    DWORD PTR [esp],0x80487d0
   0x080486bd <+16>:	call   0x8048560 <puts@plt>
   0x080486c2 <+21>:	mov    DWORD PTR [esp],0x80487ee
   0x080486c9 <+28>:	call   0x8048560 <puts@plt>
   0x080486ce <+33>:	mov    DWORD PTR [esp],0x80487d0
   0x080486d5 <+40>:	call   0x8048560 <puts@plt>
   0x080486da <+45>:	mov    DWORD PTR [esp],0x804880c
   0x080486e1 <+52>:	call   0x8048550 <printf@plt>
   0x080486e6 <+57>:	lea    eax,[esp+0x1c]
   0x080486ea <+61>:	mov    DWORD PTR [esp+0x4],eax
   0x080486ee <+65>:	mov    DWORD PTR [esp],0x8048818
   0x080486f5 <+72>:	call   0x80485a0 <__isoc99_scanf@plt>
   0x080486fa <+77>:	mov    eax,DWORD PTR [esp+0x1c]
   0x080486fe <+81>:	cmp    eax,0x149a
   0x08048703 <+86>:	jne    0x8048724 <main+119>
   0x08048705 <+88>:	mov    DWORD PTR [esp],0x804881b
   0x0804870c <+95>:	call   0x8048560 <puts@plt>
   0x08048711 <+100>:	mov    DWORD PTR [esp],0x804882b
   0x08048718 <+107>:	call   0x8048570 <system@plt>
   0x0804871d <+112>:	mov    eax,0x0
   0x08048722 <+117>:	jmp    0x8048735 <main+136>
   0x08048724 <+119>:	mov    DWORD PTR [esp],0x8048833
   0x0804872b <+126>:	call   0x8048560 <puts@plt>
   0x08048730 <+131>:	mov    eax,0x1
   0x08048735 <+136>:	leave  
   0x08048736 <+137>:	ret    
End of assembler dump.
```

Nota-se, em `<main+81>`:

```sh
    0x080486ea <+61>:    mov    DWORD PTR [esp+0x4],eax
    0x080486ee <+65>:    mov    DWORD PTR [esp],0x8048818
    0x080486f5 <+72>:    call   0x80485a0 <__isoc99_scanf@plt>
    0x080486fa <+77>:    mov    eax,DWORD PTR [esp+0x1c]
    0x080486fe <+81>:    cmp    eax,0x149a
    0x08048703 <+86>:    jne    0x8048724 <main+119>
```

Há uma chamada de `scanf()` e uma comparação entre o valor inputado e `0x149a` (5274 em decimal). Portanto, ao inserir `5274` como senha, devemos satisfazer a comparação e evitar o jump subsequente.Assim, então, acessamos o usuário `lab1B` e recuperamos a senha `n0_str1ngs_n0_pr0bl3m`.

```sh
#!/bin/bash

file="/tmp/lab1B_pass.txt"

(python3 -c "print(str(5274))"; echo "cat /home/lab1B/.pass > $file") | /levels/lab01/lab1C

echo "Password stored at $file"
```

## Lab1B

Conferindo as informações do arquivo:

```sh
lab1B@warzone:/levels/lab01$ file lab1B
lab1B: setuid ELF 32-bit LSB  executable, Intel 80386, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.24, BuildID[sha1]=c9f07b581bd8d97cdc7c0ff1a288e20aea2df0f5, stripped
```

```sh
gdb-peda$ checksec
CANARY    : ENABLED
FORTIFY   : disabled
NX        : ENABLED
PIE       : disabled
RELRO     : Partial
```

Esse arquivo de 32 bits, agora, possui algumas proteções de stack.

### Análise das funções

Dessa vez, somos promptados com a mesma inteface de login:

```
-----------------------------
--- RPISEC - CrackMe v1.0 ---
-----------------------------

Password: 
```
Realizaremos, então, o disassembly da main:

```sh
Dump of assembler code for function main:
   0x08048be4 <+0>:	push   ebp
   0x08048be5 <+1>:	mov    ebp,esp
   0x08048be7 <+3>:	and    esp,0xfffffff0
   0x08048bea <+6>:	sub    esp,0x20
   0x08048bed <+9>:	push   eax
   0x08048bee <+10>:	xor    eax,eax
   0x08048bf0 <+12>:	je     0x8048bf5 <main+17>
   0x08048bf2 <+14>:	add    esp,0x4
   0x08048bf5 <+17>:	pop    eax
   0x08048bf6 <+18>:	mov    DWORD PTR [esp],0x0
   0x08048bfd <+25>:	call   0x80487b0 <time@plt>
   0x08048c02 <+30>:	mov    DWORD PTR [esp],eax
   0x08048c05 <+33>:	call   0x8048800 <srand@plt>
   0x08048c0a <+38>:	mov    DWORD PTR [esp],0x8048d88
   0x08048c11 <+45>:	call   0x80487d0 <puts@plt>
   0x08048c16 <+50>:	mov    DWORD PTR [esp],0x8048da6
   0x08048c1d <+57>:	call   0x80487d0 <puts@plt>
   0x08048c22 <+62>:	mov    DWORD PTR [esp],0x8048dc4
   0x08048c29 <+69>:	call   0x80487d0 <puts@plt>
   0x08048c2e <+74>:	mov    DWORD PTR [esp],0x8048de2
   0x08048c35 <+81>:	call   0x8048780 <printf@plt>
   0x08048c3a <+86>:	lea    eax,[esp+0x1c]
   0x08048c3e <+90>:	mov    DWORD PTR [esp+0x4],eax
   0x08048c42 <+94>:	mov    DWORD PTR [esp],0x8048dee
   0x08048c49 <+101>:	call   0x8048840 <__isoc99_scanf@plt>
   0x08048c4e <+106>:	mov    eax,DWORD PTR [esp+0x1c]
   0x08048c52 <+110>:	mov    DWORD PTR [esp+0x4],0x1337d00d
   0x08048c5a <+118>:	mov    DWORD PTR [esp],eax
   0x08048c5d <+121>:	call   0x8048a74 <test>     # chama test()
   0x08048c62 <+126>:	mov    eax,0x0
   0x08048c67 <+131>:	leave  
   0x08048c68 <+132>:	ret    
End of assembler dump.

```

Ao efetuar o disassembly, reparamos que a função `test()` é chamada em `<main+121>`:

```sh
   0x08048c3e <+90>:    mov    DWORD PTR [esp+0x4],eax
   0x08048c42 <+94>:    mov    DWORD PTR [esp],0x8048dee
   0x08048c49 <+101>:   call   0x8048840 <__isoc99_scanf@plt>
   0x08048c4e <+106>:   mov    eax,DWORD PTR [esp+0x1c]
   0x08048c52 <+110>:   mov    DWORD PTR [esp+0x4],0x1337d00d
   0x08048c5a <+118>:   mov    DWORD PTR [esp],eax
   0x08048c5d <+121>:   call   0x8048a74 <test>     # chama test()
```

Repare que o input do usuário é inserido como parametro, além de um valor fixo `0x1337d00d`. Conferindo a função `test()`:

```sh
Dump of assembler code for function test:
   0x08048a74 <+0>:	push   ebp
   0x08048a75 <+1>:	mov    ebp,esp
   0x08048a77 <+3>:	sub    esp,0x28
   0x08048a7a <+6>:	mov    eax,DWORD PTR [ebp+0x8]
   0x08048a7d <+9>:	mov    edx,DWORD PTR [ebp+0xc]
   0x08048a80 <+12>:	sub    edx,eax                  #subtrai os inputs
   0x08048a82 <+14>:	mov    eax,edx
   0x08048a84 <+16>:	mov    DWORD PTR [ebp-0xc],eax
   0x08048a87 <+19>:	cmp    DWORD PTR [ebp-0xc],0x15
   0x08048a8b <+23>:	ja     0x8048bd5 <test+353>         
   0x08048a91 <+29>:	mov    eax,DWORD PTR [ebp-0xc]      
   0x08048a94 <+32>:	shl    eax,0x2
   0x08048a97 <+35>:	add    eax,0x8048d30
   0x08048a9c <+40>:	mov    eax,DWORD PTR [eax]
   0x08048a9e <+42>:	jmp    eax
   0x08048aa0 <+44>:	mov    eax,DWORD PTR [ebp-0xc]
   0x08048aa3 <+47>:	mov    DWORD PTR [esp],eax
   0x08048aa6 <+50>:	call   0x80489b7 <decrypt>
   0x08048aab <+55>:	jmp    0x8048be2 <test+366>
   0x08048ab0 <+60>:	mov    eax,DWORD PTR [ebp-0xc]
   0x08048ab3 <+63>:	mov    DWORD PTR [esp],eax
   0x08048ab6 <+66>:	call   0x80489b7 <decrypt>
   .
   .
   .
   0x08048b85 <+273>:	jmp    0x8048be2 <test+366>
   0x08048b87 <+275>:	mov    eax,DWORD PTR [ebp-0xc]
   0x08048b8a <+278>:	mov    DWORD PTR [esp],eax
   0x08048b8d <+281>:	call   0x80489b7 <decrypt>
   0x08048b92 <+286>:	jmp    0x8048be2 <test+366>
   0x08048b94 <+288>:	mov    eax,DWORD PTR [ebp-0xc]
   0x08048b97 <+291>:	mov    DWORD PTR [esp],eax
   0x08048b9a <+294>:	call   0x80489b7 <decrypt>
   0x08048b9f <+299>:	jmp    0x8048be2 <test+366>
   0x08048ba1 <+301>:	mov    eax,DWORD PTR [ebp-0xc]
   0x08048ba4 <+304>:	mov    DWORD PTR [esp],eax
   0x08048ba7 <+307>:	call   0x80489b7 <decrypt>
   0x08048bac <+312>:	jmp    0x8048be2 <test+366>
   0x08048bae <+314>:	mov    eax,DWORD PTR [ebp-0xc]
   0x08048bb1 <+317>:	mov    DWORD PTR [esp],eax
   0x08048bb4 <+320>:	call   0x80489b7 <decrypt>
   0x08048bb9 <+325>:	jmp    0x8048be2 <test+366>
   0x08048bbb <+327>:	mov    eax,DWORD PTR [ebp-0xc]
   0x08048bbe <+330>:	mov    DWORD PTR [esp],eax
   0x08048bc1 <+333>:	call   0x80489b7 <decrypt>
   0x08048bc6 <+338>:	jmp    0x8048be2 <test+366>
   0x08048bc8 <+340>:	mov    eax,DWORD PTR [ebp-0xc]
   0x08048bcb <+343>:	mov    DWORD PTR [esp],eax
   0x08048bce <+346>:	call   0x80489b7 <decrypt>
   0x08048bd3 <+351>:	jmp    0x8048be2 <test+366>
   0x08048bd5 <+353>:	call   0x8048830 <rand@plt>
   0x08048bda <+358>:	mov    DWORD PTR [esp],eax
   0x08048bdd <+361>:	call   0x80489b7 <decrypt>
   0x08048be2 <+366>:	leave  
   0x08048be3 <+367>:	ret    
End of assembler dump.
```

Investigando a função `test()`, vemos que existe um grande if-else (ou switch-case) que realiza a chamada de uma função `decrypt()` em todos os seus casos. O único caso que se diferencia é o último, cujo é usada a função `rand()` para ser o parametro dessa chamada. 

Em partes, vamos analizar o comportamento da função para entendê-la. Primeiramente:

```sh
   0x08048a7a <+6>: mov    eax,DWORD PTR [ebp+0x8] # input do user
   0x08048a7d <+9>: mov    edx,DWORD PTR [ebp+0xc] # 0x1337d00d
   0x08048a80 <+12>:    sub    edx,eax          # subtrai os valores
```

Nesse momento, é calculada a diferença entre a senha inputada e o valor hexadecimal. Após isso:

```sh
   0x08048a82 <+14>:    mov    eax,edx
   0x08048a84 <+16>:    mov    DWORD PTR [ebp-0xc],eax
   0x08048a87 <+19>:    cmp    DWORD PTR [ebp-0xc],0x15
   0x08048a8b <+23>:    ja     0x8048bd5 <test+353>
```

Aqui, vemos o que provoca o caso do switch-case com o `rand()`: a diferença entre os valores ser maior que 21. Então, caso a diferença seja igual ou menor que 21:

```sh
   0x08048a91 <+29>:    mov    eax,DWORD PTR [ebp-0xc]
   0x08048a94 <+32>:    shl    eax,0x2
   0x08048a97 <+35>:    add    eax,0x8048d30
   0x08048a9c <+40>:    mov    eax,DWORD PTR [eax]
   0x08048a9e <+42>:    jmp    eax
```

Em todo caso, esse jump é tomado para um dos blocos do switch-case (que são todos iguais). Conferindo eles:

```sh
   0x08048aa0 <+XX>:    mov    eax,DWORD PTR [ebp-0xc]
   0x08048aa3 <+XX>:    mov    DWORD PTR [esp],eax
   0x08048aa6 <+XX>:    call   0x80489b7 <decrypt>
```

Basicamente, a diferença entre `0x1337d00d` e o input é enviada como parametro para `decrypt()`. Vejamos, então, a função:

```sh
Dump of assembler code for function decrypt:
   0x080489b7 <+0>:	push   ebp
   0x080489b8 <+1>:	mov    ebp,esp
   0x080489ba <+3>:	sub    esp,0x38
   0x080489bd <+6>:	mov    eax,gs:0x14
   0x080489c3 <+12>:	mov    DWORD PTR [ebp-0xc],eax
   0x080489c6 <+15>:	xor    eax,eax
   0x080489c8 <+17>:	mov    DWORD PTR [ebp-0x1d],0x757c7d51
   0x080489cf <+24>:	mov    DWORD PTR [ebp-0x19],0x67667360
   0x080489d6 <+31>:	mov    DWORD PTR [ebp-0x15],0x7b66737e
   0x080489dd <+38>:	mov    DWORD PTR [ebp-0x11],0x33617c7d
   0x080489e4 <+45>:	mov    BYTE PTR [ebp-0xd],0x0
   0x080489e8 <+49>:	push   eax
   0x080489e9 <+50>:	xor    eax,eax
   0x080489eb <+52>:	je     0x80489f0 <decrypt+57>
   0x080489ed <+54>:	add    esp,0x4
   0x080489f0 <+57>:	pop    eax
   0x080489f1 <+58>:	lea    eax,[ebp-0x1d]
   0x080489f4 <+61>:	mov    DWORD PTR [esp],eax
   0x080489f7 <+64>:	call   0x8048810 <strlen@plt>
   0x080489fc <+69>:	mov    DWORD PTR [ebp-0x24],eax
   0x080489ff <+72>:	mov    DWORD PTR [ebp-0x28],0x0
   0x08048a06 <+79>:	jmp    0x8048a28 <decrypt+113>
   0x08048a08 <+81>:	lea    edx,[ebp-0x1d]
   0x08048a0b <+84>:	mov    eax,DWORD PTR [ebp-0x28]
   0x08048a0e <+87>:	add    eax,edx
   0x08048a10 <+89>:	movzx  eax,BYTE PTR [eax]
   0x08048a13 <+92>:	mov    edx,eax
   0x08048a15 <+94>:	mov    eax,DWORD PTR [ebp+0x8]
   0x08048a18 <+97>:	xor    eax,edx
   0x08048a1a <+99>:	lea    ecx,[ebp-0x1d]
   0x08048a1d <+102>:	mov    edx,DWORD PTR [ebp-0x28]
   0x08048a20 <+105>:	add    edx,ecx
   0x08048a22 <+107>:	mov    BYTE PTR [edx],al
   0x08048a24 <+109>:	add    DWORD PTR [ebp-0x28],0x1
   0x08048a28 <+113>:	mov    eax,DWORD PTR [ebp-0x28]
   0x08048a2b <+116>:	cmp    eax,DWORD PTR [ebp-0x24]
   0x08048a2e <+119>:	jb     0x8048a08 <decrypt+81>
   0x08048a30 <+121>:	mov    DWORD PTR [esp+0x4],0x8048d03
   0x08048a38 <+129>:	lea    eax,[ebp-0x1d]
   0x08048a3b <+132>:	mov    DWORD PTR [esp],eax
   0x08048a3e <+135>:	call   0x8048770 <strcmp@plt>
   0x08048a43 <+140>:	test   eax,eax
   0x08048a45 <+142>:	jne    0x8048a55 <decrypt+158>
   0x08048a47 <+144>:	mov    DWORD PTR [esp],0x8048d14
   0x08048a4e <+151>:	call   0x80487e0 <system@plt>
   0x08048a53 <+156>:	jmp    0x8048a61 <decrypt+170>
   0x08048a55 <+158>:	mov    DWORD PTR [esp],0x8048d1c
   0x08048a5c <+165>:	call   0x80487d0 <puts@plt>
   0x08048a61 <+170>:	mov    eax,DWORD PTR [ebp-0xc]
   0x08048a64 <+173>:	xor    eax,DWORD PTR gs:0x14
   0x08048a6b <+180>:	je     0x8048a72 <decrypt+187>
   0x08048a6d <+182>:	call   0x80487c0 <__stack_chk_fail@plt>
   0x08048a72 <+187>:	leave  
   0x08048a73 <+188>:	ret    
End of assembler dump.
```

Em `decrypt()`, vemos a existência de uma string hardcoded:

```sh
   0x080489c8 <+17>:    mov    DWORD PTR [ebp-0x1d],0x757c7d51
   0x080489cf <+24>:    mov    DWORD PTR [ebp-0x19],0x67667360
   0x080489d6 <+31>:    mov    DWORD PTR [ebp-0x15],0x7b66737e
   0x080489dd <+38>:    mov    DWORD PTR [ebp-0x11],0x33617c7d
```

Transformando em caracteres, resulta em 
```Q}|u\`sfg~sf{}|a3``` (tenha em mente que estamos tratando de Little Endian!).

Além disso, existe um loop onde todos os caracteres dessa string estranha são decodificados (usando XOR) baseados no valor da diferença calculado em `test()`:

```sh
   0x08048a08 <+81>:    lea    edx,[ebp-0x1d]
   0x08048a0b <+84>:    mov    eax,DWORD PTR [ebp-0x28]
   0x08048a0e <+87>:    add    eax,edx
   0x08048a10 <+89>:    movzx  eax,BYTE PTR [eax]
   0x08048a13 <+92>:    mov    edx,eax
   0x08048a15 <+94>:    mov    eax,DWORD PTR [ebp+0x8]
   0x08048a18 <+97>:    xor    eax,edx
   0x08048a1a <+99>:    lea    ecx,[ebp-0x1d]
   0x08048a1d <+102>:   mov    edx,DWORD PTR [ebp-0x28]
   0x08048a20 <+105>:   add    edx,ecx
   0x08048a22 <+107>:   mov    BYTE PTR [edx],al
   0x08048a24 <+109>:   add    DWORD PTR [ebp-0x28],0x1
   0x08048a28 <+113>:   mov    eax,DWORD PTR [ebp-0x28]
   0x08048a2b <+116>:   cmp    eax,DWORD PTR [ebp-0x24]
   0x08048a2e <+119>:   jb     0x8048a08 <decrypt+81>
```

Após a decriptação, a string final é vista novamente em `main<+132>`:
```sh
   0x08048a30 <+121>:   mov    DWORD PTR [esp+0x4],0x8048d03 # string nova?
   0x08048a38 <+129>:   lea    eax,[ebp-0x1d]
   0x08048a3b <+132>:   mov    DWORD PTR [esp],eax # string final
   0x08048a3e <+135>:   call   0x8048770 <strcmp@plt>
```
Ela é enviada com uma outra string (do endereço `0x8048d03`) para ser comparada. Afinal, que string é essa?
```sh
gdb-peda$ x/s 0x8048d03
0x8048d03:	"Congratulations!"
```
Ok! Temos, então, a string `Congratulations!` como alvo!

### Objetivo

Como controlamos apenas o input, devemos inputar um número que, ao subtrair `0x1337d00d`, resulte em uma chave válida para a decriptação de 
```Q}|u\`sfg~sf{}|a3``` em `Congratulations!`. Nosso objetivo é descobrir essa chave e saber o payload válido.


### Resolvendo

Dado o funcionamento do XOR, podemos realizar XOR entre a string original e a encriptada e, assim, sabemos a chave simétrica. Usando Python:

```py
>>> ord("Q") ^ ord("C")
18
```

Precisamos, então, de uma diferença de 18! Como fazemos `0x1337D00D - input`, sabemos que `0x1337D00D + 0x12` é o input desejado. Assim, chegamos a `322424827` e voilà! Temos a senha `1337_3nCRyptI0n_br0`.

### Exploit

```py
#!/usr/bin/python

import pwn

# XOR allow us to do that :P
key = 0x1337d00d - (ord("Q") ^ ord("C"))

# Path to the save file
save_file = "/tmp/lab1A_pass.txt"


# Initiating process
p = pwn.process('/levels/lab01/lab1B')

# Sending the payloads
p.sendline(str(key))
p.sendline("cat /home/lab1A/.pass > "+ save_file)

print("[!] Password saved on " + save_file)

p.close()
```

## Lab1A

Conferindo as informações do arquivo:

```sh
lab1A@warzone:/levels/lab01$ file lab1A
lab1A: setuid ELF 32-bit LSB  executable, Intel 80386, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.24, BuildID[sha1]=8207a5ad5821e47f25412161c60dc24fd2f3386e, stripped
```

```sh
gdb-peda$ checksec
CANARY    : ENABLED
FORTIFY   : disabled
NX        : ENABLED
PIE       : disabled
RELRO     : Partial
```

Repare que todos os programas estão em 32 bits. Fiz todas essas verificações por prática, mas todos os programas desse curso são em 32 bits.

### Análise das funções

Realizando o disassembly da `main()`, verificamos que há 2 inputs pertinentes: um usuário e um serial.

```
.---------------------------.
|---------  RPISEC  --------|
|+ SECURE LOGIN SYS v. 3.0 +|
|---------------------------|
|~- Enter your Username:  ~-|
'---------------------------'
UsernameTeste
.---------------------------.
| !! NEW ACCOUNT DETECTED !!|
|---------------------------|
|~- Input your serial:    ~-|
'---------------------------'
123456
```

Realizando o disassembly da função `main()`, temos:

```sh
Dump of assembler code for function main:
   0x08048b44 <+0>:	push   ebp
   0x08048b45 <+1>:	mov    ebp,esp
   0x08048b47 <+3>:	and    esp,0xfffffff0
   0x08048b4a <+6>:	sub    esp,0x40
   0x08048b4d <+9>:	mov    eax,DWORD PTR [ebp+0xc]
   0x08048b50 <+12>:	mov    DWORD PTR [esp+0xc],eax
   0x08048b54 <+16>:	mov    eax,gs:0x14
   0x08048b5a <+22>:	mov    DWORD PTR [esp+0x3c],eax
   0x08048b5e <+26>:	xor    eax,eax
   0x08048b60 <+28>:	push   eax
   0x08048b61 <+29>:	xor    eax,eax
   0x08048b63 <+31>:	je     0x8048b68 <main+36>
   0x08048b65 <+33>:	add    esp,0x4
   0x08048b68 <+36>:	pop    eax
   0x08048b69 <+37>:	mov    DWORD PTR [esp],0x8048d73
   0x08048b70 <+44>:	call   0x8048810 <puts@plt>
   0x08048b75 <+49>:	mov    DWORD PTR [esp],0x8048d91
   0x08048b7c <+56>:	call   0x8048810 <puts@plt>
   0x08048b81 <+61>:	mov    DWORD PTR [esp],0x8048daf
   0x08048b88 <+68>:	call   0x8048810 <puts@plt>
   0x08048b8d <+73>:	mov    DWORD PTR [esp],0x8048dcd
   0x08048b94 <+80>:	call   0x8048810 <puts@plt>
   0x08048b99 <+85>:	mov    DWORD PTR [esp],0x8048deb
   0x08048ba0 <+92>:	call   0x8048810 <puts@plt>
   0x08048ba5 <+97>:	mov    DWORD PTR [esp],0x8048e09
   0x08048bac <+104>:	call   0x8048810 <puts@plt>
   0x08048bb1 <+109>:	mov    eax,ds:0x804b060
   0x08048bb6 <+114>:	mov    DWORD PTR [esp+0x8],eax 
   0x08048bba <+118>:	mov    DWORD PTR [esp+0x4],0x20
   0x08048bc2 <+126>:	lea    eax,[esp+0x1c]
   0x08048bc6 <+130>:	mov    DWORD PTR [esp],eax
   0x08048bc9 <+133>:	call   0x80487d0 <fgets@plt> # Coleta o username como string
   0x08048bce <+138>:	mov    DWORD PTR [esp],0x8048d73
   0x08048bd5 <+145>:	call   0x8048810 <puts@plt>
   0x08048bda <+150>:	mov    DWORD PTR [esp],0x8048e27
   0x08048be1 <+157>:	call   0x8048810 <puts@plt>
   0x08048be6 <+162>:	mov    DWORD PTR [esp],0x8048dcd
   0x08048bed <+169>:	call   0x8048810 <puts@plt>
   0x08048bf2 <+174>:	mov    DWORD PTR [esp],0x8048e45
   0x08048bf9 <+181>:	call   0x8048810 <puts@plt>
   0x08048bfe <+186>:	mov    DWORD PTR [esp],0x8048e09
   0x08048c05 <+193>:	call   0x8048810 <puts@plt>
   0x08048c0a <+198>:	lea    eax,[esp+0x18]
   0x08048c0e <+202>:	mov    DWORD PTR [esp+0x4],eax
   0x08048c12 <+206>:	mov    DWORD PTR [esp],0x8048d00
   0x08048c19 <+213>:	call   0x8048860 <__isoc99_scanf@plt> # Coleta o serial como int
   0x08048c1e <+218>:	mov    eax,DWORD PTR [esp+0x18]
   0x08048c22 <+222>:	mov    DWORD PTR [esp+0x4],eax
   0x08048c26 <+226>:	lea    eax,[esp+0x1c]
   0x08048c2a <+230>:	mov    DWORD PTR [esp],eax
   0x08048c2d <+233>:	call   0x8048a0f <auth>
   0x08048c32 <+238>:	test   eax,eax
   0x08048c34 <+240>:	jne    0x8048c55 <main+273>
   0x08048c36 <+242>:	mov    DWORD PTR [esp],0x8048e63
   0x08048c3d <+249>:	call   0x8048810 <puts@plt>
   0x08048c42 <+254>:	mov    DWORD PTR [esp],0x8048e72
   0x08048c49 <+261>:	call   0x8048820 <system@plt>
   0x08048c4e <+266>:	mov    eax,0x0
   0x08048c53 <+271>:	jmp    0x8048c5a <main+278>
   0x08048c55 <+273>:	mov    eax,0x1
   0x08048c5a <+278>:	mov    edx,DWORD PTR [esp+0x3c]
   0x08048c5e <+282>:	xor    edx,DWORD PTR gs:0x14
   0x08048c65 <+289>:	je     0x8048c6c <main+296>
   0x08048c67 <+291>:	call   0x8048800 <__stack_chk_fail@plt>
   0x08048c6c <+296>:	leave  
   0x08048c6d <+297>:	ret    
End of assembler dump.
```
Sendo um pouco mais breve, os inputs são armazenados na stack e enviados para `auth()`, em `<main+233>`. Realizando o disassembly de `auth()`:

```sh
Dump of assembler code for function auth:
   0x08048a0f <+0>:	push   ebp
   0x08048a10 <+1>:	mov    ebp,esp
   0x08048a12 <+3>:	sub    esp,0x28
   0x08048a15 <+6>:	mov    DWORD PTR [esp+0x4],0x8048d03
   0x08048a1d <+14>:	mov    eax,DWORD PTR [ebp+0x8]
   0x08048a20 <+17>:	mov    DWORD PTR [esp],eax
   0x08048a23 <+20>:	call   0x80487a0 <strcspn@plt>
   0x08048a28 <+25>:	mov    edx,DWORD PTR [ebp+0x8]
   0x08048a2b <+28>:	add    eax,edx
   0x08048a2d <+30>:	mov    BYTE PTR [eax],0x0
   0x08048a30 <+33>:	mov    DWORD PTR [esp+0x4],0x20
   0x08048a38 <+41>:	mov    eax,DWORD PTR [ebp+0x8]
   0x08048a3b <+44>:	mov    DWORD PTR [esp],eax
   0x08048a3e <+47>:	call   0x8048850 <strnlen@plt>
   0x08048a43 <+52>:	mov    DWORD PTR [ebp-0xc],eax
   0x08048a46 <+55>:	push   eax
   0x08048a47 <+56>:	xor    eax,eax
   0x08048a49 <+58>:	je     0x8048a4e <auth+63>
   0x08048a4b <+60>:	add    esp,0x4
   0x08048a4e <+63>:	pop    eax
   0x08048a4f <+64>:	cmp    DWORD PTR [ebp-0xc],0x5
   0x08048a53 <+68>:	jg     0x8048a5f <auth+80>
   0x08048a55 <+70>:	mov    eax,0x1
   0x08048a5a <+75>:	jmp    0x8048b42 <auth+307>
   0x08048a5f <+80>:	mov    DWORD PTR [esp+0xc],0x0
   0x08048a67 <+88>:	mov    DWORD PTR [esp+0x8],0x1
   0x08048a6f <+96>:	mov    DWORD PTR [esp+0x4],0x0
   0x08048a77 <+104>:	mov    DWORD PTR [esp],0x0
   0x08048a7e <+111>:	call   0x8048870 <ptrace@plt>
   0x08048a83 <+116>:	cmp    eax,0xffffffff
   0x08048a86 <+119>:	jne    0x8048ab6 <auth+167>
   0x08048a88 <+121>:	mov    DWORD PTR [esp],0x8048d08
   0x08048a8f <+128>:	call   0x8048810 <puts@plt>
   0x08048a94 <+133>:	mov    DWORD PTR [esp],0x8048d2c
   0x08048a9b <+140>:	call   0x8048810 <puts@plt>
   0x08048aa0 <+145>:	mov    DWORD PTR [esp],0x8048d50
   0x08048aa7 <+152>:	call   0x8048810 <puts@plt>
   0x08048aac <+157>:	mov    eax,0x1
   0x08048ab1 <+162>:	jmp    0x8048b42 <auth+307>
   0x08048ab6 <+167>:	mov    eax,DWORD PTR [ebp+0x8]
   0x08048ab9 <+170>:	add    eax,0x3
   0x08048abc <+173>:	movzx  eax,BYTE PTR [eax]
   0x08048abf <+176>:	movsx  eax,al
   0x08048ac2 <+179>:	xor    eax,0x1337
   0x08048ac7 <+184>:	add    eax,0x5eeded
   0x08048acc <+189>:	mov    DWORD PTR [ebp-0x10],eax
   0x08048acf <+192>:	mov    DWORD PTR [ebp-0x14],0x0
   0x08048ad6 <+199>:	jmp    0x8048b26 <auth+279>
   0x08048ad8 <+201>:	mov    edx,DWORD PTR [ebp-0x14]
   0x08048adb <+204>:	mov    eax,DWORD PTR [ebp+0x8]
   0x08048ade <+207>:	add    eax,edx
   0x08048ae0 <+209>:	movzx  eax,BYTE PTR [eax]
   0x08048ae3 <+212>:	cmp    al,0x1f
   0x08048ae5 <+214>:	jg     0x8048aee <auth+223>
   0x08048ae7 <+216>:	mov    eax,0x1
   0x08048aec <+221>:	jmp    0x8048b42 <auth+307>
   0x08048aee <+223>:	mov    edx,DWORD PTR [ebp-0x14]
   0x08048af1 <+226>:	mov    eax,DWORD PTR [ebp+0x8]
   0x08048af4 <+229>:	add    eax,edx
   0x08048af6 <+231>:	movzx  eax,BYTE PTR [eax]
   0x08048af9 <+234>:	movsx  eax,al
   0x08048afc <+237>:	xor    eax,DWORD PTR [ebp-0x10]
   0x08048aff <+240>:	mov    ecx,eax
   0x08048b01 <+242>:	mov    edx,0x88233b2b
   0x08048b06 <+247>:	mov    eax,ecx
   0x08048b08 <+249>:	mul    edx
   0x08048b0a <+251>:	mov    eax,ecx
   0x08048b0c <+253>:	sub    eax,edx
   0x08048b0e <+255>:	shr    eax,1
   0x08048b10 <+257>:	add    eax,edx
   0x08048b12 <+259>:	shr    eax,0xa
   0x08048b15 <+262>:	imul   eax,eax,0x539
   0x08048b1b <+268>:	sub    ecx,eax
   0x08048b1d <+270>:	mov    eax,ecx
   0x08048b1f <+272>:	add    DWORD PTR [ebp-0x10],eax
   0x08048b22 <+275>:	add    DWORD PTR [ebp-0x14],0x1
   0x08048b26 <+279>:	mov    eax,DWORD PTR [ebp-0x14]
   0x08048b29 <+282>:	cmp    eax,DWORD PTR [ebp-0xc]
   0x08048b2c <+285>:	jl     0x8048ad8 <auth+201>
   0x08048b2e <+287>:	mov    eax,DWORD PTR [ebp+0xc]
   0x08048b31 <+290>:	cmp    eax,DWORD PTR [ebp-0x10]
   0x08048b34 <+293>:	je     0x8048b3d <auth+302>
   0x08048b36 <+295>:	mov    eax,0x1
   0x08048b3b <+300>:	jmp    0x8048b42 <auth+307>
   0x08048b3d <+302>:	mov    eax,0x0
   0x08048b42 <+307>:	leave  
   0x08048b43 <+308>:	ret    
End of assembler dump.
```

O assembly dessa função está muito extenso. Portanto, para facilitar, utilizei o decompilador do Ghidra para entender melhor o que estava ocorrendo:


```C
/* WARNING: Removing unreachable block (ram,0x08048a4b) */
/* WARNING: Restarted to delay deadcode elimination for space: stack */

undefined4 auth(char *param_1,uint param_2)

{
  size_t sVar1;
  undefined4 uVar2;
  long lVar3;
  int local_18;
  uint local_14;
  
  sVar1 = strcspn(param_1,"\n");
  param_1[sVar1] = '\0';
  sVar1 = strnlen(param_1,0x20);
  if ((int)sVar1 < 6) {
    uVar2 = 1;
  }
  else {
    lVar3 = ptrace(PTRACE_TRACEME);
    if (lVar3 == -1) {
      puts("\x1b[32m.---------------------------.");
      puts("\x1b[31m| !! TAMPERING DETECTED !!  |");
      puts("\x1b[32m\'---------------------------\'");
      uVar2 = 1;
    }
    else {
      local_14 = ((int)param_1[3] ^ 0x1337U) + 0x5eeded;
      for (local_18 = 0; local_18 < (int)sVar1; local_18 = local_18 + 1) {
        if (param_1[local_18] < ' ') {
          return 1;
        }
        local_14 = local_14 + ((int)param_1[local_18] ^ local_14) % 0x539;
      }
      if (param_2 == local_14) {
        uVar2 = 0;
      }
      else {
        uVar2 = 1;
      }
    }
  }
  return uVar2;
}
```

Adaptando os nomes de algumas variáveis para facilitar a leitura, resultamos em:

```C
int auth(char *username,uint serial)

{
  size_t inputLength;
  undefined4 flag;
  long ptrace?;
  int i;
  uint temp;
  
  inputLength = strcspn(username,"\n");
  username[inputLength] = '\0';
  inputLength = strnlen(username,0x20);
  if ((int)inputLength < 6) {
    flag = 1;
  }
  else {
    ptrace? = ptrace(PTRACE_TRACEME);
    if (ptrace? == -1) {
      puts("\x1b[32m.---------------------------.");
      puts("\x1b[31m| !! TAMPERING DETECTED !!  |");
      puts("\x1b[32m\'---------------------------\'");
      flag = 1;
    }
    else {
      temp = ((int)username[3] ^ 0x1337U) + 0x5eeded;
      for (i = 0; i < (int)inputLength; i = i + 1) {
        if (username[i] < ' ') {
          return 1;
        }
        temp = temp + ((int)username[i] ^ temp) % 0x539;
      }
      if (serial == temp) {
        flag = 0;
      }
      else {
        flag = 1;
      }
    }
  }
  return flag;
}
```


Em `auth()`, podemos verificar que o programa retira, do nome do usuário, o caracter `\n` e substitui por `\0`, além de calcular e armazenar o comprimento da string. Cabe salientar que usernames com menos de 6 caracteres não são aceitos pelo programa e resultam em retorno imediato com valor 1 (que invalida a sessão). Após isso, ele faz algo em uma lógica fixa com o username e compara com o serial inputado para validar. 

### Resolução

Para resolver o problema, criei um pequeno keygen a partir da lógica exibida pelo decompilador.

```py
#!/usr/bin/python

import pwn

#input
username = str(raw_input("[!] Insert the username that you want to use: "))

# Serial logic
num = (ord(username[3]) ^ 0x1337) + 6221293

for char in username:
    if char == ' ':
        print("lol")
        exit(3)

    if char == '\n':
        continue

    num = num + (ord(char) ^ num) % 1337

# Printing out the serial generated for the inputed username
print("[!] Serial: " + str(num))

 # Starting process
p = pwn.process("/levels/lab01/lab1A")

# Sending payload
p.sendline(username)

p.sendline(str(num))

# Give the user the shell
p.interactive()

# Close the program
p.close()
```

A partir de um username inputado, ele cria um serial válido e executa o programa. Basta utilizá-lo e voilà! Fim de laboratório!

## Ferramentas utilizadas:

- [GEF](https://github.com/hugsy/gef), uma extensão para o [GDB](https://www.gnu.org/savannah-checkouts/gnu/gdb/index.html)
- [GHIDRA](https://ghidra-sre.org/), uma ferramenta de análise de binários.
- [pwntools](https://github.com/Gallopsled/pwntools), uma biblioteca de Python para fabricação de exploits.
