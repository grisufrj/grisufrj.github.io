---
title: "Game Hacking: Usando Patching de Maneira Dinâmica Para Modificar Jogos"
date: 2023-08-03
published: 2023-08-03
description: "Implementando patches dinâmicos em jogos com pymem"
tags: [Game Hacking]
images: ["/images/2023-08-03-1.png"]
author: ruhptura
categories:
  - "Medium"
tags:
  - "Game Hacking"
  - "Engenharia Reversa"
  - "Python"  
  - "ASM"

---

## Introdução

Você alguma vez já se perguntou se seria possível alterar o código fonte de um jogo à sua maneira e ainda assim continuar jogando? Nesse artigo, vamos modificar um jogo em tempo real em seu nível mais baixo, utilizando assembly, e desenvolver uma modificação simples em python utilizando a técnica de *patching* de memória.

Antes porém, é possível que você não saiba alguns termos que foram utilizados. Se você está se perguntando o que é assembly afinal, basicamente é uma linguagem que representa de maneira humanamente legível o próprio código de máquina, ou seja, é a linguagem de mais baixo nível possível, onde cada processador possui a sua própria. Instruções em ASM (assembly) são importantes pois conseguimos entender o que um jogo ou um programa qualquer está executando, mesmo após compilado em sua forma final.

Por outro lado, a palavra *patching* refere-se ao ato de alterar o código binário do jogo, através justamenta de suas instruções em ASM, como se você estivesse “reprogramando” aquele *game*.

Para demonstrar um pouco do poder de manipulação das instruções em ASM, usarei o [Assault Cube](https://github.com/assaultcube/AC) como exemplo, que é um jogo de tiro em primeira pessoa *open source*. Nosso objetivo será modificar o jogo, de maneira que quando o jogador receba um tiro ele ganhe vida!

Importante notar que parte da razão da escolha desse jogo é que ele não tem *anticheat*. Portanto, não tente aplicar o conteúdo desse artigo em qualquer jogo sem esse conhecimento, pois você pode ser banido. Além disso, seja uma pessoa ética e limite-se ao modo *singleplayer* para não atrapalhar a diversão de outras pessoas que estão sendo honestas.

## Fazendo análise dinâmica

O programa mais famoso, e sem dúvidas um dos mais úteis para se fazer análise dinâmica em jogos é o [Cheat Engine](https://github.com/cheat-engine/cheat-engine/). Usaremos ele para descobrir a posição de memória da instrução desejada.

Primeiramente, vamos descobrir a posição de memória da vida do jogador, ela será útil pois é o atributo alterado quando levamos um tiro, e conseguimos identificar a função modificadora a partir disso. Abrimos então o Cheat Engine e o jogo, carregamos o processo do Assault Cube, e pesquisamos pelo valor nominal da vida mostrado na tela, que inicialmente é 100.

![Figura 1](/images/2023-08-03-1.png)

Como foram muitos resultados encontrados, precisamos filtrar mais; para isso, vamos perder vida de alguma maneira dentro do jogo e pesquisar pelo novo valor.

![Figura 2](/images/2023-08-03-2.png)

Após ficar com 99 de HP e filtrar os resultados, chegamos em duas posições de memória. Testando uma por uma descobrimos a posição correta. Agora que temos em mãos o endereço de memória que representa a vida do personagem, observamos as instruções que alteram seu valor quando algum inimigo atira e nos causa dano. A instrução encontrada é a seguinte:

![Figura 3](/images/2023-08-03-3.png)

```nasm
sub [ebx+04],edi
```

Essa instrução subtrai um valor qualquer salvo no registrador **EDI** do valor de HP presente no endereço **EBX+04**. Sabendo disso, podemos então fazer o *patching*, substituindo **sub** por **add**, invertendo assim a lógica de dano do jogo. 

Vale dizer que os três bytes em hexadecimal que aparecem ao lado da instrução (**29 7B 04**), são equivalentes a **sub [ebx+04],edi** para o computador. 

## Fazendo o patching

Vamos então modificar diretamente a instrução escrita nessa região de memória:

![Figura 4](/images/2023-08-03-4.png)

O que fizemos aqui basicamente foi utilizar o nosso sistema operacional para modificar a nossa memória RAM, em uma região específica para afetar apenas o atributo do jogo que queremos alterar.

Somente com isso, agora toda vez que dano for causado, as entidades no mapa vão ganhar vida, entrando em modo imortal, atingindo então nosso objetivo. Conseguimos definitivamente modificar o jogo!

![Figura 5](/images/2023-08-03-5.png)

O Cheat Engine nos ajuda tanto a descobrir os endereços corretos como a modificar os mesmos, mas é possível fazer essa alteração de outras formas. Logo abaixo tem um exemplo de código em python, que realiza o mesmo *patching* que fizemos acima, utilizando a biblioteca pymem, que nos permite interagir com processos e memória.

Segue o código abaixo de referência, mas note que devido à versão do jogo, você precisará fazer engenharia reversa novamente e descobrir os *offsets* necessários.

```python
import pymem
import win32api
import time

PLAYER = 0x109B74 #Offset para se chegar no jogador (descoberto com engenharia reversa)
INC_HP = 0x28D1F #Offset do início das instruções até a instrução de dano tomado
TEXT_BASE = 0x1000 #Offset para se chegar nas instruções

def hack():
	p_handle = pymem.Pymem('ac_client.exe')
	for module in list(p_handle.list_modules()):
		if module.name == 'ac_client.exe':
			client = module.lpBaseOfDll #Endereço base do executável

	if not client:
		print("[-] Módulo não encontrado, saindo...")
		return

	print("[+] Cheat iniciado com sucesso!")
	print("F1: Inverter dano ON/OFF")
	print("END: Sair\n")

	inc_hp = False

	while(True):
		time.sleep(0.01)

		local_player = p_handle.read_uint(client + PLAYER)

		if not local_player:
			continue

		if win32api.GetAsyncKeyState(35):
			print("\nSaindo...")
			return

		if win32api.GetAsyncKeyState(112) & 1:
			inc_hp = not inc_hp

			if inc_hp:
				p_handle.write_bytes(client + TEXT_BASE + INC_HP, b"\x01\x7B\x04", 3) #Patching: invertendo o dano
				print("Inverter dano: ATIVADO")

			if not inc_hp:
				p_handle.write_bytes(client + TEXT_BASE + INC_HP, b"\x29\x7B\x04", 3) #Patching: voltando dano ao normal
				print("Inverter dano: DESATIVADO")

if __name__ == '__main__':
	hack()
```

## Referências

- Assault Cube: <https://github.com/assaultcube/AC>
- Cheat Engine: <https://github.com/cheat-engine/cheat-engine/>
- Documentação do Pymem: <https://pymem.readthedocs.io/en/latest/>