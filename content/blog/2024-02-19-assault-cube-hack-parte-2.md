---
title: "Assault Cube Hack - Parte 2"
date: 2024-02-19
published: 2024-02-19
tags: [Game Hacking]
series: ["Assault Cube Hacking"]
author: jurbas
categories:
  - "Easy"
tags:
  - "C++"
  - "Game Hacking"
---

# Desenvolvimento de Código

## **Criando o Projeto**

Antes de começar o código vamos criar um projeto no Visual Studio, procure e selecione _Dynamic Link Library (DLL)_ para C++ como tipo de projeto como na imagem a seguir.

![Untitled](/images/ac-hack-7.png)

Dê um nome ao projeto e escolha um local para salvá-lo e teremos uma estrutura inicial desta forma.

![Untitled](/images/ac-hack-8.png)

A estrutura do nosso código se dividirá em 3 funções, **`DllMain`**, **`HackThread`** e **`UpdatePlayerStats`**. Irei explicar mais adiante detalhadamente cada uma.

```cpp
BOOL APIENTRY DllMain(HMODULE hModule, DWORD  ul_reason_for_call, LPVOID lpReserved);
DWORD WINAPI HackThread(HMODULE hModule);
void UpdatePlayerStats(bool bHealth, bool bAmmo, uintptr_t* localPlayerPtr);
```

## Implementando DllMain

**`DllMain`** é chamada pelo Windows quando a DLL é carregada ou descarregada. Quando a DLL é anexada a um processo (_DLL_PROCESS_ATTACH_), uma nova thread é criada para executar o código do nosso _hack_. E para isso chamamos **`CreateThread`** [](https://learn.microsoft.com/pt-br/windows/win32/api/processthreadsapi/nf-processthreadsapi-createthread)para criar uma nova _thread_ passando **`HackThread`** como argumento.

Ao criar uma nova _thread_ a função **`CreateThread`** [](https://learn.microsoft.com/pt-br/windows/win32/api/processthreadsapi/nf-processthreadsapi-createthread)retorna um _handle_ (uma referência) para essa _thread_ nova criada. Por isso ao terminar a execução precisamos usar **`CloseHandle`** para fechar esse _handle_ e liberar qualquer recurso associado a ele.

```cpp
BOOL APIENTRY DllMain(HMODULE hModule, DWORD  ul_reason_for_call, LPVOID lpReserved) {
    switch (ul_reason_for_call) {
    case DLL_PROCESS_ATTACH:
        CloseHandle(CreateThread(nullptr, 0, (LPTHREAD_START_ROUTINE)HackThread, hModule, 0, nullptr));
        break;
    case DLL_THREAD_ATTACH:
    case DLL_THREAD_DETACH:
    case DLL_PROCESS_DETACH:
        break;
    }
    return TRUE;
}
```

## Implementando HackThread

A função **`HackThread`** atua como o núcleo do _hack_, semelhante a uma função **`main`** , **`HackThread`** será responsável por gerenciar o fluxo de execução do _hack_, chamando funções específicas e gerenciando a lógica principal.

Iremos alocar um console para imprimir mensagens de debug, por exemplo se o cheat está ou não ativado.

```cpp
AllocConsole();
freopen_s((FILE**)stdout, "CONOUT$", "w", stdout);
std::cout << "Hack ativado. Pressione END para sair." << std::endl;
```

O loop principal atua como o coração do cheat. Em cada iteração, o programa deve verificar se as teclas de ativação foram pressionadas, e para isso utilizaremos a função **`GetAsyncKeyState()`**.

Dentro do loop, desreferenciamos _localPlayerPtr_ usando o offset que conseguimos no Cheat Engine e a partir dele que iremos acessar a vida e a munição usando seus respectivos offsets.

Ao fim de cada iteração chamamos **`UpdatePlayerStats`** que irá verificar a ativação do cheat e sobrescrever os valores da vida e munição caso estejam ativos. Usamos **`Sleep`** para poupar a utilização de CPU sem diminuir a eficiência do \*hack\*.

```cpp
while (true) {
        uintptr_t* localPlayerPtr = (uintptr_t*)(moduleBase + 0x109B74);
        if (GetAsyncKeyState(VK_END) & 1) {
						std::cout << "Finalizando HackThread." << std::endl;
						break;
        }
        if (GetAsyncKeyState(VK_F1) & 1) {
            bHealth = !bHealth;
            std::cout << "Saude infinita " << (bHealth ? "ativada" : "desativada") << std::endl;
        }
        if (GetAsyncKeyState(VK_F2) & 1) {
            bAmmo = !bAmmo;
            std::cout << "Municao infinita " << (bAmmo ? "ativada" : "desativada") << std::endl;
        }
        UpdatePlayerStats(bHealth, bAmmo, localPlayerPtr);
        Sleep(10);
    }
```

Ao quebrar o loop, indicando a finalização do _hack_, é crucial realizar a limpeza dos recursos utilizados. Para isso, utilizamos **`fclose`** para finalizar os buffers do console e **`FreeConsole`** para desanexar e fechar o console alocado. Por fim, chamamos **`FreeLibraryAndExitThread`** para liberar a DLL e terminar a thread de execução sem prejudicar o processo do jogo.

```cpp
fclose((FILE*)stdout);
FreeConsole();
FreeLibraryAndExitThread(hModule, 0);
```

## Implementando UpdatePlayerStats

A função **`UpdatePlayerStats`** será responsável por modificar os dados do jogo, como vida e munição.

Inicialmente, verificamos a validade do ponteiro **`localPlayerPtr`** , a tentativa de acessar ou modificar dados através de um ponteiro nulo pode levar a um crash do jogo. Somente quando confirmada a ativação de algum cheat, a função continua com a desreferenciação dos ponteiros.

```cpp
void UpdatePlayerStats(bool bHealth, bool bAmmo, uintptr_t* localPlayerPtr) {
    if (localPlayerPtr) { // Checamos se o jogador existe para não ocasionar um crash
        if (bHealth) {
						// Desreferencia a vida adicionando o offset a localPlayerPtr
						// e atualiza o valor para 999
            *(int*)(*localPlayerPtr + 0xF8) = 999;
        }

        if (bAmmo) {
            std::vector<unsigned int> ammoOffsets = { 0x378, 0x14, 0x0 };
            uintptr_t addr = *localPlayerPtr;
						// Navega pelos ponteiros multinível dado a lista de offsets,
						// desreferencia e altera o valor da munição atual
            for (unsigned int i = 0; i < ammoOffsets.size(); ++i) {
                addr = *(uintptr_t*)(addr + ammoOffsets[i]);
            }
            *(int*)addr = 999;
        }
    }
}
```

## Código final

```cpp
#include "pch.h"
#include <iostream>
#include <vector>
#include <Windows.h>

void UpdatePlayerStats(bool bHealth, bool bAmmo, uintptr_t* localPlayerPtr) {
    if (localPlayerPtr) {
        if (bHealth) {
            *(int*)(*localPlayerPtr + 0xF8) = 999;
        }

        if (bAmmo) {
            std::vector<unsigned int> ammoOffsets = { 0x378, 0x14, 0x0 };
            uintptr_t addr = (uintptr_t)localPlayerPtr;
            for (unsigned int i = 0; i < ammoOffsets.size(); ++i)
            {
                addr = *(uintptr_t*)addr;
                addr += ammoOffsets[i];
            }
            *(int*)addr = 999;

        }
    }
}

DWORD WINAPI HackThread(HMODULE hModule) {
    AllocConsole();
    freopen_s((FILE**)stdout, "CONOUT$", "w", stdout);
    std::cout << "HackThread iniciada. Pressione END para sair." << std::endl;

    uintptr_t moduleBase = (uintptr_t)GetModuleHandle(nullptr);
    bool bHealth = false, bAmmo = false;

    while (true) {
        uintptr_t* localPlayerPtr = (uintptr_t*)(moduleBase + 0x109B74);

        if (GetAsyncKeyState(VK_END) & 1) {
            std::cout << "Finalizando HackThread." << std::endl;
            break;
        }

        if (GetAsyncKeyState(VK_F1) & 1) {
            bHealth = !bHealth;
            std::cout << "Saude infinita " << (bHealth ? "ativada" : "desativada") << std::endl;
        }
        if (GetAsyncKeyState(VK_F2) & 1) {
            bAmmo = !bAmmo;
            std::cout << "Municao infinita " << (bAmmo ? "ativada" : "desativada") << std::endl;
        }
        UpdatePlayerStats(bHealth, bAmmo, localPlayerPtr);

        Sleep(10);
    }

    fclose((FILE*)stdout);
    FreeConsole();
    FreeLibraryAndExitThread(hModule, 0);
    return 0;

}

BOOL APIENTRY DllMain(HMODULE hModule, DWORD  ul_reason_for_call, LPVOID lpReserved) {
    switch (ul_reason_for_call) {
    case DLL_PROCESS_ATTACH:
        CloseHandle(CreateThread(nullptr, 0, (LPTHREAD_START_ROUTINE)HackThread, hModule, 0, nullptr));
        break;
    case DLL_THREAD_ATTACH:
    case DLL_THREAD_DETACH:
    case DLL_PROCESS_DETACH:
        break;
    }
    return TRUE;
}
```

## Referências da API do Windows

- **[CreateThread](https://learn.microsoft.com/pt-br/windows/win32/api/processthreadsapi/nf-processthreadsapi-createthread)**
- **[CloseHandle](https://learn.microsoft.com/pt-br/windows/win32/api/handleapi/nf-handleapi-closehandle)**
- **[AllocConsole](https://learn.microsoft.com/pt-br/windows/console/allocconsole)**
- **[freopen_s](https://learn.microsoft.com/pt-br/cpp/c-runtime-library/reference/freopen-s-wfreopen-s?view=msvc-170)**
- **[GetModuleHandle](https://learn.microsoft.com/en-us/windows/win32/api/libloaderapi/nf-libloaderapi-getmodulehandlea)**
- **[GetAsyncKeyState](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-getasynckeystate)**
- **[fclose](https://learn.microsoft.com/pt-br/cpp/c-runtime-library/reference/fclose-fcloseall?view=msvc-170)**
- **[FreeConsole](https://learn.microsoft.com/pt-br/windows/console/freeconsole)**
- **[FreeLibraryAndExitThread](https://learn.microsoft.com/en-us/windows/win32/api/libloaderapi/nf-libloaderapi-freelibraryandexitthread)**
