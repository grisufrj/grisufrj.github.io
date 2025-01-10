---
title: "Redes: IDS e IPS"
date: 2024-01-08
published: 2024-01-08
description: "Principais diferenças entre sistemas de prevenção e detecção de intrusões"
categories:
  - "Easy"
tags:
  - "Redes"
images: ["/images/2024-01-06-9.png"]
author: jhaysonj
---

## Introdução

O objetivo dessa postagem é introduzir o leitor às principais diferenças entre IDS (sistemas de detecção a intrusões) e IPS (sistemas de prevenção a intrusões), destacando a diferença de desempenho e aplicação desses dois sistemas. 

## O que é o IDS?

O IDS (Sistema de Detecção de Intrusões) é projetado para identificar atividades suspeitas ou comportamentos anômalos na rede. O IDS monitora o tráfego em tempo real em busca de padrões que possam indicar uma ameaça.

## O que é o IPS?

Por outro lado, o IPS (Sistema de Prevenção de Intrusões) vai além da detecção, agindo proativamente para bloquear ou impedir atividades maliciosas assim que são identificadas.
Ação Tomada:

## Difenças de ação
IDS: Um IDS alerta os administradores sobre possíveis ameaças, mas não toma medidas diretas para impedir ou interromper o ataque.

IPS: Ao contrário, um IPS age automaticamente para bloquear ou prevenir atividades maliciosas assim que são detectadas, contribuindo para uma resposta rápida e eficaz.

## Modo de Operação:
IDS: Operando em modo passivo, um IDS monitora o tráfego sem interferir diretamente, fornecendo insights valiosos para análise de segurança.

IPS: Em contrapartida, um IPS opera em modo ativo, intervindo ativamente para evitar que ameaças se materializem em ataques reais.

## Implementação:
IDS: Geralmente, os IDS são implementados em ambientes em que a ênfase está na detecção de intrusões, sem a necessidade de intervenção imediata.

IPS: Já os IPS são preferencialmente implementados em ambientes onde a prevenção é crucial, como em redes corporativas que requerem uma resposta rápida a ameaças.

## Desempenho:
IDS: Como os IDS focam na análise de padrões e alertas, eles podem ser menos suscetíveis a falsos positivos, mas, por outro lado, podem ser mais lentos na resposta a ameaças reais.

IPS: Enquanto isso, os IPS podem ser mais propensos a falsos positivos devido à sua natureza proativa, mas oferecem uma resposta mais rápida e eficaz.

Nos exemplos acima, podemos entender falsos positivos quando o sistema categoriza uma ação como intrusiva, quando na verdade não é.



## Apresentando o Snort
O snort é um programa que funciona como IDS, para isso, utiliza um conjunto de regras pré estabelecidas para gerar os alertas, essas regras ficam no diretório `/etc/snort/rules`. Além disso, podemos usar o snort para gerar logs, o arquivo de log gerado pelo snort fica localizado em `/var/log/snort/alert`.

**Comando**
```
snort -A fast -q -h 192.168.0.0/24 -c /etc/snort/snort.conf
```

**Explicando o comando:**

Esse comando inicia o Snort em modo de alerta rápido, no modo silencioso, monitorando a rede 192.168.0.0/24 e utilizando o arquivo de configuração localizado em /etc/snort/snort.conf. O Snort estará operando com base nas regras e configurações definidas nesse arquivo, alertando sobre possíveis atividades suspeitas na rede monitorada.

**Explicando as flags:**
- -A fast: Especifica o modo de alerta rápido. Nesse modo, o Snort imprime apenas informações essenciais sobre alertas, tornando a saída mais concisa.

- -q: Ativa o modo silencioso (quiet). Isso reduz a quantidade de informações extras que o Snort normalmente exibiria durante a execução.

- -h 192.168.0.0/24: Define a rede alvo para monitoramento. Neste caso, o Snort estará monitorando a rede com o endereço IP de 192.168.0.0 e máscara de sub-rede 255.255.255.0 (ou /24).

- -c /etc/snort/snort.conf: Especifica o arquivo de configuração a ser utilizado. Neste caso, o arquivo de configuração está localizado em /etc/snort/snort.conf. O arquivo de configuração é crucial, pois contém as regras e configurações específicas do Snort.

Vale destacar que o host vai de acordo com a rede que queremos configurar, no meu caso `192.168.0.0/24`.

IP da maquina com snort: 192.168.0.8

### Criando nossa primeira Regra

Podemos criar nossa própria regra e salvar no diretório de regras `/etc/snort/rules`. 

As regras criadas seguem o seguinte padrão:
![Figura 1](/images/2024-01-19-1.png)
Para consultar as possíveis opções acesse a [Documentação Snort](https://docs.snort.org/rules/options/)


Dentro do arquivo gris.rules colocamos a regra abaixo e salvamos no diretório `/etc/snort/rules`
```
alert tcp any any -> 192.168.0.19 any (msg: "Possivel Port Scanning"; sid:1000001; rev:001;)
```


Vamos editar o arquivo de regras do snort para incluir a regra que criamos
```
sudo nano /etc/snort/snort.conf
```

**OBS:** Se executarmos o comando acima sem super usuário não teremos permissão de leitura do arquivo, para conseguir acessar editar o arquivo corretamente, utilize o `sudo`.

No trecho em que há o include de várias regras `# Step #7: Customize your rule set`, incluimos a regra `gris.rules` que acabamos de criar.

```
include $RULE_PATH/gris.rules
```

### Log gerado
No comando abaixo, iniciamos a captura de log na máquina alvo
```
snort -A console -q -h 192.168.0.0/24 -c /etc/snort/snort.conf
```

Na máquina atacante, realizando port scan com uso do nmap
```
nmap -v -sS --top-ports=10 --open -Pn 192.168.0.8
```

Com isso, registramos o log gerado a partir do port scan realizado
```
01/09-20:01:35.520242  [**] [1:1000001:1] Possível Port Scanning [**] [Priority: 0] {TCP} 192.168.0.7:36787 -> 192.168.0.8:139
01/09-20:01:35.520346  [**] [1:1000001:1] Possível Port Scanning [**] [Priority: 0] {TCP} 192.168.0.7:36787 -> 192.168.0.8:23
01/09-20:01:35.520362  [**] [1:1000001:1] Possível Port Scanning [**] [Priority: 0] {TCP} 192.168.0.7:36787 -> 192.168.0.8:443
01/09-20:01:35.520418  [**] [1:1000001:1] Possível Port Scanning [**] [Priority: 0] {TCP} 192.168.0.7:36787 -> 192.168.0.8:445
01/09-20:01:35.520456  [**] [1:1000001:1] Possível Port Scanning [**] [Priority: 0] {TCP} 192.168.0.7:36787 -> 192.168.0.8:3389
01/09-20:01:35.520474  [**] [1:1000001:1] Possível Port Scanning [**] [Priority: 0] {TCP} 192.168.0.7:36787 -> 192.168.0.8:25
01/09-20:01:35.520500  [**] [1:1000001:1] Possível Port Scanning [**] [Priority: 0] {TCP} 192.168.0.7:36787 -> 192.168.0.8:22
01/09-20:01:35.520535  [**] [1:1000001:1] Possível Port Scanning [**] [Priority: 0] {TCP} 192.168.0.7:36787 -> 192.168.0.8:80
01/09-20:01:35.520562  [**] [1:1000001:1] Possível Port Scanning [**] [Priority: 0] {TCP} 192.168.0.7:36787 -> 192.168.0.8:110
01/09-20:01:35.520576  [**] [1:1000001:1] Possível Port Scanning [**] [Priority: 0] {TCP} 192.168.0.7:36787 -> 192.168.0.8:21
01/09-20:01:35.520624  [**] [1:1000001:1] Possível Port Scanning [**] [Priority: 0] {TCP} 192.168.0.7:36787 -> 192.168.0.8:443
01/09-20:01:35.520633  [**] [1:1000001:1] Possível Port Scanning [**] [Priority: 0] {TCP} 192.168.0.7:36787 -> 192.168.0.8:80
01/09-20:01:35.520661  [**] [1:1000001:1] Possível Port Scanning [**] [Priority: 0] {TCP} 192.168.0.7:36787 -> 192.168.0.8:21

```
**Falso positivo**
Obviamente os testes mostrados tratam-se de uma simplificação, já que, segundo o nosso IDS, todas as conexões TCP seriam erroneamente categorizadas como Port Scan.

## Regra 2
Podemos criar uma regra mais realistica. De forma análoga ao que foi feito na regra 1, podemos adicionar uma nova regra no arquivo `/etc/snort/rules/gris.rules`
```
alert tcp any any -> 192.168.0.8 any (msg: "Possível Port Scanning";sid:1000001;rev:001;)
alert tcp any any -> 192.168.0.8 80 (msg: "Acesso ao arquivo robots.txt";content:"robots.txt";sid:1000002; rev:001;)
```

No comando abaixo, iniciamos a captura de log na máquina alvo
```
snort -A console -q -h 192.168.0.0/24 -c /etc/snort/snort.conf
```

Com isso, na máquina atacante ao acessarmos o endereço 192.168.0.8/robots.txt registramos o log
```
01/09-20:24:17.287207  [**] [1:1000002:1] Acesso ao arquivo robots.txt [**] [Priority: 0] {TCP} 192.168.0.7:42886 -> 192.168.0.8:80
```
Ainda que o endereço `192.168.0.8/robots.txt` não seja válido, apenas pelo fato de realizarmos a requisição para este endereço o IDS registra o log.


## Referências

- Documentação 1 do Snort <https://docs.snort.org/intro>
- Documentação 2 do Snort <http://manual-snort-org.s3-website-us-east-1.amazonaws.com/node1.html>
- GitHub Snort <https://github.com/snort3/snort3#documentation>
