---
title: "Softphones e telefonia pela Internet"
date: 2024-01-13
published: 2024-01-13
description: "Discussão de alguns protocolos da camada TCP/IP e configuração de um softphone usando VoIP."
categories:
  - "Easy"
tags:
  - "Redes"
images: []
author: gbrods
---

## Introdução

Você já ouviu falar de VoIP e softphones? Neste post, vamos aprender um pouco sobre os fundamentos da telefonia pela Internet e ver na prática a configuração de um software para efetuar chamadas telefônicas que se aproveita da linha fixa telefônica que vem junto do seu modem de internet.

Antigamente, para realizar chamadas telefônicas, era comum a utilização de um telefone fixo com fio. Para isso, era necessário se conectar à rede PSTN (_Public Switched Telephone Network_ ou Rede Telefônica Pública Comutada), uma infraestrutura mundial de redes de telefonia pública que utiliza cabos de cobre ou fibra óptica para transmitir dados de voz analógicos.

Com o avanço da Internet, surgiram melhores alternativas para a telefonia convencional. Uma delas é o VoIP (_Voice Over Internet Protocol_ ou Voz sobre Protocolo de Internet), uma tecnologia que viabiliza a transmissão de voz em forma de pacotes de dados. Esses pacotes são enviados pela Internet e permitem que duas máquinas troquem informações entre si. 

## Principais protocolos utilizados no VoIP

Para entendermos melhor as etapas dessa comunicação, vamos estudar os principais protocolos utilizados em uma conexão pela internet.

### Pilha de protocolos da Internet

A pilha TCP/IP é um conjunto de protocolos que definem como os dispositivos se comunicam na internet. Ela é composta por cinco camadas, que são:

- **Aplicação**: realiza a interface entre a aplicação e a rede, controla os serviços e protocolos utilizados pelas aplicações dos dispositivos finais e permite a comunicação entre eles. Alguns dos principais protocolos são: HTTP (HyperText Transfer Protocol), DNS (Domain Name System) e SIP (Session Initiation Protocol);

- **Transporte**: responsável por realizar efetivamente os serviços de comunicação de dados e entrega confiável dos pacotes entre a origem e o destino (fim a fim). Os principais protocolos são: TCP (Transmission Control Protocol), UDP (User Datagram Protocol) e, para nosso estudo, o RTP (Real-time Transport Protocol);

- **Rede**: faz o roteamento dos pacotes de dados e realiza a comunicação entre os hosts conectados à rede, independentemente de sua localização física. Alguns dos principais protocolos são: IP (Internet Protocol) e ICMP (Internet Control Message Protocol);

- **Enlace**: camada responsável por realizar a conexão física entre os dispositivos de rede e permitir a transferência de dados entre dispositivos adjacentes (comunicação salto a salto). Alguns dos principais protocolos são: Ethernet e Wi-Fi;

- **Física**: lida diretamente com a transmissão física dos bits “no fio”.

Agora que estudamos a visão geral sobre a pilha TCP/IP, podemos focar nos principais protocolos e suas funções específicas para nosso estudo de VoIP.

### IP (Internet Protocol)

O protocolo IP é o alicerce da internet, responsável por encaminhar os pacotes até o destino e interconectar redes de diferentes tecnologias. Ele fornece endereçamento único para cada dispositivo na rede, permitindo a identificação e a entrega adequada dos pacotes. Embora seja crucial, o IP não garante entrega confiável ou ordenada dos dados. 

Existem duas principais versões do IP: o IPv4 e o IPv6. De forma resumida, o IPv4 utiliza endereços de 32 bits (2<sup>32</sup> endereços únicos), enquanto o IPv6 emprega endereços de 128 bits (2<sup>128</sup> endereços únicos), oferecendo maior quantidade de endereços para acomodar a crescente demanda por dispositivos conectados à internet.

![Figura 1](/images/2024-01-13-1.png)

**Figura 1:** Diferenças entre os cabeçalhos do IPv4 e IPv6.<br> *Fonte: https://www.networkacademy.io/ccna/ipv6/ipv4-vs-ipv6*

### TCP (Transport Control Protocol):

O protocolo TCP, por sua vez, oferece uma entrega confiável de dados, pois estabelece uma conexão entre os dispositivos comunicantes e garante que os pacotes cheguem na ordem correta e sem perdas. Caso ocorram problemas durante a transmissão, o TCP retransmite os pacotes afetados.

![Figura 2](/images/2024-01-13-2.png)

**Figura 2:** Cabeçalho do protocolo TCP.<br> *Fonte: https://intronetworks.cs.luc.edu/current2/uhtml/tcpA.html*

### UDP (User Datagram Protocol):

Já o protocolo UDP adota uma abordagem não orientada a conexão, priorizando a velocidade na entrega dos dados. Embora não garanta a entrega confiável e possa resultar em perdas de pacotes, sua simplicidade o torna ágil e eficaz para transmissões em tempo real, como o VoIP. Tal característica é essencial para manter a sincronização entre os participantes, embora possa resultar em algumas inconsistências de qualidade devido à falta de correção de erros.

![Figura 3](/images/2024-01-13-3.png)

**Figura 3:** Cabeçalho do protocolo UDP.<br> *Fonte: https://intronetworks.cs.luc.edu/current2/uhtml/udp.html*

### SIP (Session Initiation Protocol):

O SIP é bastante importante no VoIP, pois é um protocolo de sinalização usado para criar, controlar e finalizar sessões das chamadas telefônicas. De forma resumida, o SIP define dois tipos de mensagens: REQUEST e RESPONSE.

A mensagem REQUEST é enviada de um cliente a um servidor e pode ter vários métodos, que representam uma ação necessária. Existem 14 delas, mas as principais que veremos na demonstração são:
* **PUBLISH**: publica o estado de um evento;
* **INVITE**: é utilizado quando se deseja convidar um novo participante para uma sessão;
* **REGISTER**: envia informações sobre identificação e localização do usuário, como hostname e endereço IP;
* **ACK**: uma mensagem com este método é enviada por um usuário que mandou um INVITE para avisar que uma mensagem do tipo RESPONSE foi recebida;
* **BYE**: é utilizado para encerrar a participação numa sessão.

Já a mensagem RESPONSE é usada para indicar o resultado de uma solicitação que foi realizada durante a sessão. Existem vários códigos de resposta SIP, contudo, os mais importantes para nosso estudo são:

| 1XX – Respostas Provisórias | Descrição |
| -------- | ------- |
| 100 Trying | Indica que o roteador mais próximo do servidor recebeu uma tentativa de resposta. |
| 180 Ringing | Indica que o dispositivo do usuário recebeu uma solicitação e está alertando o destino (ou seja, o telefone do destino está tocando). |

| 2XX – Respostas bem sucedidas | Descrição |
| -------- | ------- |
| 200 OK  | A solicitação foi completada com sucesso. |

| 3XX – Respostas de redirecionamento | Descrição |
| -------- | ------- |
| 301 Moved Permanently  | O usuário não pode mais ser localizado no endereçamento que foi enviado a chamada. |
| 302 Moved Temporarily  | A chamada não pode ser completada porque o endereço solicitado está temporariamente desabilitado, fora de serviço ou expirou. |

| 4XX – Respostas de falha | Descrição |
| -------- | ------- |
| 400 Bad Request  | A chamada não pode ser completada devido a um erro como número digitado errado, algum bloqueio da operadora ou até mesmo um erro interno na rede do cliente. |
| 401 Unauthorized  | O pedido não pode ser completado pois não foi possível autenticar o usuário. |
| 403 Forbidden  | O servidor recebeu a solicitação, porém está se negando a completar a chamada, devido a algum erro entre as pontas em decorrência de uma configuração de rota de entrada ou de saída errada, e até mesmo a um bloqueio em uma das pontas. |
| 480 Temporarily Unavailable  | Local solicitado se encontra indisponível no momento. |
| 486 Busy Here  | Destino ocupado. |
| 487 Request Terminated  | Solicitação cancelada pelo destino ou por algum agente entre a fonte do pedido e o destino.  |
| 488 Not Acceptable Here  | Alguns aspectos da descrição da sessão ou a solicitação URI não são aceitas. |

| 5XX – Erros de falha do servidor | Descrição |
| -------- | ------- |
| 500 Server Internal Error  | Devido a uma condição especial o servidor não pôde completar o pedido. |
| 503 Service Unavailable  | O servidor está em manutenção ou indisponível temporariamente. |

| 6XX – Respostas de Falhas Globais | Descrição |
| -------- | ------- |
| 606 Not Acceptable  | O agente do usuário fez contato com sucesso, porém alguns aspectos da descrição da sessão não são aceitos. |

*Fonte: https://www.worknets.com.br/2020/04/10/lista-de-codigos-de-resposta-sip/*

Portanto, vimos que o SIP (Session Initiation Protocol) estabelece e gerencia a sessão de comunicação. Agora, vamos ver como os dados dessa comunicação são entregues. 

### RTP (Real-time Transport Protocol):

O RTP é um protocolo de transporte que é implementado na camada de aplicação, é executado sobre o UDP e realiza o transporte fim-a-fim em tempo real de pacotes de mídia (como áudio e vídeo) durante uma sessão de comunicação. 

Juntamente ao RTP, existe um outro protocolo que serve para monitorar a qualidade de serviço: o protocolo de controle RTCP (Real Time Control Protocol). Ele não transporta dados, pois sua principal função é realizar controle de fluxo e congestionamento para os participantes de uma sessão.

## Visão geral

Utilizando tudo que aprendemos até o momento, veja um exemplo de uma chamada VoIP sendo estabelecida entre dois indivíduos.

![Figura 4](/images/2024-01-13-4.png)

**Figura 4:** Exemplo de uma chamada VoIP sendo estabelecida.<br> *Fonte: https://www.nextiva.com/blog/sip-protocol.html*

Falta apenas um conceito para irmos à parte prática do post, que é: como a fala é convertida em um sinal digital?

## Codificadores e decodificadores

Um CODED VoIP é um dispositivo de hardware ou software que é capaz de codificar e decodificar um sinal. De forma simplificada, um CODEC converte um sinal de voz analógico em sua forma digital. Em seguida, ele converte o sinal digital comprimido de volta para sua forma analógica original.

Veja abaixo alguns tipos de CODECS VoIP que são utilizados:

![Figura 5](/images/2024-01-13-5.png)

**Figura 5:** Exemplos de CODECs utilizados no VoIP. <br>*Fonte: SUDRÉ, Gilberto. Entendendo a Tecnologia VoIP.*

Vale ressaltar que o G.711 é o mesmo CODEC utilizado pela rede PSTN. Existem duas variações:

* **G.711 A_law** : utilizado em países fora da América do Norte. Também é utilizado para ligações telefônicas internacionais.  
* **G.711 u_law** : utilizado principalmente na América do Norte.

## Mão na massa!

Finalmente, estamos prontos para configurar o softphone. É preciso que você:

* **Tenha um plano de internet com telefone fixo** (seu modem deve ter uma entrada RJ-11)
* **Tenha acesso ao login de administrador de seu modem** (admin, userAdmin, AdminGPON ... ). Se você não souber o endereço IP ou o login para acessar as informações do seu modem, verifique as orientações que estão atrás do aparelho.

![Figura 6](/images/2024-01-13-6.png)

* **Tenha acesso às informações de voz da sua linha** (endereço ip do `Soft Switch` e `User Agent`, bem como seu número de telefone fixo `Phone Number`). Entre nas informações de seu modem, procure uma aba equivalente a _Voice Information_ e anote as informações.

![Figura 7](/images/2024-01-13-7.png)

* Se houver, mude a prioridade da VLAN (Virtual Local Area Network) correspondente ao VoIP para `0` (quanto menor o valor de `VLAN PRI`, maior é a prioridade dos pacotes de VoIP para o modem). Isso é muito importante para a qualidade de suas chamadas.

![Figura 8](/images/2024-01-13-8.png)

* **Instale um softphone** em seu computador (irei utilizar o MicroSIP no sistema operacional Windows). Ao abrir o MicroSIP, clique na seta do canto superior direito e clique em `Adicionar Conta`. 

![Figura 9](/images/2024-01-13-9.png)

* **Preencha as informações** que você anotou anteriormente.

* Em `Servidor SIP`, escreva o endereço IP correspondente ao `User Agent`.
* Em `Proxy SIP`, escreva o endereço IP correspondente ao `Soft Switch`.
* Em `Domínio`, veja qual é o endereço de seu provedor de serviço. No meu caso, é `ims.oi.net.br:5060`. Note que a porta 5060 é a porta padrão do protocolo SIP.
* Em `Usuário` e `Login`, escreva: `+telefone@domínio`.
* Em `Senha`, escreva seu número de telefone sem o DDD. Exemplo: `40028922`
* Em `Transporte`, escolha `UDP+TCP`.
* Clique em Salvar.

![Figura 10](/images/2024-01-13-10.png)

Agora, na tela inicial do MicroSIP, clique novamente na seta superior direita e selecione `Configurações`. Após abrir, localize os codecs disponíveis, utilize as setas e habilite apenas o codec `G.711 A-law`. Após isso, clique em salvar.

![Figura 11](/images/2024-01-13-11.png)

Pronto! Agora, nosso softphone está configurado e pronto para realizar chamadas.

![Figura 12](/images/2024-01-13-12.png)

Com a ajuda do Wireshark (programa de análise de tráfego de rede), vamos ligar para o número `40028922` e verificar as requisições relacionadas aos protocolos SIP e RTP.

![Figura 13](/images/2024-01-13-13.png)

Início da conexão (observe as mensagens INVITE, TRYING, RINGING e os pacotes RTP)

![Figura 14](/images/2024-01-13-14.png)

Fim da conexão (observe as mensagens BYE e OK)

Espero que este post tenha te ajudado a entender um pouco mais sobre os protocolos da Internet e os mecanismos que estão relacionados à tecnologia VoIP.

## Referências Bibliográficas

3CX. **What are SIP methods - requests and responses?**. Disponível em: https://www.3cx.com/pbx/sip-methods/

Worknets. **Lista de códigos de resposta SIP**. Disponível em: https://www.worknets.com.br/2020/04/10/lista-de-codigos-de-resposta-sip/

BERNAL, Huber. **Telefonia IP**. Disponível em: https://www.teleco.com.br/tutoriais/tutorialtelip/default.asp

Canal do Toth. **Curso Básico de VoIP & SIP**. Disponível em:
https://www.youtube.com/playlist?list=PLYWlOBwQPyDqkMW82OAgamy2DP3DAZDV1

LAGOIA, Paulo. **Push-to-Talk no Celular II: Protocolos**. Disponível em: https://www.teleco.com.br/tutoriais/tutorialpushtotalk2/default.asp

Nextiva. **What Is Session Initiation Protocol (SIP) & How Does It Work?**. Disponível em: https://www.nextiva.com/blog/sip-protocol.html

SUDRÉ, Gilberto. **Entendendo a Tecnologia VoIP**. Disponível em: https://pt.slideshare.net/gilsudre/entendendo-a-tecnologia-voip-presentation