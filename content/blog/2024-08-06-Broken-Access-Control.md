---
title: "Broken Access Control: Entendendo Vulnerabilidades de Controle de Acesso"
date: 2024-06-08
published: 2024-06-08
description: "Um guia básico para entender Vulnerabilidades de Controle de Acesso"
tags:
  - "web"
category: "easy"
images: ["/images/2024-01-08-12.png"]
author: richardpiccoli
---

## O que é Broken Access Control?

Broken Access Control (BAC), ou Controle de Acesso Quebrado, é uma falha de segurança onde os mecanismos de controle de acesso de um sistema não são implementados ou configurados corretamente, permitindo que usuários mal-intencionados acessem recursos que deveriam estar restritos. Isso pode incluir a visualização de informações confidenciais, a modificação de dados ou a execução de operações privilegiadas.

## O que é OWASP Top 10?

O OWASP Top 10 é uma lista publicada pela Open Web Application Security Project (OWASP) que destaca as dez vulnerabilidades mais críticas de segurança em aplicações web. Este documento é amplamente reconhecido e utilizado como um padrão de referência para o desenvolvimento seguro de software. A lista é atualizada periodicamente para refletir as novas ameaças e tendências de segurança.

![Figura 1](/images/2024-06-08-11.png)

## Tipos de Broken Access Control:

- **Escalonamento Vertical de Privilégios:** Quando um usuário com poucos privilégios ganha acesso a funções ou dados de alto privilégio, muitas vezes devido a um controle de acesso baseado em função (RBAC) inadequado ou princípios de menor privilégio mal implementados.
- **Escalonamento Horizontal de Privilégios:** Quando um usuário ganha acesso a recursos ou dados que não deveria, mesmo estando no mesmo nível de privilégio, muitas vezes devido a um controle de acesso baseado em atributos (ABAC) inadequado ou má segregação de dados.
- **Burlar Controle de Acesso:** Quando um invasor encontra uma maneira de contornar os mecanismos de controle de acesso, como explorando vulnerabilidades na lógica de autenticação ou autorização.
- **Referência Direta de Objeto Insegura (IDOR):** Quando uma aplicação expõe dados ou funcionalidades sensíveis através de referências diretas a objetos, permitindo acesso não autorizado.

![Figura 1](/images/2024-06-08-05.png)

## Exemplos Comuns:

- **APIs ou endpoints desprotegidos** que permitem acesso não autorizado a dados sensíveis.
- **Listas de controle de acesso (ACLs)** ou sistemas de controle de acesso baseado em função (RBAC) configurados de forma insegura.
- **Mecanismos de autenticação fracos ou ausentes**, como falta de limitação de taxa ou políticas de senha inadequadas.
- **Validação ou sanitização de entrada inadequadas**, permitindo que invasores manipulem decisões de controle de acesso.
- **Sistemas de controle de acesso configurados incorretamente ou desatualizados**, como permissões desatualizadas ou contas esquecidas.

## Vetores de Ataque:

- **Manipulação de API:** Manipular solicitações de API para acessar recursos ou dados não autorizados.
- **Manipulação de Parâmetros:** Modificar parâmetros de solicitação para burlar verificações de controle de acesso.
- **Sequestro de Sessão:** Roubar ou manipular sessões de usuário para obter acesso não autorizado.
- **Escalonamento de Privilégios:** Explorar vulnerabilidades para elevar privilégios e acessar recursos restritos.

## Detecção e Prevenção:

- **Auditorias de Segurança Regulares:** Realizar auditorias de segurança regulares para identificar e abordar vulnerabilidades de controle de acesso.
- **Implementação de Menor Privilégio**: Aplicar princípios de menor privilégio para limitar o acesso a recursos e dados sensíveis.
- **Autenticação Forte:** Implementar mecanismos de autenticação forte, como autenticação multifator (MFA) e limitação de taxa.
- **Validação de Entrada:** Validar e sanitizar a entrada do usuário para evitar a manipulação de decisões de controle de acesso.
- **Listas de Controle de Acesso (ACLs):** Implementar e revisar regularmente as ACLs para garantir o controle de acesso adequado.

## Ferramentas e Técnicas:

- **Burp Suite:** Uma ferramenta de teste de segurança de aplicações web que pode ajudar a identificar vulnerabilidades de controle de acesso quebrado.
- **ZAP (Zed Attack Proxy):** Um scanner de segurança de aplicações web de código aberto que pode detectar problemas de controle de acesso quebrado.
- **Ferramentas de Teste de Segurança de API:** Ferramentas como Postman, SoapUI ou API Scanner podem ajudar a testar o controle de acesso de APIs.
- **Testes Manuais:** Realizar testes manuais para identificar vulnerabilidades de controle de acesso quebrado, como tentando acessar recursos ou dados não autorizados.

## **Estudos de Casos**

## Facebook Data Breach (2019)

**Descrição do Ataque**

Em 2019, o Facebook sofreu um vazamento de dados devido a uma falha de Broken Access Control. Cerca de 419 milhões de registros de usuários foram expostos publicamente na internet. Estes registros incluíam números de telefone e identificações de usuários do Facebook.

- **Identificação da Vulnerabilidade:**
  A vulnerabilidade foi decorrente de uma API mal configurada que permitia o acesso a dados de usuários sem a devida autenticação ou autorização.
- **Exploração da Vulnerabilidade:**
  Atacantes exploraram a API do Facebook que permitia a busca de perfis através de números de telefone, o que resultou no acesso não autorizado a informações pessoais dos usuários.
- **Impacto do Ataque:**

  **Exposição de Dados Pessoais:** Números de telefone e identificações de milhões de usuários foram expostos.

  **Potencial para Fraudes:** Informações expostas poderiam ser utilizadas para campanhas de phishing, fraudes e ataques de engenharia social.

  **Perda de Confiança:** A confiança dos usuários no Facebook foi abalada, impactando negativamente a reputação da empresa.

**Medidas Corretivas**

- **Revisão de Controle de Acesso:**
  Implementação de verificações rigorosas para garantir que apenas usuários autenticados e autorizados possam acessar APIs sensíveis.
- **Auditoria e Monitoramento:**
  Auditoria contínua dos mecanismos de controle de acesso e monitoramento em tempo real para detectar e responder rapidamente a tentativas de acesso não autorizadas.
- **Educação e Treinamento:**
  Capacitação de desenvolvedores e administradores sobre as melhores práticas de segurança para evitar configurações incorretas.

**Conclusão do vazamento**

O caso do vazamento de dados do Facebook em 2019 ilustra claramente as consequências graves que podem resultar de um controle de acesso inadequado. Este incidente destaca a importância de implementar, revisar e monitorar constantemente os mecanismos de controle de acesso para proteger os dados e a integridade dos sistemas.

## Vazamento de Dados da First American Financial Corp (2019)

**Descrição do Ataque**

A First American Financial Corp, uma empresa americana de serviços financeiros imobiliários e de liquidação de hipotecas, enfrentou um dos maiores vazamentos de dados da história em 2019. Ben Shoval, um desenvolvedor imobiliário, descobriu que aproximadamente 885 milhões de arquivos contendo dados sensíveis de clientes desde 2003 estavam livremente disponíveis. Ele notificou a empresa sobre o problema.

**Identificação da Vulnerabilidade**

Este vazamento foi resultado de um erro humano. Em janeiro daquele ano, a equipe interna descobriu uma falha de Insecure Direct Object Reference (IDOR) durante um teste de penetração manual. Essa falha permitia que usuários acessassem informações privadas usando uma URL específica e alterando sequencialmente seus números. Sem a devida autenticação, qualquer usuário podia acessar qualquer informação livremente.

**Exploração da Vulnerabilidade**

- **Insecure Direct Object Reference (IDOR):** Esta falha permitia que qualquer pessoa acessasse documentos privados apenas alterando sequencialmente os números no URL. Sem a autenticação adequada, qualquer pessoa podia visualizar documentos confidenciais.
- **Erro Humano:** A falha foi inicialmente descoberta pela equipe interna, mas aparentemente não foi corrigida a tempo, resultando na exposição massiva dos dados.

**Impacto do Ataque**

- **Exposição de Dados Sensíveis:** Aproximadamente 885 milhões de arquivos foram expostos, incluindo documentos críticos como registros de hipotecas, extratos bancários, informações de transações e números de seguridade social.
- **Risco de Fraude e Roubo de Identidade:** As informações expostas poderiam ser usadas para fraudes, roubo de identidade e outros crimes cibernéticos.
- **Danos à Reputação:** A confiança dos clientes na First American Financial Corp foi severamente abalada, resultando em danos significativos à reputação da empresa.

**Medidas Corretivas**

- **Correção da Falha de IDOR:** Implementação de controles de acesso adequados para garantir que apenas usuários autenticados e autorizados possam acessar informações sensíveis.
- **Revisão e Melhoria das Práticas de Segurança:** Realização de auditorias de segurança regulares e testes de penetração para identificar e corrigir vulnerabilidades antes que possam ser exploradas.
- **Educação e Treinamento:** Capacitação contínua da equipe de desenvolvimento sobre as melhores práticas de segurança, especialmente em relação ao controle de acesso e manuseio de dados sensíveis.
- **Monitoramento Contínuo:** Implementação de monitoramento em tempo real para detectar e responder rapidamente a tentativas de acesso não autorizadas.

**Conclusão do vazamento**

O vazamento de dados da First American Financial Corp em 2019 destaca a importância crítica de implementar controles de acesso robustos e de realizar auditorias de segurança contínuas. Erros humanos, como a falha de corrigir vulnerabilidades conhecidas, podem ter consequências devastadoras. Este incidente serve como um lembrete para todas as organizações sobre a necessidade de vigilância constante e de práticas de segurança cibernética rigorosas para proteger dados sensíveis e manter a confiança do cliente.

## Brecha de Segurança na MGM Resorts International (2023)

**Descrição do Ataque**

Em 2023, a MGM Resorts International, uma gigante do setor de jogos avaliada em $14 bilhões, sofreu uma violação de segurança orquestrada pelo grupo de hackers Scattered Spider. Este ataque resultou em uma interrupção significativa dos sistemas da MGM. Pesquisadores conectaram os grupos de ransomware ALPHV/Blackcat/Scattered Spider aos ataques, com o ALPHV/Blackcat assumindo abertamente a responsabilidade.

**Identificação da Vulnerabilidade**

- **Privilégios de Super Administrador:** O grupo de hackers conseguiu persistência na rede da MGM com privilégios de super administrador. Isso sugere que eles tiveram acesso elevado e contínuo ao sistema, permitindo uma ampla visibilidade e a implantação de backdoors.

**Exploração da Vulnerabilidade**

- **Implantação de Ransomware:** Após bloquear a rede da MGM, os hackers implantaram ransomware, tornando os sistemas da empresa inacessíveis.
- **Exfiltração de Dados:** O grupo Scattered Spider alegou ter exfiltrado dados e ameaçou expor qualquer Informação Pessoal Identificável (PII) encontrada, a menos que um resgate significativo fosse pago.

**Impacto do Ataque**

- **Interrupção dos Sistemas:** O ataque causou uma interrupção significativa nos sistemas da MGM Resorts, afetando suas operações diárias e possivelmente impactando negativamente a experiência dos clientes.
- **Ameaça de Exposição de Dados:** A ameaça de expor dados exfiltrados colocou a privacidade dos clientes em risco e poderia levar a consequências legais e de reputação para a empresa.
- **Risco Financeiro:** A demanda por resgate representou um risco financeiro direto, além dos custos associados à resposta ao incidente e à recuperação dos sistemas afetados.

**Medidas Corretivas**

- **Reforço dos Controles de Acesso:** Implementação de medidas de segurança mais rigorosas para garantir que apenas usuários autorizados tenham acesso a sistemas críticos, com um foco especial em limitar privilégios administrativos.
- **Monitoramento e Detecção Avançada:** Adoção de ferramentas de monitoramento avançadas para detectar atividades suspeitas em tempo real e responder rapidamente a possíveis intrusões.
- **Auditorias de Segurança e Treinamento:** Realização de auditorias regulares de segurança para identificar e corrigir vulnerabilidades, além de treinamento contínuo para a equipe sobre práticas de segurança cibernética.
- **Melhorias na Gestão de Backups:** Estabelecimento de políticas robustas de backup e recuperação de desastres para garantir que dados críticos possam ser restaurados rapidamente em caso de ataques de ransomware.

**Conclusão do vazamento**

A violação de segurança na MGM Resorts International em 2023 destaca a necessidade crítica de reforçar a segurança cibernética, especialmente contra ataques sofisticados de ransomware. A capacidade dos hackers de obter privilégios de super administrador e implantar ransomware com sucesso demonstra a importância de implementar controles de acesso rigorosos, monitoramento avançado e medidas proativas de defesa cibernética. Este incidente serve como um lembrete da constante evolução das ameaças cibernéticas e da necessidade de vigilância contínua para proteger dados sensíveis e garantir a resiliência operacional.

## **Exemplos práticos em Broken Access Control**

## Funcionalidade de Admin desprotegida

![Figura 1](/images/2024-06-08-03.png)

Podemos tentar algumas formas diferentes para acessar o painel de administrador, neste caso basta colocarmos ‘/administrador-panel’ na URL. Como não há contramedidas, podemos acessar uma página que não deveria ser possível e agir como administrador.

![Figura 1](/images/2024-06-08-06.png)

## Funcionalidade de Admin desprotegida Com URL imprevisível

![Figura 1](/images/2024-06-08-10.png)

Neste caso a URL é imprevisível, significando que mesmo com Brute Force encontrar o painel do admin seria extremamente difícil, portanto o que podemos fazer neste caso é acessar o código fonte da página para procurar por alguma informação.

![Figura 1](/images/2024-06-08-02.png)

Podemos ver que o que procurávamos estava Hardcoded no código fonte da página. Com isso basta colocar ‘/admin-ms7fbd’ na URL para conseguirmos acessar o painel do administrador.

## Controle por parâmetro de solicitação feita pelo usuário

Neste caso temos uma vulnerabilidade em que o usuário pode manipular uma solicitação. Quando entramos na conta de usuário é enviada uma solicitação para o servidor, e inspecionando a página, indo em Application e em seguida em Cookies, podemos ver que o campo Admin é dado como ‘false’, portanto basta colocarmos como ‘true’.

![Figura 1](/images/2024-06-08-01.png)

Após atualizarmos a página podemos ver que conseguimos entrar como admin.

## O papel do usuário pode ser modificado no perfil do usuário

Aqui temos uma vulnerabilidade que permite o usuário se tornar admin modificando o ‘roleid’.

![Figura 1](/images/2024-06-08-04.png)

Podemos ver que quando se faz uma requisição para mudar o email uma das informações dada como resposta é o roleid do usuário, se pudermos modificar o valor do roleid poderemos acessar outras contas.

![Figura 1](/images/2024-06-08-08.png)

E assim conseguimos acessar as funcionalidades de admin.

## O controle de acesso baseado em URL pode ser contornado

Para este caso queremos usar X-Original-URL, que basicamente faz com que o request use outro URL, e para testarmos se a aplicação é vulnerável podemos tentar acessar um diretório que sabemos que não existe, e se a resposta for 404 Not Found confirmamos que a aplicação está vulnerável à este ataque.

![Figura 1](/images/2024-06-08-07.png)

Colocando /admin no X-Original-Url conseguimos burlar a restrição de acesso, mas se tentarmos fazer alguma coisa que o admin poderia fazer receberemos que o acesso foi negado, portanto como podemos ver o endpoint que queremos usar que no caso seria para deletar o usuário carlos. Porém como não pode colocar argumentos no X-Original-Url, basta colocarmos no cabeçalho.

![Figura 1](/images/2024-06-08-09.png)

Após enviarmos essa requisição recebemos 302 Found, o que significa que provavelmente funcionou.

## Conclusão

Broken Access Control é uma das vulnerabilidades de segurança mais críticas e comuns nas aplicações web hoje em dia. Esta vulnerabilidade pode resultar em graves consequências, como a exposição de dados sensíveis, comprometimento de sistemas e perda de confiança dos usuários. Exemplos notáveis, como os vazamentos de dados do Facebook, First American Financial Corp e MGM Resorts International, ilustram o impacto devastador que essas falhas podem causar.

Para mitigar os riscos associados ao Broken Access Control, é essencial implementar práticas robustas de segurança, incluindo auditorias regulares, princípios de menor privilégio, autenticação forte e validação adequada de entrada. Ferramentas como Burp Suite, ZAP e outras ferramentas de teste de segurança de API são recursos valiosos para identificar e corrigir essas vulnerabilidades.

Além disso, a educação e o treinamento contínuos para desenvolvedores e administradores são fundamentais para garantir que as melhores práticas de segurança sejam seguidas e que as configurações incorretas sejam evitadas. Com uma abordagem proativa e vigilante, é possível proteger sistemas e dados contra acessos não autorizados, mantendo a integridade e a confiança das aplicações e dos serviços oferecidos.

## Referências:

https://brightsec.com/blog/broken-access-control-attack-examples-and-4-defensive-measures/

https://dev.to/gabogaldino/explicandobroken-access-control-para-desenvolvedores-5dmn

https://medium.com/@nidhirohilla.nr/gate-crasher-fixing-broken-access-control-80906ce6b143#:~:text=Cambridge%20Analytica%20Scandal%3A%20Broken%20Access,million%20users%20without%20their%20consent.

https://www.nytimes.com/2018/04/04/us/politics/cambridge-analytica-scandal-fallout.html

https://qbaindia.com/articles-%26-blogs/f/hacks-history---broken-access-control

https://securityleaders.com.br/mgm-resorts-calcula-perdas-de-us-100-milhoes-devido-a-vazamento-de-dados/

https://www.skyhighsecurity.com/pt/about/resources/intelligence-digest/mgm-resorts-cyberattack-from-cloud-to-casino-floor.html

https://dracones.com.br/ataque-hacker-em-las-vegas/

https://www.bloomberglinea.com.br/internacional/ciberataque-a-mgm-resorts-paralisa-maquinas-de-cassino-e-entradas-em-hoteis/

https://sud-defcon.medium.com/tryhackme-owasp-broken-access-control-writeup-afc0c911f5b6

https://securityboulevard.com/2023/03/23-most-notorious-hacks-history-that-fall-under-owasp-top-10/

https://www.indusface.com/blog/owasp-top-10-vulnerabilities-in-2021-how-to-mitigate-them/

https://owasp.org/Top10/A01_2021-Broken_Access_Control/

https://dev.to/gabogaldino/explicandobroken-access-control-para-desenvolvedores-5dmn

https://sud-defcon.medium.com/tryhackme-owasp-broken-access-control-writeup-afc0c911f5b6

https://medium.com/@nidhirohilla.nr/gate-crasher-fixing-broken-access-control-80906ce6b143#:~:text=Cambridge%20Analytica%20Scandal%3A%20Broken%20Access,million%20users%20without%20their%20consent.

https://portswigger.net/web-security/access-control
