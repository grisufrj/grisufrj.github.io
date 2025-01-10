---
title: "Forense Digital: Aquisição de logs e memória volátil/não volátil"
date: 2024-08-09
description: Resumo de ferramentas para forense digital, como logs e aquisição de imagem
categories:
  - easy
tags:
  - forense
author: pedrofm
---
## Introdução

Quando um incidente de segurança ocorre, é importante obter a imagem de memória do sistema pós incidente, para poder melhor analisar o que ocorreu e como ele foi invadido.
## Aquisição de Imagens

É importante diferenciar o conceito de cópia e imagem, especialmente para memória não volátil. A cópia do armazenamento de um dispositivo é a clonagem de todos seus arquivos ainda existentes (ou seja, arquivos deletados não serão copiados, pois o SO não pode mais os reconhecer).

Por outro lado, a imagem faz um clone bit a bit do HD/SSD inteiro, incluindo as partes apagadas anteriormente pelo SO. Ou seja, a imagem rende mais informações, e pode mostrar dados antes removidos.
### Memória Não Volátil 

A memória não volátil é a informação que se mantém ao desligar a máquina. Pode envolver o que está armazenado no HD/SSD, os logs do sistema, a chaves de registros ou arquivos prefetch.
#### HD / SSD

Nota: por causa da diferença no funcionamento, arquivos apagados em um HD tem mais facilidade de serem recuperados do que um SSD.
Como SSDs funcionam a partir da mudança do estado de elétrons (diferente do HD, que escreve em um disco), recuperar dados apagados em um SSD é muito mais difícil
#### Criando uma imagem do HD/SSD

Para sistemas Windows, pode-se usar o FTK Imager. Para Linux, o Guymager (do sistema CAINE) pode criar uma imagem.

O FTK Imager é uma ferramenta de forense digital focada pro Windows, e o CAINE é uma distribuição Linux especializada para forense digital
#### Log de eventos:

Para o sistema operacional, o log de eventos é um registro de todas as coisas que ocorreu com o sistema, como processos e conexões iniciadas.
Para sistemas Windows, eles são armazenados no diretório C:\\Windows\\System32\\winevt\\Logs, enquanto que em sistemas Linux a maioria deles são armazenados em \\var\\log.
Para consultar logs no linux, podemos usar o journalctl, que unifica todos os logs de sistema.
#### Chave de Registros:

A Chave de Registro é uma coleção de informações sobre partes do sistema como hardware, software, drivers, configurações de usuário e outras configurações do sistema operacional e da máquina.

Para o Windows, as chaves de registros são armazenadas no diretório C:\\Windows\\System32\\config. Além disso, cada usuário tem seu próprio registro em seu diretório chamado NTUSER.DAT, que guarda suas configurações pessoais.

Para o Linux, softwares do Linux normalmente armazenam suas configurações em um arquivo de texto individual, configurações da máquina são armazenadas no diretório /etc e configurações de usuários normalmente estão localizadas no diretório /home de cada um, normalmente em arquivos escondidos.
#### Prefetch:

O Prefetch é um sistema no Windows que carrega mais rapidamente os programas que são mais usados, armazenando o arquivo .pf na pasta.
A vantagem desse diretório para forense digital é a possibilidade de visualizar se há algum programa indevido sendo muito utilizado, podendo ser um malware.

Todos esses dados também podem ser copiados com um aplicativo como Triage ou kape.

### Memória Volátil

No caso, a memória volátil seria a memória que é apagada ao desligar a máquina (RAM, conexões de rede, processos ativos).

#### Utilizando FTK Imager (Windows)

O FTK Imager possui também a função de adquirir a memória volátil de uma máquina.
Depois de terminar, ele vai gerar um arquivo .mem.
#### Utilizando winpmem (Windows)

Utilizando o winpmem pela linha de comando, podemos criar um arquivo .raw com as informações da memória.
	Comando: ``winpmem_64.exe <nome do arquivo>.raw``

#### Utilizando AVML (Adquire Volatile Memory for Linux)

Para sistemas Linux, o AVML pode criar uma imagem da memória do sistema.

#### Leitura de memória não volátil

Depois de adquirir a imagem da memória, o volatility pode ser usado para ler a memória e identificar processos, conexões de rede e outras partes da memória não volátil.

## Referências:

https://www.mentebinaria.com.br/courses/course/2-dfir/

https://www.cyberciti.biz/faq/linux-log-files-location-and-how-do-i-view-logs-files/

https://linuxhandbook.com/journalctl-command/

https://www.forensicfocus.com/articles/windows-registry-analysis-101/

https://meuwindows.com/o-que-e-pasta-prefetch-para-que-serve/

Volatility: https://github.com/volatilityfoundation/volatility3

winpmem:  https://github.com/Velocidex/WinPmem/releases

CAINE: https://www.caine-live.net/

Kape: https://www.sans.org/tools/kape/

AVML: https://github.com/microsoft/avml



