---
layout: post
title: Resumo sobre as características e o funcionamento do Docker - Parte I
description: "Uma introdução ao funcionamento do Docker, sem referências externas."
tags: [Docker, Linux]
date: 2017-10-08
author: thiagowfx
published: 2017-10-08
---

O [docker](https://www.docker.com/) é um meio termo entre uma máquina virtual (= que roda um sistema guest (convidado) dentro de um sistema host (hospedeiro)) e um sistema rodando simultaneamente e concorrentemente com outro. Em primeiro lugar, ele é bem mais leve e mais rápido do que uma máquina virtual completa, pois reutiliza vários sub-sistemas do sistema host (diferentemente de uma máquina virtual, que tipicamente "emula" todos os aspectos de hardware e de software do guest), tal como o kernel. Em segundo lugar, assim como uma máquina virtual, tudo o que roda no docker está, a princípio (e por padrão) isolado do sistema host. Isso é feito por debaixo dos panos através de tecnologias existentes no kernel do Linux tais como cgroups e namespaces; uma consequência disso é que só é possível rodar o docker em sistemas relativamente recentes (por exemplo, qualquer Ubuntu 14.04 para cima já está de bom tamanho).

![Comparação entre as arquiteturas de máquina virtual e do docker (containers). Créditos da imagem: http://patg.net/containers,virtualization,docker/2014/06/05/docker-intro/](http://patg.net/assets/container_vs_vm.jpg)


Comparação entre as arquiteturas de máquina virtual e do docker (containers). Créditos da imagem: http://patg.net/containers,virtualization,docker/2014/06/05/docker-intro/


Da mesma forma (analogamente) que um programa de máquinas virtuais (como o VirtualBox) opera com arquivos de máquina virtual, o docker opera com **imagens**. Uma imagem é uma espécie de mini-sistema Linux que pode rodar em um **container** no docker. Por exemplo, existem imagens do Ubuntu, do Alpine Linux, do Arch Linux, do CentOS, e assim por diante. Imagens não precisam ser apenas distribuições de linux; por exemplo, também existem imagens de python, de ruby, de nodejs. A arquitetura é tal que é bastante fácil criar novas imagens a partir de imagens pré-existentes; por exemplo, você pode pegar a imagem do Ubuntu e instalar um nginx nela, e depois criar uma nova imagem com esse novo conjunto (Ubuntu + nginx), e dar o nome de "nginx" a ela.


Cada nova **camada** que você adiciona a uma imagem é chamada de **layer**. No exemplo anterior, tomamos uma imagem do Ubuntu e adicionamos um **layer** de nginx a ele.


![Exemplo de camadas no docker. Créditos da imagem: http://blog.bigstep.com/developers-love-docker/](https://cdn-images-1.medium.com/max/800/1*rCBBYVVJBlnNdo2f3OVyWw.png)


Exemplo de camadas no docker. Créditos da imagem: http://blog.bigstep.com/developers-love-docker/


Imagens são criadas a partir de Dockerfiles, que são arquivos que possuem instruções de como gerar imagens. Um exemplo típico de instrução para adicionar no Dockerfile é `apt-get install <package>`. Para rodar as imagens no docker, containers são criados. Um container é nada mais do que um ambiente / instância de execução a partir de uma imagem.


Para saber mais
---------------


* https://blog.docker.com/
* https://docs.docker.com/get-started/
