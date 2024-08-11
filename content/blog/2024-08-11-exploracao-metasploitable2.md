---
title: "Exploração de vulnerabilidades em ambientes virtuais"
date: 2024-08-11
published: false
description: "Exploração de vulnerabilidades da máquina Metasploitable 2."
categories:
  - "Easy"
tags:
  - "Redes"
images: []
authors: 
  - gbrods
  - brenoppr
---

## Introdução

Neste post, vamos abordar como identificar e explorar algumas vulnerabilidades comuns encontradas na máquina vulnerável Metasploitable 2, utilizando ferramentas e métodos que são amplamente aplicáveis em cenários reais.

## O que é o Metasploitable 2

Metasploitable 2 é uma máquina virtual intencionalmente vulnerável, desenvolvida pela Rapid7, projetada para ser uma plataforma de prática para profissionais e entusiastas de segurança cibernética. Ela simula um ambiente de rede com várias vulnerabilidades conhecidas, permitindo que os usuários testem e aprimorem suas habilidades em pentest sem comprometer sistemas reais. Essa máquina contém diversos serviços mal configurados e softwares desatualizados, representando um campo de treino ideal para aprender a identificar e explorar falhas de segurança de forma controlada.

<!-- 

![Figura 1](/images/2024-01-13-1.png) // Imagem do metasploitable na VM 

**Figura 1:** Diferenças entre os cabeçalhos do IPv4 e IPv6.<br> *Fonte: https://www.networkacademy.io/ccna/ipv6/ipv4-vs-ipv6*

-->

Um guia para a instalação do ambiente do Metasploitable 2 pode ser encontrado aqui (inserir link guia de instalação)

## Exploração inicial: Enumeração de portas

A exploração inicial de qualquer sistema vulnerável começa com a enumeração de portas, uma etapa essencial para identificar quais serviços estão em execução e, potencialmente, vulneráveis. Para isso, utilizamos o Nmap (Network Mapper), uma ferramenta poderosa e amplamente utilizada em testes de penetração. O Nmap permite mapear a rede e descobrir portas abertas, além de coletar informações detalhadas sobre os serviços, versões e sistemas operacionais em execução. Essas informações são cruciais para planejar as etapas subsequentes da exploração, pois ajudam a direcionar os esforços para vulnerabilidades específicas que podem ser exploradas com sucesso.

INSERIR IMAGEM DO NMAP

Podemos ver diversas portas abertas, cada uma com um serviço diferente que pode ou não ser vulnerável. Neste post, não abordaremos todas as vulnerabilidades, mas podemos imediatemente abordar algumas que nos saltam os olhos

## Bindshell
## vdFTPd
## UnrealRCD
## Senhas e quebra de Hash

## Introdução ao Metasploit: Explorando Serviços Vulneráveis

Até o momento, utilizamos diversas ferramentas para mapear e interagir com os serviços ativos na máquina. Embora essas ferramentas individuais sejam poderosas, há um framework que consolida várias dessas capacidades em uma única plataforma: o Metasploit, que contém centenas de módulos para exploração de vulnerabilidades.

A seguir, vamos explorar o módulo que lida com servidores FTP vulneráveis para verificar se o serviço FTP que encontramos permite acesso anônimo, um vetor de ataque comum em sistemas mal configurados.

## Anonymous FTP

Abc

## Considerações finais 

## Referências Bibliográficas

3CX. **What are SIP methods - requests and responses?**. Disponível em: https://www.3cx.com/pbx/sip-methods/

Worknets. **Lista de códigos de resposta SIP**. Disponível em: https://www.worknets.com.br/2020/04/10/lista-de-codigos-de-resposta-sip/