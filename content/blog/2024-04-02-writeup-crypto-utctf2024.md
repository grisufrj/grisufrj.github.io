---
title: "Writeup de alguns challenges de crypto do UTCTF 2024"
date: 2024-04-02
published: 2024-04-02
author: dkvhr
categories:
  - "Easy"
tags:
  - "Criptografia"
  - "CTF"
---

# RSA-256
### Descrição: 
```
Based on the military-grade encryption offered by AES-256, RSA-256 will usher in a new era of cutting-edge security... or at least, better security than RSA-128.
```

Código do chall:

```python
N = 77483692467084448965814418730866278616923517800664484047176015901835675610073
e = 65537
c = 11711610210897103123119971051169511511195828365955053549510511595100101100125
```

Esse foi um challenge bem simples de RSA. Pelo nome do chall e dando uma olhada no tamanho do N, a gente consegue ver que o N não é tão grande (justamente 256 bits!)
Você pode conferir isso da seguinte forma:
```python
N.bit_length()
```
Que nos retorna 256, o tamanho em bits.
Uma alternativa é fatorar o N e achar os fatores que o compõem. Outra, talvez mais simples, é procurar o número em um banco de dados e ver se tem os fatores lá.

![FactorDB](/images/FactorDB.png)

Aí ficou fácil.

```python
p = 1025252665848145091840062845209085931
q = 75575216771551332467177108987001026743883
phi = (p-1)*(q-1)
d = pow(e, -1, phi)
m = pow(c, d, N)
print(int.to_bytes(m, 32))
```

Achamos assim a flag ```utflag{just_send_plaintext}```

# Beginner: Anti-dcode.fr
### Descrição:
``` 
I've heard that everyone just uses dcode.fr to solve all of their crypto problems. Shameful, really.
This is really just a basic Caesar cipher, with a few extra random characters on either side of the flag. Dcode can handle that, right? >:)

The '{', '}', and '_' characters aren't part of the Caesar cipher, just a-z. As a reminder, all flags start with "utflag{". 
```
O arquivo é enorme e contém mais de 250,000 caracteres, então não tenho como colocar aqui.
Para resolver, procurei um site que trabalha com cifras de César e coloquei o texto encriptado lá. O site que utilizei foi o [https://cryptii.com/pipes/caesar-cipher](https://cryptii.com/pipes/caesar-cipher).
Mas primeiro, salvei o texto completo no meu clipboard. Como eu uso o X, o seguinte comando foi o que funcionou para mim:
```bash
cat text.txt | xclip -selection c
```

Finalmente, com um shift de 18, consegui achar a flag lá.

![shift18](/images/shift18.png)

(Procurei no Ctrl+F a cada novo shift)

![flag_shift_18](/images/flag_shift_18.png)

Por fim, a gente acha a flag ```utflag{rip_dcode}```. Bem simples :)

# numbers go brrr
### Descrição:
```
I wrote an amazing encryption service. It is definitely flawless, so I'll encrypt the flag and give it to you.
```

Código:
```python
#!/usr/bin/env python3
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad
import time

seed = int(time.time() * 1000) % (10 ** 6)
def get_random_number():
    global seed
    seed = int(str(seed * seed).zfill(12)[3:9])
    return seed

def encrypt(message):
    key = b''
    for i in range(8):
        key += (get_random_number() % (2 ** 16)).to_bytes(2, 'big')
    cipher = AES.new(key, AES.MODE_ECB)
    ciphertext = cipher.encrypt(pad(message, AES.block_size))
    return ciphertext.hex()

print("Thanks for using our encryption service! To get the encrypted flag, type 1. To encrypt a message, type 2.")
while True:
    print("What would you like to do (1 - get encrypted flag, 2 - encrypt a message)?")
    user_input = int(input())
    if(user_input == 1):
        break

    print("What is your message?")
    message = input()
    print("Here is your encrypted message:", encrypt(message.encode()))


flag = open('./src/flag.txt', 'r').read()
print("Here is the encrypted flag:", encrypt(flag.encode()))

```
Analisando o código, podemos ver que o que ele faz é pegar uma seed de acordo com o tempo atual para gerar uma chave "aleatória" para um esquema AES.
O que a gente precisa lembrar aqui é que AES é um sistema de criptografia simétrica, então resumidamente a chave para encriptar é a mesma para decriptar.
Isso quer dizer que se a gente achar a chave, podemos decriptar a flag facilmente.

Mas como achamos a chave?

Note como a chave é criada:
```python
def encrypt(message):
    key = b''
    for i in range(8):
        key += (get_random_number() % (2 ** 16)).to_bytes(2, 'big')
    cipher = AES.new(key, AES.MODE_ECB)
    ciphertext = cipher.encrypt(pad(message, AES.block_size))
    return ciphertext.hex()
```
Basicamente, são gerados números aleatórios a partir da get_random_number. Isso é calculado módulo 2^16 e é concatenado na chave.
Agora vamos olhar melhor para a função get_random_number:
```python
seed = int(time.time() * 1000) % (10 ** 6)
def get_random_number():
    global seed
    seed = int(str(seed * seed).zfill(12)[3:9])
    return seed
```
A função usa a variável global seed e faz algumas operações. O que a gente precisa perceber aqui é que, tendo a seed gerada, podemos facilmente calcular os números "aleatórios" gerados pela função também.

Essa seed é calculada com base no tempo em que o programa é iniciado e chega na time.time().
No python, a time.time() retorna um float com algumas casas decimais de precisão, que é um dos pontos que o chall explora. O que acontece é que ocorre uma multiplicação por 1000 do time.time(), então os milissegundos acabam sendo levados em conta.
Depois de pensar por um tempo em como eu faria para saber o tempo em milissegundos, o Ottoboni sugeriu simplesmente usar bruteforce para achar essas casas decimais, já que são 10^3 possibilidades.
Foi o que eu fiz. Funcionou!

Aqui o meu solve:
```python
#!/usr/bin/env python3
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad, unpad
import time
from pwn import *

sh = remote('betta.utctf.live', 7356)
current_time = time.time()
seed = (int(current_time) * 1000) % (10 ** 6)


def get_random_number():
    global seed
    seed = int(str(seed * seed).zfill(12)[3:9])
    return seed


def decrypt(ciphertext_hex, key):
    #key = b''
    #for i in range(8):
    #    key += (get_random_number_decrypt(seed) % (2 ** 16)).to_bytes(2, 'big')
    cipher = AES.new(key, AES.MODE_ECB)
    ciphertext = bytes.fromhex(ciphertext_hex)
    decrypted_message = unpad(cipher.decrypt(ciphertext), AES.block_size)
    return decrypted_message.decode('utf-8')


sh.recvuntil(b'?')
sh.sendline(b'1')
sh.recvuntil(b'Here is the encrypted flag: ')
encrypted_flag = sh.recvline().decode().strip()
print(f"Encrypted flag is: {encrypted_flag}")

for i in range(1, 1000):
    try:
        key = b''
        for j in range(8):
            key += (get_random_number() % (2 ** 16)).to_bytes(2, 'big')
        decrypt(encrypted_flag, key)
    except ValueError:
        print(f"{i} nao foi")
        seed = ((int(current_time) * 1000) + i) % (10 ** 6)
        continue
    print(decrypt(encrypted_flag, key))
    break


sh.interactive()
```
# Bits and pieces
### Descrição:
```
I really really like RSA, so implemented it myself <3.

A two parter.
```
Código do chall:

```python
n1= 16895844090302140592659203092326754397916615877156418083775983326567262857434286784352755691231372524046947817027609871339779052340298851455825343914565349651333283551138205456284824077873043013595313773956794816682958706482754685120090750397747015038669047713101397337825418638859770626618854997324831793483659910322937454178396049671348919161991562332828398316094938835561259917841140366936226953293604869404280861112141284704018480497443189808649594222983536682286615023646284397886256209485789545675225329069539408667982428192470430204799653602931007107335558965120815430420898506688511671241705574335613090682013
e1= 65537
c1= 7818321254750334008379589501292325137682074322887683915464861106561934924365660251934320703022566522347141167914364318838415147127470950035180892461318743733126352087505518644388733527228841614726465965063829798897019439281915857574681062185664885100301873341937972872093168047018772766147350521571412432577721606426701002748739547026207569446359265024200993747841661884692928926039185964274224841237045619928248330951699007619244530879692563852129885323775823816451787955743942968401187507702618237082254283484203161006940664144806744142758756632646039371103714891470816121641325719797534020540250766889785919814382

n2= 22160567763948492895090996477047180485455524932702696697570991168736807463988465318899280678030104758714228331712868417831523511943197686617200545714707332594532611440360591874484774459472586464202240208125663048882939144024375040954148333792401257005790372881106262295967972148685076689432551379850079201234407868804450612865472429316169948404048708078383285810578598637431494164050174843806035033795105585543061957794162099125273596995686952118842090801867908842775373362066408634559153339824637727686109642585264413233583449179272399592842009933883647300090091041520319428330663770540635256486617825262149407200317
e2= 65537
c2= 19690520754051173647211685164072637555800784045910293368304706863370317909953687036313142136905145035923461684882237012444470624603324950525342723531350867347220681870482876998144413576696234307889695564386378507641438147676387327512816972488162619290220067572175960616418052216207456516160477378246666363877325851823689429475469383672825775159901117234555363911938490115559955086071530659273866145507400856136591391884526718884267990093630051614232280554396776513566245029154917966361698708629039129727327128483243363394841238956869151344974086425362274696045998136718784402364220587942046822063205137520791363319144

n3= 30411521910612406343993844830038303042143033746292579505901870953143975096282414718336718528037226099433670922614061664943892535514165683437199134278311973454116349060301041910849566746140890727885805721657086881479617492719586633881232556353366139554061188176830768575643015098049227964483233358203790768451798571704097416317067159175992894745746804122229684121275771877235870287805477152050742436672871552080666302532175003523693101768152753770024596485981429603734379784791055870925138803002395176578318147445903935688821423158926063921552282638439035914577171715576836189246536239295484699682522744627111615899081
e3= 65537
c3= 17407076170882273876432597038388758264230617761068651657734759714156681119134231664293550430901872572856333330745780794113236587515588367725879684954488698153571665447141528395185542787913364717776209909588729447283115651585815847333568874548696816813748100515388820080812467785181990042664564706242879424162602753729028187519433639583471983065246575409341038859576101783940398158000236250734758549527625716150775997198493235465480875148169558815498752869321570202908633179473348243670372581519248414555681834596365572626822309814663046580083035403339576751500705695598043247593357230327746709126221695232509039271637
```

Batendo o olho no código, a coisa mais imediata que me veio a mente foi um broadcast attack. A gente pega os valores de n e os valores de c, põe em um array, calcula o teorema chinês do resto e depois tira a raiz.
No entanto, se tentarmos isso, olha o que acontece:
```python
sage: ns = [n1, n2, n3]
sage: cs = [c1, c2, c3]
sage: crt(cs, ns)
-----------------------------------------------------------------------
ValueError                            Traceback (most recent call last)
Cell In[5], line 1
----> 1 crt(cs, ns)

File /usr/lib/python3.11/site-packages/sage/arith/misc.py:3450, in crt(a, b, m, n)
   3329 r"""
   3330 Return a solution to a Chinese Remainder Theorem problem.
   3331
   (...)
   3447     58
   3448 """
   3449 if isinstance(a, list):
-> 3450     return CRT_list(a, b)
   3452 try:
   3453     f = (b-a).quo_rem

File /usr/lib/python3.11/site-packages/sage/arith/misc.py:3566, in CRT_list(values, moduli)
   3564 vs, ms = values[::2], moduli[::2]
   3565 for i, (v, m) in enumerate(zip(values[1::2], moduli[1::2])):
-> 3566     vs[i] = CRT(vs[i], v, ms[i], m)
   3567     ms[i] = lcm(ms[i], m)
   3568 values, moduli = vs, ms

File /usr/lib/python3.11/site-packages/sage/arith/misc.py:3464, in crt(a, b, m, n)
   3462 q, r = f(g)
   3463 if r != 0:
-> 3464     raise ValueError("no solution to crt problem since gcd(%s,%s) does not divide %s-%s" % (m, n, a, b))
   3465 from sage.arith.functions import lcm
   3467 x = a + q*alpha*py_scalar_to_element(m)

ValueError: no solution to crt problem since gcd(374421497892249265823794031537470448 ...
```

Recebemos uma exceção!

Investigando mais a fundo, decidi olhar se os valores de ```n``` possuem algum termo em comum:

```python
sage: gcd(n1, n2)
1
sage: gcd(n1, n3)
1
sage: gcd(n2, n3)
175136386393724074897068211302311758514344898633187862983126380556807924872210372704023620020763131468811275018725481764101835410780850364387004844957680252860643364609959757601263568806626614487575229052115194838589297358422557307359118854093864998895206960681533165623745478696564104830629591040860031236467
```

E olha só, não é que tem!

Se eles possuem um fator em comum, podemos dividir ambos pelo gcd(n2, n3), o que vai deixar n2 e n3 com apenas um termo. Dessa forma, já sabemos os primos p2, q2, p3, q3.
Ok, mas e o n1?
Tentando fatorar em minha máquina, percebi que isso poderia levar *bastante* tempo. Então decidi jogar o valor no FactorDB, assim como fizemos no primeiro chall.

![FactorDB2](/images/FactorDB2.png)

E lá achamos os nossos fatores p1 e q1 :)

Finalizando o solve:

```python
from Crypto.Util.number import long_to_bytes, getPrime
from sage.all import *

load('vals.sage')

p2 = int(n2 / gcd(n2, n3))
p3 = int(n3 / gcd(n2, n3))
q2 = gcd(n2, n3)
q3 = gcd(n2, n3)

phi2 = (p2-1)*(q2-1)
phi3 = (p3-1)*(q3-1)

e = 65537

d2 = pow(e, -1, phi2)
d3 = pow(e, -1, phi3)

m2 = long_to_bytes(pow(c2, d2, n2))
m3 = long_to_bytes(pow(c3, d3, n3))

p1 = 129984014749130366259742130443330376923069118727641845190136006048911945242427603092160936004682857611235008521722596025476170673607376869837675885556290582081941522328978811710862857253777650447221864279732376499043513950683086803379743964370215090077032772967632331576620201195241241611325672953583711299819
q1 = 129984014749130366259742130443330376923069118727641845190136006048911945242427603092160936004682857611235008521722596025476170673607376869837675885556290582081941522328978811710862857253777650447221864279732376499043513950683086803379743964370215090077032772967632331576620201195241241611325672953583711295127
phi1 = (p1-1)*(q1-1)
d1 = pow(e, -1, phi1)
m1 = long_to_bytes(pow(c1, d1, n1))

print(m1.decode(), m2.decode(), m3.decode())
```

Rodando o solve, achamos a flag: ```utflag{oh_no_it_didnt_work_</3_i_guess_i_can_just_use_standard_libraries_in_the_future}```
