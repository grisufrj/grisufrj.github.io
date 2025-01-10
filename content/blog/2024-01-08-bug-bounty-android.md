---
title: "Bug bounty android: Um breve guia de um bug bounty em Android"
date: 2024-01-08
published: 2024-01-08
description: "Um breve guia de um bug bounty em Android"
tags:
  - "Android"
images: ["/images/2024-01-08-12.png"]
author: richardpiccoli
---

## Introdução

Bug bounty em Android é um programa no qual empresas ou desenvolvedores oferecem recompensas para pessoas que descobrem e relatam vulnerabilidades de segurança em seus aplicativos ou sistemas operacionais Android. Neste trabalho, abordaremos os principais tópicos relacionados a bug bounty em Android, incluindo a importância do programa, as etapas envolvidas, técnicas comuns de testes de segurança e melhores práticas.

## I. Importância do Bug Bounty em Android:

- A crescente importância da segurança no ecossistema Android é impulsionada pelo aumento do número de ameaças cibernéticas direcionadas a dispositivos móveis.
- Os riscos potenciais de não ter um programa de bug bounty incluem o vazamento de dados dos usuários, comprometimento da integridade dos aplicativos e perda de reputação da empresa.
- Benefícios para os desenvolvedores e empresas que adotam um programa de bug bounty incluem a identificação antecipada de vulnerabilidades, melhoria da segurança e aumento da confiança dos usuários.

## II. Etapas de um Bug Bounty em Android:

**Preparação:**

- Definição dos objetivos do programa, para identificar e corrigir vulnerabilidades antes que sejam exploradas por agentes mal-intencionados.
- Definição do escopo para especificar quais aplicativos, versões do Android, dispositivos e outras áreas estão incluídas no programa de bug bounty.
- Estabelecimento de regras claras para relatórios e recompensas, é importante definir o formato esperado dos relatórios de vulnerabilidade e as diretrizes para determinar as recompensas.

**Descoberta de Vulnerabilidades:**

- Técnicas comuns de testes de segurança em Android incluem análise estática de código, análise de reversão, fuzzing e testes de penetração.
- Teste de análise estática de código para examinar o código-fonte em busca de vulnerabilidades conhecidas, uso inadequado de APIs e más práticas de segurança.
- Teste de análise de reversão: examinar o código compilado em busca de vulnerabilidades, como strings sensíveis não criptografadas ou acesso inadequado a recursos.
- Fuzzing para enviar inputs aleatórios ou manipulados para identificar falhas de programação, como buffer overflows ou crashes do aplicativo.
- Testes de penetração são usados para simular ataques de hackers reais para identificar pontos fracos no aplicativo ou no sistema operacional Android.

**Relato de Vulnerabilidades:**

- Os relatórios de vulnerabilidades devem ser detalhados e conter informações claras sobre a vulnerabilidade encontrada, incluindo passos para reprodução, contexto técnico e impacto potencial.
- As informações relevantes podem incluir a versão do Android, o dispositivo usado, o aplicativo específico e qualquer outro detalhe que ajude a entender a vulnerabilidade.
- Recomenda-se utilizar plataformas de bug bounty, como HackerOne ou Bugcrowd, para garantir que os relatórios sejam recebidos e tratados adequadamente.
- Quando usado alguma ferramenta de análise automática é bom sempre verificar a vulnerabilidade antes de reportar.

**Validação de Vulnerabilidades:**

- A equipe responsável pelo programa de bug bounty irá revisar e validar os relatórios de vulnerabilidade recebidos.
- É importante fornecer provas claras da vulnerabilidade, como capturas de tela, logs ou scripts de exploração.
- A comunicação adequada entre o pesquisador e a equipe de bug bounty é fundamental para esclarecer dúvidas e garantir que a vulnerabilidade seja compreendida corretamente.

**Recompensas e Reconhecimento:**

- As recompensas em bug bounty podem variar, desde recompensas monetárias até agradecimentos públicos e até mesmo convites para eventos de segurança.
- A transparência no processo de recompensas é importante para motivar os pesquisadores e reconhecer seu trabalho.
- O reconhecimento público dos pesquisadores, seja por meio de um hall da fama ou de divulgação responsável, também é uma prática comum para destacar suas contribuições.

## III. Alguns tópicos adicionais relevantes para o bug bounty em Android:

**Ferramentas comuns utilizadas em bug bounty em Android:**

- Drozer é uma ferramenta de segurança de aplicativos Android que ajuda os pesquisadores a encontrar vulnerabilidades e realizar testes de segurança.
- Frida é uma estrutura de injeção de código que permite manipulação dinâmica de aplicativos Android para identificar vulnerabilidades ou contornar medidas de segurança.
- MobSF (Mobile Security Framework) é uma estrutura de teste de segurança que automatiza a análise estática e dinâmica de aplicativos Android para identificar vulnerabilidades conhecidas e desconhecidas.

**Melhores práticas para mitigação de vulnerabilidades em aplicativos Android:**

- Uso seguro de APIs é fundamental que os desenvolvedores implementem corretamente as APIs do Android, evitando o uso inadequado que possa levar a vulnerabilidades.
- Criptografia adequada é necessária para os dados sensíveis, tanto em repouso quanto em trânsito, para protegê-los contra acesso não autorizado.
- Validação de entrada, todos os dados de entrada devem ser validados e sanitizados corretamente para evitar ataques de injeção de código, como SQL injection ou XSS.
- Atualizações regulares, os desenvolvedores devem fornecer atualizações de segurança regulares para corrigir vulnerabilidades conhecidas e melhorar a segurança geral do aplicativo.

**Desafios e limitações do bug bounty em Android:**

- Surgimento de técnicas de evasão de segurança, hackers estão constantemente desenvolvendo novas técnicas para evitar a detecção de vulnerabilidades em aplicativos Android, o que pode dificultar a descoberta de vulnerabilidades.
- Complexidade de encontrar vulnerabilidades em dispositivos móveis, como os dispositivos móveis são altamente diversificados em termos de hardware, sistema operacional e aplicativos, o que torna o processo de encontrar vulnerabilidades mais complexo em comparação com plataformas tradicionais.

**Top 3 vulnerabilidades de API encontradas em 2023 pela OWASP:**

- API1:2023 - Broken Object Level Authorization As APIs tendem a expor endpoints que lidam com identificadores de objeto, criando uma ampla superfície de ataque de problemas de controle de acesso em nível de objeto. As verificações de autorização em nível de objeto devem ser consideradas em todas as funções que acessam uma fonte de dados usando um ID do usuário.
- API2:2023 - Broken Authentication Os mecanismos de autenticação geralmente são implementados incorretamente, permitindo que os invasores comprometam os tokens de autenticação ou explorem falhas de implementação para assumir as identidades de outros usuários temporária ou permanentemente. Comprometer a capacidade de um sistema de identificar o cliente/usuário compromete a segurança geral da API.
- API3:2023 - Broken Object Property Level Authorization Esta categoria combina API3:2019 Excessive Data Exposure e API6:2019 - Mass Assignment, com foco na causa raiz: a falta ou validação inadequada de autorização no nível da propriedade do objeto. Isso leva à exposição ou manipulação de informações por partes não autorizadas.

## IV. Começando um Bug Bounty:

**1 - Plataformas para Bug Bounty:**

Existem diversas plataformas que podemos usar para escolhermos o programa desejado, umas das principais são:

- Bug Hunters da Google, em que disponibiliza programas de Bug Bounty tanto para aplicativos da Google como para aplicativos populares de outras plataformas.
- Hackerone, em que fazem CTF que podem te dar convites caso ache as flags, tem muitos programas nesta plataforma, porém têm muitas pessoas utilizando o que pode dificultar.
- Bugcrowd também é uma plataforma muito grande com diversos programas e usuários, que oferece ajuda quando se escreve um report inválido com comentários.
- Além de Itigriti, Synack e muitas outras plataformas que se pode escolher que se encaixe melhor com suas preferências.

**2 - Programa de escolha:**

Para escolher um programa é importante saber qual é o seu tipo de preferência, muitas pessoas dão preferência por programas que tenham o escopo grande, e que tenham muitas funcionalidades.

**3 - Bug Bounty ou VDP:**

- O Vulnerability Disclosure Program (VDP) é uma iniciativa que estabelece um canal formal para relatar vulnerabilidades de forma ética e responsável, VDP é ótimo para iniciantes na área.
- O Bug Bounty é uma modalidade dentro do VDP, onde empresas oferecem recompensas, como dinheiro, para pesquisadores que encontram e relatam vulnerabilidades específicas.
- Ambos os programas visam melhorar a segurança, permitindo a identificação e correção de falhas antes que causem danos. São uma forma de incentivar a colaboração e proteger legalmente os pesquisadores que ajudam a aprimorar a segurança das organizações.

**4 - Uma estratégia para Bug Bounty Android:**

- Estudo do programa em sua utilização normal para se saber bem o funcionamento do programa.
- Busca por informações sensíveis em logs.
- Procurar por informações sensíveis hardcoded.
- Busca por informações sensíveis guardadas de forma insegura.
- Testar por vulnerabilidades de injeção.
- Procurar por vulnerabilidades de controle de acesso.
- Verificar se há buffer overflow em bibliotecas externas.
- Executar metodologias de testes API.
- Assim como utilizar ferramentas de scan automatizado.

**5 - Por fim, vamos fazer um passo a passo de uma tentativa de Bug Bounty:**

Começamos procurando um programa de Bug Bounty que tenhamos interesse, nesse caso escolhemos o programa do app da SHEIN oferecido no Hackerone.

![Figura 1](/images/2024-01-08-01.png)

Como o foco deste trabalho é em Android, vamos escolher o escopo com o tipo de ‘Android: Play Store’.

![Figura 1](/images/2024-01-08-02.png)

Clicando no hipertexto ‘SHEIN-Fashion Shopping Online’ somos direcionados para a página da SHEIN na Play Store. Agora precisamos do .apk do aplicativo para conseguirmos colocar em nosso emulador de Android, para isso podemos colocar o endereço da página da Play Store neste site: https://apps.evozi.com/apk-downloader/ para conseguirmos baixar o .apk.

![Figura 1](/images/2024-01-08-03.png)

Com o .apk baixado podemos instalá-lo em nosso emulador com os comandos adb push e adb install. Com o apk instalado podemos começar nosso processo para encontrar bugs. Antes de tudo, vamos usar um pouco o aplicativo e ficar familiarizado com todas suas funcionalidades de processos. Podemos fazer isso com o logcat rodando de fundo para que possamos analisar se há alguma informação importante sendo mostrada no log, como erros por exemplo.

![Figura 1](/images/2024-01-08-04.png)

Após isso, podemos procurar por informações hardcoded com o Jadx como chaves secretas, senhas e qualquer outra informação que não deveria estar em hardcode.

![Figura 1](/images/2024-01-08-05.png)

Neste caso não encontrei nada do tipo, portanto iremos procurar informações sensíveis armazenadas de forma insegura.
Podemos ir em shared preferences em /data/data/com.zzkko/shared_prefs e procurar alguma informação.
Também podemos ir em databases em /data/data/com.zzkko/databases.

![Figura 1](/images/2024-01-08-06.png)

Podemos também tentar vulnerabilidades de injeção SQL, porém não vi como aplicar neste caso. Depois de tentarmos encontrar vulnerabilidades de controle de acesso e buffer overflow em bibliotecas externas vamos tentar atacar de outra forma.

Vamos realizar testes de API, para isso vamos entrar no Burp Suite.Vamos entrar no emulador com o proxy específico para ele, então teremos que abri-lo com ‘emulator -avd oreo -writable-system -http-proxy 192.168.1.18:8081’.

A primeira coisa que temos que fazer é procurar e estudar os endpoints, fazer uma tabela no Exel por exemplo ajuda, mas pode ser bloco de notas, papel ou o que for melhor. Estando familiarizado com os endpoints podemos fazer uma abordagem mais organizada.

![Figura 1](/images/2024-01-08-07.png)

Podemos começar com este, já que envolve produtos e tem bastante informação no Response.
As abas que usaremos serão basicamente a aba Target, Repeater e Intruder.

- Em Target selecionamos um endpoint que desejamos verificar para enviar para a aba Repeater.
- Em Repeater Conseguimos manipular os Requests e Responses, Ex.:
  GET /api/users/6 HTTP/1.1
  Pode mudar este header para:
  GET /api/users/ HTTP/1.1
  E com isso ver o Respose de todos os usuários no lugar de somente o usuário com id 6.
- Em Intruder podemos realizar Brute Force, e enviar requisições desejadas para variáveis específicas (Ex.: users, id, etc). Pode adicionar o playload desejado marcando a variável e apertando em 'add $'.

![Figura 1](/images/2024-01-08-08.png)

E com isso selecionar as palavras chave que desejamos para o brute force.

![Figura 1](/images/2024-01-08-09.png)

Podemos agora tentar alterar diretamente no Repeater para conseguirmos enviar o que queremos, como por exemplo no lugar de GET colocarmos PUT para alterar o valor de algo que desejamos.

Para isso precisamos fazer um pequeno código em json utilizando ‘Content-Type:application/json’ para fazer um código json funcionar
Ex.: {"discount":80}

Como muitos aplicativos e programas podem ser muito grandes e com muitas funcionalidades podemos usar ferramentas para fazer análises automatizadas. É importante notar que sempre precisamos verificar o que essa ferramenta automatizada nos aponta como falha de segurança ou vulnerabilidade se realmente tal linha de código apresenta um risco ou se a vulnerabilidade não se aplica para o contexto do programa.

A ferramenta que usaremos será MobSF para conseguirmos um relatório completo:

![Figura 1](/images/2024-01-08-10.png)

MobSF tem diversas funcionalidades, e explicitadas de uma forma muito amigável para o usuário. Com o resumo geral da classificação do aplicativo logo no começo, e conforme for descendo na página é mostrado em detalhe cada característica específica da análise, que também pode ser acessada pelo painel a esquerda.

Alguns pontos que podemos destacar nesta análise são network security:

![Figura 1](/images/2024-01-08-11.png)

Que deixa texto puro trafegar por todos os domínios.
Na análise do AndroidManifest podemos ver que aparentemente existe risco de algum tipo de abuso pelos intents, além de alguns outros avisos e outras informações achadas no app.

![Figura 1](/images/2024-01-08-12.png)

**Sites importantes:**

https://bughunters.google.com/

https://blog.oversecured.com/

https://www.hackerone.com/

https://owasp.org/

https://www.bugcrowd.com/

https://www.intigriti.com/
