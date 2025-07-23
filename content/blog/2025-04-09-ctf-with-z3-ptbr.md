---
title: "Resolvendo challenges de CTF usando Z3"
date: 2025-04-09 00:00:00 -0300
categories: [rev, z3, writeup]
tags: [z3, writeup]
math: true
mermaid: true
author: davi
# image: //
#   path: /commons/devices-mockup.png
#   width: 800
#   height: 500
#   alt: Responsive rendering of Chirpy theme on multiple devices.
---

Isso é um writeup de 2 challenges do PlaidCTF 2025 que podiam ser resolvidos usando Z3!

O primeiro challenge é um de criptografia chamado `excav8`. O segundo é um challenge de engenharia reversa chamado `Fool's Gulch`.

# Excav8

Como o próprio nome diz, ele tem sim algo relacionado com V8.

Esse é o arquivo `chall.py` fornecido pelo desafio:

```python
import subprocess

secret = open('secret.txt').read().strip()
secretbits = ''.join(f'{ord(i):08b}' for i in secret)

output = []

for bit in secretbits: 
    if bit == '0':
        output += [float(i) for i in subprocess.check_output('./d8 gen.js', shell=True).decode().split()]
    else:
        output += [float(i) for i in subprocess.check_output('node gen.js', shell=True).decode().split()]

for i in output:
    print(i)
```

E esse é o arquivo `gen.js` também fornecido:

```js
for (let i = 0; i < 24; i++) {
    console.log(Math.random());
}
```

Pelo código, dá para ver que ele lê um arquivo `secret.txt` meio sus (Nesse caso esse arquivo vai ser a flag) e transforma todo caractere em uma representação 8-bit. Depois disso, itera por cada bit, usando o binário `d8` para gerar 24 números aleatórios se o bit for 0 e usando o `node` para gerar 24 números aleatórios se o bit for 1.

O importante a se notar aqui é que a versão da engine do V8 usada aqui é do Chrome 13.6.1 e a versão do V8 usada pelo Node para gerar números aleatórios é um pouco mais antiga.

## Random Number Generation

Por quê a versão é importante?

Se olharmos para a implementação no V8: [https://github.com/v8/v8/blob/13.6.1/src/base/utils/random-number-generator.h#L111](https://github.com/v8/v8/blob/13.6.1/src/base/utils/random-number-generator.h#L111)

```cpp
  // Static and exposed for external use.
  static inline double ToDouble(uint64_t state0) {
    // Get a random [0,2**53) integer value (up to MAX_SAFE_INTEGER) by dropping
    // 11 bits of the state.
    double random_0_to_2_53 = static_cast<double>(state0 >> 11);
    // Map this to [0,1) by division with 2**53.
    constexpr double k2_53{static_cast<uint64_t>(1) << 53};
    return random_0_to_2_53 / k2_53;
  }

  // Static and exposed for external use.
  static inline void XorShift128(uint64_t* state0, uint64_t* state1) {
    uint64_t s1 = *state0;
    uint64_t s0 = *state1;
    *state0 = s0;
    s1 ^= s1 << 23;
    s1 ^= s1 >> 17;
    s1 ^= s0;
    s1 ^= s0 >> 26;
    *state1 = s1;
  }
```

E olharmos para a implementação do Node: [https://github.com/nodejs/node/blob/main/deps/v8/src/base/utils/random-number-generator.h#L111](https://github.com/nodejs/node/blob/main/deps/v8/src/base/utils/random-number-generator.h#L111)

```cpp
  // Static and exposed for external use.
  static inline double ToDouble(uint64_t state0) {
    // Exponent for double values for [1.0 .. 2.0)
    static const uint64_t kExponentBits = uint64_t{0x3FF0000000000000};
    uint64_t random = (state0 >> 12) | kExponentBits;
    return base::bit_cast<double>(random) - 1;
  }

  // Static and exposed for external use.
  static inline void XorShift128(uint64_t* state0, uint64_t* state1) {
    uint64_t s1 = *state0;
    uint64_t s0 = *state1;
    *state0 = s0;
    s1 ^= s1 << 23;
    s1 ^= s1 >> 17;
    s1 ^= s0;
    s1 ^= s0 >> 26;
    *state1 = s1;
  }
```

A maneira como convertemos floats é diferente!

Isso é bem importante, porque se usarmos o fato de que essas implementações são diferentes podemos descobrir qual foi usada para gerar cada número.

## Usando o Z3

[Z3](https://github.com/Z3Prover/z3) é um solucionador SMT. Você pode dar algumas restrições de um problema a ele e pedir para ele achar uma solução que satisfaça todas essas restrições.

Usar o Z3 nesse caso é lowkey bastante poggers. O jeito que um número aleatório é gerado usando a `Math.random()` se encaixa exatamente na categoria de coisas que podem ser resolvidas pelo Z3.

Basicamente podemos criar um modelo para o Z3 que tenta solucionar a geração de números aleatórios do Node (há vários desses pela internet) para cada batch de número fornecido e ver se ele consegue achar solução. Se dermos os 24 números gerados e as devidas restrições e pedirmos para ele achar uma solução (achar o estado interno) e ele de fato achar uma, então o bit vai ser 1!

Se não conseguirmos achar nenhuma solução, isso quer dizer que os números não devem ter sido gerados seguindo as mesmas restrições (então eles devem ter sido gerados pela nova implementação do V8) e o bit deve ser 0.

Dessa forma, podemos achar todos os bits da flag e resolver o desafio :)

Meu solve:

```python
#!/usr/bin/python3
import z3
import struct
import sys

def read_numbers(filename):
    with open(filename, 'r') as f:
        return [float(line.strip()) for line in f if line.strip()]

def check_batch(reversed_batch):
    solver = z3.Solver()
    se_state0, se_state1 = z3.BitVecs("se_state0 se_state1", 64)
    current_s0, current_s1 = se_state0, se_state1

    for num in reversed_batch:
        new_s0 = current_s1
        s1 = current_s0
        s1 ^= s1 << 23
        s1 ^= z3.LShR(s1, 17)
        s1 ^= new_s0
        s1 ^= z3.LShR(new_s0, 26)
        new_s1 = s1

        float_plus_1 = num + 1
        packed = struct.pack('d', float_plus_1)
        ulong = struct.unpack('<Q', packed)[0]
        mantissa = ulong & ((1 << 52) - 1)

        solver.add(z3.LShR(new_s0, 12) == mantissa)

        current_s0, current_s1 = new_s0, new_s1

    return solver.check() == z3.sat

def main():
    numbers = read_numbers('output.txt')
    batch_size = 24
    batches = [numbers[i:i+batch_size] for i in range(0, len(numbers), batch_size)]
    result_bits = []

    for batch in batches:
        if len(batch) != batch_size:
            continue
        reversed_batch = batch[::-1]
        valid = check_batch(reversed_batch)
        result_bits.append('1' if valid else '0')
        print('1' if valid else '0')

    print(''.join(result_bits))

if __name__ == "__main__":
    main()
```

É importante salientar que o V8 usa uma cache LIFO para gerar os números. Sempre que você pede para ele gerar um único número aleatório, na verdade ele gera bem mais números e os salva em um bucket. Isso significa que você precisa fornecer o batch de números na ordem contrária.

O resultado é esse aqui:

```
011001100110110001100001011001110011101000100000010100000100001101010100010001100111101101000010011101010110100101101100010001000011000101101110010001110101111101110110001110000101111101101001001101010101111101010011011101010100001101101000010111110011010001011111011100000110000100110001010011100010111000101110001011100111110100001010011100000110000101110011011100110111011101101111011100100110010000100000011101000110111100100000011100000110000101110010011101000010000000110010001110100010000001101111011000010111000100110001010011010100010000111001001100100110010101110110010100100111001101000100010110100111011001001000
```

Convertendo:

```
flag: PCTF{BuilD1nG_v8_i5_SuCh_4_pa1N...}
password to part 2: oaq1MD92evRsDZvH
```

Você pode achar outras implementações de um V8 random number cracker usando Z3 [nesse vídeo do YouTube](https://www.youtube.com/watch?v=_Iv6fBrcbAM). É um vídeo muito bem feito e me ajudou bastante a resolver esse challenge! Você deveria dar uma olhada nele quando puder.

# Fool's Gulch

Esse é um pouco diferente. É um challenge de engenharia reversa com um binário em aarch64.

## O setup

Eu estou em uma arquitetura x86-64, então pode ser que fique meio yabai.

O que eu fiz para rodar o programa foi o seguinte:

1. Instalei as bibliotecas de aarch64 no meu sistema

2. Copiei o `ld` para a pasta do challenge (na verdade qualquer lugar serve)

No meu caso, o `ld` estava em `/usr/aarch64-linux-gnu/lib/ld-linux-aarch64.so.1`.

Então eu só precisei fazer um `cp /usr/aarch64-linux-gnu/lib/ld-linux-aarch64.so.1 aarch64-libs/lib`.

3. Fiz o mesmo para a libc: `cp /usr/aarch64-linux-gnu/lib/libc.so.6 aarch64-libs/lib`.

4. Agora dá para usar o qemu para emular a biblioteca e passar a localização das bibliotecas como um argumento: `qemu-aarch64 -g 1234 -L aarch64-libs ./prospectors_claim`.

Aqui o `-g 1234` serve para debug. Nesse caso, rode o gdb-multiarch no binário e conecte à porta 1234 para debugar. Se você usar o argumento `-g 1234`, o binário não vai rodar até que você faça attach do gdb nele.

## Decompilando

Quando decompilado (usando o Ghidra), um arquivo BEM grande é gerado. Esse arquivo contém um monte de if conditions:

```c
undefined8 main(void)

{
  byte local_55;
  byte local_54;
  byte local_53;
  byte local_52;
  byte local_51;
  byte local_50;
  byte local_4f;
  byte local_4e;
  byte local_4d;
  byte local_4c;
  byte local_4b;
  byte local_4a;
  byte local_49;
  byte local_48;
  byte local_47;
  byte local_46;
  byte local_45;
  byte local_44;
  byte local_43;
  byte local_42;
  byte local_41;
  byte local_40;
  byte local_3f;
  byte local_3e;
  byte local_3d;
  byte local_3c;
  byte local_3b;
  byte local_3a;
  byte local_39;
  byte local_38;
  byte local_37;
  byte local_36;
  byte local_35;
  byte local_34;
  byte local_33;
  byte local_32;
  char local_31;
  byte local_30;
  byte local_2f;
  char local_2e;
  byte local_2d;
  byte local_2c;
  byte local_2b;
  byte local_2a;
  byte local_29;
  byte local_28;
  byte local_27;
  byte local_26;
  byte local_25;
  byte local_24;
  byte local_23;
  byte local_22;
  byte local_21;
  byte local_20;
  byte local_1f;
  byte local_1e;
  byte local_1d;
  byte local_1c;
  byte local_1b;
  byte local_1a;
  byte local_19;
  byte local_18;
  byte local_17;
  byte local_16;
  undefined4 local_14;

  local_14 = 0;
  setvbuf(_stdout,(char *)0x0,2,0);
  printf("=== The Prospector\'s Claim ===\n");
  printf("Old Man Jenkins\' map to his modest gold claim has been floating around\n");
  printf("Fool\'s Gulch for years. Most folks think it\'s worthless, but you\'ve\n");
  printf("noticed something peculiar in the worn-out corners...\n\n");
  printf("Enter the claim sequence: ");
  fgets((char *)&local_55,0x41,_stdin);
  if (local_3a == 0x32) {
    bump(&score);
  }
  if ((local_36 ^ local_3d) == 0xaa) {
    bump(&score);
  }
  if ((byte)(local_36 + local_27) == -0x70) {
    bump(&score);
  }
  if ((byte)(local_1a + local_31) == -0x7f) {
    bump(&score);
  }
  if (local_48 == 0xbe) {
    bump(&score);
  }
  if (local_35 == 100) {
    bump(&score);
  }
  if (local_3b == 0x31) {
    bump(&score);
  }
  if (local_2b == 0x39) {
    bump(&score);
  }
  if ((byte)(local_28 + local_16) == -0x6d) {
    bump(&score);
  }
  if (local_23 == 0x31) {
    bump(&score);
  }
  if ((byte)(local_54 + local_46) == '8') {
    bump(&score);
  }
  if ((local_48 ^ local_32) == 0x8e) {
    bump(&score);
  }
  if ((local_54 ^ local_27) == 0x71) {
    bump(&score);
  }
  if (local_2b == 0xb4) {
    bump(&score);
  }
  if (local_42 == 0x36) {
    bump(&score);
  }
  if (local_39 == local_25) {
    bump(&score);
  }
  if ((local_4b ^ local_34) == 3) {
    bump(&score);
  }
  if (local_37 == 0x76) {
    bump(&score);
  }
  if (local_50 == 0x32) {
    bump(&score);
  }
  if ((local_54 ^ local_25) == 0x85) {
    bump(&score);
  }
  if (local_53 == 0x54) {
    bump(&score);
  }
  if (local_3e == 0xe1) {
    bump(&score);
  }
  if ((local_4c ^ local_1f) == 0x81) {
    bump(&score);
  }
  if ((byte)(local_20 + local_3b) == 'a') {
    bump(&score);
  }
  if ((local_43 ^ local_1c) == 7) {
    bump(&score);
  }
  if ((local_43 ^ local_50) == 0x56) {
    bump(&score);
  }
  if ((byte)(local_47 + local_20) == 'f') {
    bump(&score);
  }
  if ((local_25 ^ local_1d) == 0x54) {
    bump(&score);
  }
  if (local_21 == 0xe5) {
    bump(&score);
  }
  if (local_31 == 'o') {
    bump(&score);
  }
  if ((byte)(local_30 + local_37) == 'f') {
    bump(&score);
  }
  if (local_23 == 0x31) {
    bump(&score);
  }
  if (local_3e == 0x38) {
    bump(&score);
  }
  if (local_2e == '4') {
    bump(&score);
  }
  if ((byte)(local_1d + local_42) == -0x69) {
    bump(&score);
  }

[...]

  if ((local_30 ^ local_53) == 0x67) {
    bump(&score);
  }
  if ((local_1f ^ local_45) == 0x7b) {
    bump(&score);
  }
  if ((local_28 ^ local_43) == 0x91) {
    bump(&score);
  }
  if (local_43 == 100) {
    bump(&score);
  }
  if (score < 0x119) {
    if (score < 0xFC) {
      if (score < 0x8C) {
        printf("\nThat claim\'s as empty as a desert well in August.\n",
        printf("Not a speck of gold to be found. Try another spot, prospector!\n");
      }
      else {
        printf("The saloon erupts in laughter as you show off your \'treasure\'.\n");
        printf("Keep prospecting - or take up farming instead!\n");
      }
    }
    else {
      printf("The assayer laughs you out of his office. \"Come back when you\'ve got\n");
      printf("something worth my time, greenhorn!\"\n");
    }
  }
  else {
    printf("You\'ve struck a rich vein of gold! Your claim is officially recorded\n");
    printf("at the assayer\'s office, and the flag is yours: %s\n",&local_55);
  }
  return 0;
}
```

Se as declarações de byte forem alteradas para um char[65], o código fica bem mais limpo.

## Usando Z3

Parece exatamente como um caso para o Z3, certo?

Há um monte de constraints e é preciso achar a solução que as satisfaça.

Se olharmos melhor, não precisamos solucionar para todas as constraints. Na verdade é impossível resolver todas ao mesmo tempo.

Isso significa que só é preciso achar uma solução boa o suficiente que faça com que o score seja maior do que `0x119`.

Também dá para fazer isso usando Z3!

## Otimizando

Z3 não só acha a solução exata, como também consegue achar uma solução ótima no caso em que a exata não existe.

Para resolver o problema, É preciso pegar todas essas if conditions e escrevê-las em formato do Z3. Também precisamos fazer com que, no caso de uma condição ser satisfeita, o score aumenta. Se pudermos otimizar isso tudo e conseguir uma solução maior do que `0x119`, resolvemos o problema :)

Então eu parseei o arquivo decompilado e gerei as condições do Z3 em python em outro arquivo. Aqui o arquivo que faz isso:

```python
import re

def generate_conditions(input_file, output_file):
    pattern = re.compile(r'if\s*\((.*?)\)\s*{\s*bump\(&score\);', re.DOTALL)
    var_pattern = re.compile(r'local_([0-9a-fA-F]{2})')
    byte_cast_pattern = re.compile(r'\(byte\)\s*\((.*?)\)')
    hex_pattern = re.compile(r'-0x([0-9a-fA-F]+)')

    with open(input_file, 'r') as f:
        code = f.read()

    conditions = pattern.findall(code)

    with open(output_file, 'w') as out:
        for i, cond in enumerate(conditions, 1):
            def var_replacer(match):
                offset = int(match.group(1), 16)
                index = 0x55 - offset
                return f'flag[{index}]'

            cond = var_pattern.sub(var_replacer, cond)

            cond = byte_cast_pattern.sub(r'(\1 & 0xFF)', cond)

            def hex_replacer(match):
                val = int(match.group(1), 16)
                return f'0x{(0x100 - val):02x}'

            cond = hex_pattern.sub(hex_replacer, cond)

            out.write(f'    # Condition {i}\n')
            out.write(f'    cond{i} = ({cond})\n')
            out.write(f'    score += If(cond{i}, 1, 0)\n\n')

if __name__ == '__main__':
    generate_conditions('original.c', 'z3_conditions.txt')
```

Isso gera um arquivo enorme com 400 condições:

```
    # Condition 1
    cond1 = (flag[27] == 0x32)
    score += If(cond1, 1, 0)

    # Condition 2
    cond2 = ((flag[31] ^ flag[24]) == 0xaa)
    score += If(cond2, 1, 0)

    # Condition 3
    cond3 = ((flag[31] + flag[46] & 0xFF) == 0x90)
    score += If(cond3, 1, 0)

    # Condition 4
    cond4 = ((flag[59] + flag[36] & 0xFF) == 0x81)
    score += If(cond4, 1, 0)

    # Condition 5
    cond5 = (flag[13] == 0xbe)
    score += If(cond5, 1, 0)

    # Condition 6
    cond6 = (flag[32] == 100)
    score += If(cond6, 1, 0)

    # Condition 7
    cond7 = (flag[26] == 0x31)
    score += If(cond7, 1, 0)

[...]

    # Condition 392
    cond392 = (flag[46] == flag[38])
    score += If(cond392, 1, 0)

    # Condition 393
    cond393 = ((flag[48] ^ flag[61]) == 5)
    score += If(cond393, 1, 0)

    # Condition 394
    cond394 = (flag[12] == 0x56)
    score += If(cond394, 1, 0)

    # Condition 395
    cond395 = ((flag[44] ^ flag[28]) == 0x51)
    score += If(cond395, 1, 0)

    # Condition 396
    cond396 = (flag[45] == 0x37)
    score += If(cond396, 1, 0)

    # Condition 397
    cond397 = ((flag[37] ^ flag[2]) == 0x67)
    score += If(cond397, 1, 0)

    # Condition 398
    cond398 = ((flag[54] ^ flag[16]) == 0x7b)
    score += If(cond398, 1, 0)

    # Condition 399
    cond399 = ((flag[45] ^ flag[18]) == 0x91)
    score += If(cond399, 1, 0)

    # Condition 400
    cond400 = (flag[18] == 100)
    score += If(cond400, 1, 0)
```

Agora basta escrever o solve e colar essas condições no arquivo. Também adicionei algumas restrições extras de que os 5 primeiros caracteres devem ser "PCTF{" e o último deve ser "}", que fazem parte do formato da flag.

```python
from z3 import *

def solve_flag():
    opt = Optimize()

    flag = [BitVec(f'flag_{i}', 8) for i in range(0x41)]

    opt.add(flag[0] == ord('P'))
    opt.add(flag[1] == ord('C'))
    opt.add(flag[2] == ord('T'))
    opt.add(flag[3] == ord('F'))
    opt.add(flag[4] == ord('{'))
    opt.add(flag[0x40] == ord('}'))

    for c in flag:
        opt.add(Or(And(c >= ord(' '), c <= ord('~')), c == 0))

    score = 0

    # LOL
    # Condition 1
    cond1 = (flag[27] == 0x32)
    score += If(cond1, 1, 0)

    # Condition 2
    cond2 = ((flag[31] ^ flag[24]) == 0xaa)
    score += If(cond2, 1, 0)

    # Condition 3
    cond3 = ((flag[31] + flag[46] & 0xFF) == 0x90)
    score += If(cond3, 1, 0)

    # Condition 4
    cond4 = ((flag[59] + flag[36] & 0xFF) == 0x81)
    score += If(cond4, 1, 0)

    # Condition 5
    cond5 = (flag[13] == 0xbe)
    score += If(cond5, 1, 0)

    # Condition 6
    cond6 = (flag[32] == 100)
    score += If(cond6, 1, 0)

    # Condition 7
    cond7 = (flag[26] == 0x31)
    score += If(cond7, 1, 0)

    # Condition 8
    cond8 = (flag[42] == 0x39)
    score += If(cond8, 1, 0)

    # Condition 9
    cond9 = ((flag[45] + flag[63] & 0xFF) == 0x93)
    score += If(cond9, 1, 0)

    # Condition 10
    cond10 = (flag[50] == 0x31)
    score += If(cond10, 1, 0)

[...]

    # Condition 398
    cond398 = ((flag[54] ^ flag[16]) == 0x7b)
    score += If(cond398, 1, 0)

    # Condition 399
    cond399 = ((flag[45] ^ flag[18]) == 0x91)
    score += If(cond399, 1, 0)

    # Condition 400
    cond400 = (flag[18] == 100)
    score += If(cond400, 1, 0)


    opt.maximize(score)

    if opt.check() == sat:
        model = opt.model()
        solution = ''.join([chr(model.eval(flag[i]).as_long()) for i in range(0x41)])
        print(f"Found flag: {solution}")
    else:
        print("No solution found")

if __name__ == "__main__":
    solve_flag()
```

Rodando isso, conseguimos a flag: `Found flag: PCTF{24d16126d6739d6ada82b125534d2ae2324b39ed72e5a1200c5ac96200}`

Tentando isso no binário:

```
$ qemu-aarch64 -L aarch64-libs ./prospectors_claim
=== The Prospector's Claim ===
Old Man Jenkins' map to his modest gold claim has been floating around
Fool's Gulch for years. Most folks think it's worthless, but you've
noticed something peculiar in the worn-out corners...

Enter the claim sequence: PCTF{24d16126d6739d6ada82b125534d2ae2324b39ed72e5a1200c5ac96200} 

🌟 PAYDIRT! 🌟
You've struck a rich vein of gold! Your claim is officially recorded
at the assayer's office, and the flag is yours: PCTF{24d16126d6739d6ada82b125534d2ae2324b39ed72e5a1200c5ac96200}
```

Resolvemos o desafio :)
