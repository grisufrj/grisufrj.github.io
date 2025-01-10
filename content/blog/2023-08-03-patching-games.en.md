---
title: "Game Hacking: Using Dynamic Patching to Modify Games"
date: 2023-08-03
published: 2023-08-03
author: ruhptura
---

## Introduction

Have you ever considered modifying the source code of a game as you wish and playing with that modification? In this text, we will demonstrate how to modify a game in real-time using low-level knowledge, specifically assembly language. Additionally, we will develop a simple Python program to automate the process for us, employing the memory patching technique.

Before proceeding, it is entirely possible that some of the expressions mentioned above may not be familiar to you. If you're wondering what assembly is, it's a low-level programming language that represents machine code in a human-readable format, specific to each type of CPU. Assembly (or simply ASM) instructions are crucial as they allow us to understand what a program or game is doing, even after it has been compiled into its final form.

Furthermore, the term "patching" is used to indicate that we are modifying the game's binary code through its ASM instructions, as if we were reprogramming it.

To showcase the capabilities of this manipulation, let's take [Assault Cube](https://github.com/assaultcube/AC) as an example — a first-person shooter game that is open-source. Our objective is to modify the game in such a way that when the player gets hit by a gun, their health will regenerate!

It's essential to highlight that the reason I've chosen this game is because it lacks an anti-cheat system. Therefore, please refrain from attempting to replicate the modifications demonstrated here in any other game unless you are certain about the absence of anti-cheat measures, as doing so could lead to a ban. Additionally, I urge you to be ethical and avoid disturbing other players in multiplayer mode. Feel free to explore and experiment on your own, purely for the pursuit of knowledge.

## Dynamic Analysis

One of the most renowned and undoubtedly useful software for dynamic analysis in games is [Cheat Engine](https://github.com/cheat-engine/cheat-engine/). We will employ it to identify the memory position of the desired instruction.

First of all, our initial task is to determine the memory location of the player's health. This information will be crucial as the health attribute is modified when the player gets hit, allowing us to pinpoint the target function accurately. To achieve this, we open Cheat Engine and the game, load Assault Cube's process, and search for the HP value displayed on the screen, which is initially set to 100.

![Figure 1](/images/2023-08-03-1.png)

Numerous results were found, so we need to narrow down the search. To do that, let's intentionally decrease our health within the game and then search again for the new value.

![Figure 2](/images/2023-08-03-2.png)

After setting the HP value to 99 and refining the results, we identify two memory addresses. By testing them one by one (through direct modification), we pinpoint the correct one. Now that we have the location of player's HP in memory, we can examine precisely which instruction is responsible for altering its value when an enemy shoots at us to cause damage. The instruction we discovered is:

![Figure 3](/images/2023-08-03-3.png)

```nasm
sub [ebx+04],edi
```

This instruction subtracts a certain value stored in the **EDI** register from the HP value located at address **EBX+04**. With this information, we can proceed to patch it, altering the **sub** operation to **add**, effectively reversing the damage logic of the game.

Worth to say that the three bytes written in hexdecimal form (**29 7B 04**) that appears side by side with the instruction, are the equivalent to **sub [ebx+04],edi** for the computer.

## Patching

Let's proceed to directly modify the instruction written in this memory region:

![Figure 4](/images/2023-08-03-4.png)

What we've done here is essentially utilize our operating system to modify our RAM in a specific region, affecting only the attribute that we desired to change.

With just that modification, now every time damage is inflicted, the entities on the map will gain life, effectively becoming immortal, and we have achieved our goal. We have indeed successfully modified the game!

![Figure 5](/images/2023-08-03-5.png)

Cheat Engine assists us in discovering addresses of interest and modifying them, but it's also possible to achieve similar modifications using other methods. Below is an example of a python code that patches the game in the same manner as done above, utilizing the pymem library, which empowers us to interact with processes and memory.

Refer to the code below, but keep in mind that you may need to perform reverse engineering again to determine the correct offsets required if you attempt to replicate this, due to potential variations in the game's version.

```python
import pymem
import win32api
import time

PLAYER = 0x109B74 #Offset to player's address (discovered by reverse engineering)
INC_HP = 0x28D1F #Offset from the beginning of instructions to the target one
TEXT_BASE = 0x1000 #Offset to the beginning of instructions

def hack():
	p_handle = pymem.Pymem('ac_client.exe')
	for module in list(p_handle.list_modules()):
		if module.name == 'ac_client.exe':
			client = module.lpBaseOfDll #.exe base address

	if not client:
		print("[-] Module couldn't be found, exiting...")
		return

	print("[+] Cheat started succefully!")
	print("F1: Reverse damage ON/OFF")
	print("END: Exit\n")

	inc_hp = False

	while(True):
		time.sleep(0.01)

		local_player = p_handle.read_uint(client + PLAYER)

		if not local_player:
			continue

		if win32api.GetAsyncKeyState(35):
			print("\Exiting...")
			return

		if win32api.GetAsyncKeyState(112) & 1:
			inc_hp = not inc_hp

			if inc_hp:
				p_handle.write_bytes(client + TEXT_BASE + INC_HP, b"\x01\x7B\x04", 3) #Patching: reversing the damage
				print("Reverse damage: ON")

			if not inc_hp:
				p_handle.write_bytes(client + TEXT_BASE + INC_HP, b"\x29\x7B\x04", 3) #Patching: returning to the normal
				print("Reverse damage: OFF")

if __name__ == '__main__':
	hack()
```

## References

- Assault Cube: <https://github.com/assaultcube/AC>
- Cheat Engine: <https://github.com/cheat-engine/cheat-engine/>
- Pymem's documentation: <https://pymem.readthedocs.io/en/latest/>