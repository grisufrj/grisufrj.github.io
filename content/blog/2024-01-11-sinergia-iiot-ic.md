---
title: "Sinergia entre a Internet Industrial das Coisas (IIoT) e as Infraestruturas Críticas"
date: 2024-01-11
published: 2024-01-11
description: "Infraestruturas Críticas"
images: []
categories:
  - "Medium"
author: michellefortunato
---


## RESUMO

É consenso que a Internet Industrial das Coisas (IIoT) trouxe avanços tecnológicos para a indústria, além de novos arranjos de uma produção mais inteligente, que possibilita maior conexão entre tecnologia e desenvolvimento sustentável, além das melhorias nas funcionalidades do software SCADA, muito utilizado por diversos Sistemas de Controle Industriais (ICS). No entanto, há uma preocupação crescente com a segurança das Infraestruturas Críticas, uma vez que ficaram mais suscetíveis aos ataques cibernéticos. Nesse sentido, este resumo tem como objetivo fazer uma breve revisão bibliográfica sobre o assunto, pontuar sobre o cenário geral das inovações, além de fornecer ao leitor um exemplo de arquitetura de um sistema de tratamento de água baseado em ICS-SCADA/IIoT. A metodologia utilizada inclui além da consulta à base de artigos da Science Direct, também a base de documentos da WIPO (*World Intellectual Property Organization*). 

## INTRODUÇÃO

A Internet das Coisas (IoT) impulsionou a transformação de todos os setores econômicos devido ao aumento da interconectividade tecnológica que trouxe inúmeros benefícios a sociedade. Do mesmo modo, encontra-se a Internet Industrial das Coisas (IIoT) que faz uso de dispositivos inteligentes que visam aumento da automação, eficiência e produtividade na indústria. A IIoT está presente em praticamente todos os setores produtivos e encontra um cenário mundial desafiador que inclui, dentre outros pontos, a busca pela melhoria de específicos indicadores dos Objetivos do Desenvolvimento Sustentável (ODS) da Nações Unidas que visam atingir metas globais até 2030.

As aplicações inteligentes já são consideradas estratégicas para promoção do desenvolvimento sustentável, uma vez que podem fornecer em tempo real dados, insights e análises mais rápidas e precisas que melhoram a eficiência de variados sistemas de produção e monitoramento, além de melhorar o gerenciamento dos recursos disponíveis na indústria. A contribuição trazida pela IIoT inclui, especialmente, melhorias quanto aos indicadores ODS9 e ODS12 em que, busca-se aumento do acesso às tecnologias da informação e comunicação; apoio ao desenvolvimento tecnológico, pesquisa e inovação; modernização e facilitação do desenvolvimento de infraestrutura sustentável; melhoria da capacidade tecnológica das indústrias; desenvolvimento de infraestrutura de qualidade, confiável, sustentável e resiliente; redução do desperdício de recursos naturais tendo como referencial a produção industrial mais responsável. 

Além disso, segundo a OECD (2024) a IIoT pode auxiliar no aumento da vida útil dos equipamentos, manutenção preditiva quanto a falha das máquinas, rastreia a entrada e saída de mercadorias possibilitando um planejamento mais eficiente, otimização do consumo de energia, redução do uso de recursos naturais e redução de emissões de poluentes atmosféricos. O cenário da IIoT na indústria é promissor.  Em média, a tecnologia IIoT é utilizada por 31% das empresas européias do setor transformador e 22%, no Canadá. As empresas de energia lideram o uso com 47% nos países europeus e 46% no Canadá. Essas empresas adotaram a IIoT visando melhorar a segurança das suas instalações (76%), o controle dos processos produtivos e logística (42%) e otimizar o consumo de energia (37%). Por outro lado, IIoT trouxe muitos desafios quando da ampliação dos sistemas conectados na nuvem e com arranjos em múltiplas soluções, tornando-se falhos na prova de conceito, apresentando riscos de segurança e privacidade, expressivos custos de integração e falta de padrões e interoperabilidade. Esses fatores são relatados como os principais que retardam e/ou interrompem a implantação da IIoT. Além disso, em especial à segurança das Infraestruturas Críticas, uma vez que há maiores preocupações quanto à privacidade dos dados, sobrecarga de comunicação, gerenciamento de dados sensíveis e vulnerabilidade aos ataques cibernéticos podendo causar expressivos impactos ambientais (Kimani et al., 2019). 

Para se ter uma ideia da importância desse tema, a Agência de Segurança Cibernética e Infraestrutura dos EUA (CISA) categorizou 16 setores considerados de Infraestrutura Crítica como mostrado na Figura 1, dentre os quais, encontram-se a indústria química, nuclear, energia e os sistemas de tratamento de água, que têm sido alvos atraentes de ataques cibernéticos. A relevância dada a esses setores é devido ao alto impacto destrutivo que os ataques cibernéticos podem provocar na segurança econômica e patrimonial nacional, na saúde e na segurança ambiental da população. 

![Figura 1 – Infraestruturas críticas. Fonte: Adaptado de CISA (2022).](/images/imagem1m.jpg)


Figura 1 – Infraestruturas críticas. Fonte: Adaptado de CISA (2022).


Sabe-se que essas infraestruturas fazem parte de uma Infraestrutura Nacional Crítica (CNI) e que são prioritariamente protegidas em casos de ataques. Cada país estrutura seu CNI de forma própria e, no caso do Brasil, por meio do Decreto nº 9.573, de 22 de novembro de 2018, a Política Nacional de Segurança de Infraestruturas Críticas (PNSIC) foi estruturada incluindo, entre outros itens, instalações e sistemas cujas interrupções ou destruições, total ou parcial, provoquem sério impacto social, ambiental, econômico, político, internacional ou à segurança do Estado e da sociedade. Em suma, a PNSIC é uma política de Estado. Tão importante quanto a PNSIC é a configuração da Política Nacional de Cibersegurança (PNCiber) criada pelo Decreto nº 11.856, de 26 de dezembro de 2023 que tem como princípio a prevenção de incidentes e ataques cibernéticos ao CNI e de serviços essenciais prestados à sociedade.

As motivações para ataques cibernéticos às infraestruturas desse tipo são variadas, mas é certo que as vulnerabilidades encontradas nesses ambientes de controle e automação são maiores quando ligados à IIoT porque amplia as possibilidades de ataques ao combinar uma diversidade de dispositivos industriais, sistemas de controle, sensores, atuadores à redes heterogêneas de comunicação com e sem fio, acarretando na exploração de vulnerabilidades de interfaces web, que podem comprometer a integridade, confidencialidade e disponibilidade do serviço e/ou dados. Nesse sentido, os dispositivos que fazem parte de todo aparato IIoT podem ser alvo de ataques, espionagem, roubo de dados e destruição online, sendo a facilidade de acesso diretamente vinculada ao alcance global da Internet. Portanto, as ameaças devem ser melhor compreendidas uma vez que os dados gerados e os dispositivos IIoT podem ser detectados e controlados remotamente (Figura 2). Alguns exemplos de ameaças cibernéticas nesse contexto incluem: Man in The Middle (MITM), Distributed Denial of Service (DDoS) e injeção de malware na nuvem (Mekala et al., 2023).

![Figura 2 – Exemplo de configuração em nuvem com cenário de ataque. Fonte: Bhamare et al. (2020).](/images/imagem2m.jpg)


Figura 2 – Exemplo de configuração em nuvem com cenário de ataque. Fonte: Bhamare et al. (2020).


Além disso, Urquhart et al. (2018) pontuaram sobre a amplitude dos riscos no contexto da IIoT que, de forma ampla, podem incluir ataques cibernéticos envolvendo: Hackers patrocinados pelo Estado para roubo de segredos militares e/ou informações de eleições, espionagem e sabotagem comerciais; grupos criminosos organizados que invadem remotamente organizações visando obter informações confidenciais, em especial, grupos como Lulzsec ou Anonymous que utilizam ataques como DDoS como justificativa para fazer justiça social e/ou retaliações tendo como alvos websites ou Infraestruturas Críticas para criar interrupções de serviços acarretando em altos custos financeiros e de reputação comercial.  

De forma geral existem, pelo menos, 10 ataques cibernéticos à Infraestruturas Críticas reportados na literatura como de grande importância, dentre eles estão: (a) ataque ao sistema de elétrico na cidade de Lodz (Polônia) em 2008 que resultou no ferimentos de passageiros; (b) ataque à rede de sistema de energia da Texas Power Company em 2009; (c) ataque Stuxnet por worm em 2009 à instalação de energia nuclear iraniano que destruiu as centrífugas de enriquecimento de urânio U-235; (d) ataque ao sistema de distribuição de água em 2011 do Departamento de Água e Esgoto da cidade de South Houston, Texas; (e) ataque à represa da Bowman Avenue de Nova York em 2013 que foi destruída; (f) ataque à rede elétrica da Ucrânia por malware BlackEnergy que causou um enorme apagão em 2015; (g) ataque ao sistema ferroviário da cidade de São Francisco (EUA) em 2016 via ransomware; (h) ataque ao sistema de controle da concessionária de água da Kemuri Water Company em 2016 em que foram alterados os níveis dos produtos químicos usados no tratamento de água; (i) ataque ao edifício em Lappeenranta (Finlândia) em 2016 que causou a interrupção de aquecimento de água durante o inverno finlandês; (j) ataque à fábrica petroquímica na Arábia Saudita em 2017 que pretendia sabotar as operações e causar uma explosão. Pondera-se que a maior parte da engenharia de sistemas de controle dessas ocorrências envolviam tipicamente o software SCADA (Supervisory Control and Data Acquisition).

É importante deixar claro que, na automação industrial, existem 4 gerações de SCADA, tais como: os sistemas SCADA iniciais (primeira geração) em que os dados adquiridos e o controle realizado se davam de forma independente, sem conexões com redes de comunicação e com interface homem-máquina; nos sistemas SCADA distribuídos (segunda geração) é possível controlar vários sistemas simultaneamente; nos sistemas SCADA em rede (terceira geração) o controle ocorre por conexão com redes de dados e/ou telefônicas com fibra óptica ou Ethernet; e nos sistemas SCADA IIoT (quarta geração), na automação industrial inteligente, foi possível simplificar o gerenciamento dos dados e melhorar os sistemas SCADA, para garantir operações automatizadas e remotas sem a necessidade de intervenção humana. Para exemplificar, na Figura 3 está demonstrada uma arquitetura genérica de processo em SCADA/IIoT em malha de controle fechada.  Esse sistema de controle por retroalimentação é muito utilizado na indústria porque torna o sistema mais rápido, preciso e menos sensível a perturbações no processo industrial. No entanto, essa malha está mais suscetível a diminuição da estabilidade. Os principais componentes de um sistema SCADA incluem unidades de hardware e software, em que os aplicativos são executados em um servidor e os desktop e telas atuam como HMI (Human Machine Interface) conectada ao servidor. Além disso, duas importantes unidades estão presentes: a Unidade Terminal Mestre (MTU) que é núcleo do sistema SCADA, e a Unidade Terminal Remota (RTU) ou PLC (Controlador Lógico Programável) conectado aos sensores e atuadores do processo industrial. Nos RTUs são armazenados os dados de processo e transmitidos ao MTU por meio da rede de comunicação. Aqui, neste resumo, para fins de simplificação, não será descrita a dinâmica de resposta da malha de controle e tão pouco os tipos de controladores, sensores e atuadores. Essa tarefa está prevista para um próximo resumo.

![Figura 3 – Configuração genérica de processo em SCADA/IIoT em malha de controle fechada. Fonte: Elaboração própria.](/images/imagem3m.jpg)


Figura 3 – Configuração genérica de processo em SCADA/IIoT em malha de controle fechada. Fonte: Elaboração própria.


No caso particular de processos industriais em Infraestruturas Críticas no contexto de SCADA/IIoT, pode-se destacar um sistema simplificado de tratamento de água como demonstrado na Figura 4, em que tem o objetivo de controlar a pressão de fluxo antes de uma bomba hidráulica e após as saídas dos filtros. Esse processo foi idealizado pela empresa Define Instruments® para uma grande estação de filtração de água na região da Flórida/EUA. É importante destacar que o processo de filtração de água é importante na retirada de diversos contaminantes da água. O meio filtrante pode ser desde leitos de areia à carvão ativado para remoção de particulados com variados tipos de granulometria, além de membranas que possuem poros muito reduzidos. 

![Figura 4 – Monitoramento e controle remoto em estação de filtração de água. Fonte: Site da empresa Define Instruments®.](/images/imagem4m.jpg)


Figura 4 – Monitoramento e controle remoto em estação de filtração de água. Fonte: Site da empresa Define Instruments®.


É relevante esclarecer que os objetivos principais do controle de processos são suprimir a influência de perturbações, assegurar a estabilidade e otimizar o desempenho. Nesse sentido, as invasões cibernéticas podem ter como alvo especialmente os atuadores mais vulneráveis, acarretando adição de perturbações à malha o que gera estados inseguros da planta industrial. Yadav et al. (2021) listaram importantes ataques à sistema ICS-SCADA desde a década de 80 e mostram que 28% dos relatos são devido à ataques de malware. Sabe-se que os sistemas de controle baseados em IIoT, em especial, para Infraestruturas Críticas, passam por contínuo aperfeiçoamento, uma vez que os ataques cibernéticos se tornaram mais frequentes.  Isso promove as crescentes inovações nesse setor. 


## CENÁRIO GERAL DAS INOVAÇÕES

Para se ter uma ideia do cenário de inovações nesse setor, uma breve prospecção tecnológica foi realizada em Janeiro de 2024. Foram utilizadas algumas combinações de palavras de busca na base WIPO (World Intellectual Property Organization) para melhor compreensão da ocorrência de patentes reivindicadas para tecnologia IIoT aplicada à Infraestruturas Críticas em sistemas SCADA. Ao usar as palavras-chave “IIoT” and “critical infrastructure” foram obtidos 1284 documentos e a adição do termo “scada”, recuperou 211 patentes. Pode-se constatar com esse levantamento que houve um aumento expressivo dos depósitos de patentes em 2023, sendo os Estados Unidos como principal país depositante, como mostrado na Figura 5. Além disso, as principais empresas inventoras são Qualcomm®, Strong Force Innovation®, Schneider Electric®, Kaspersky Lab® e AVEVA Software®.  Interessante também citar que os depósitos tiveram início em 2017 e se intensificaram após a pandemia de Covid-19 quando da busca pelos termos “IIoT” and “critical infrastructure”. Ressalta-se que grande parte das invenções consideraram requisitos e/ou boas práticas de segurança cibernética, o que mostra a conscientização dos inventores quanto à necessidade de superação dos desafios em cenários de ataques virtuais às Infraestruturas Críticas. A presença marcante dos EUA em número de patentes é coerente com a estatística mostrada no estudo de Yadav et al. (2021) em que mostram os EUA como principal país atacado. Os números expressivos de ataques aos seus sistemas ICS-SCADA impulsionaram a corrida desse país na busca por mais soluções inovadoras. 

![Figura 5 – Países inventores e anos de publicação das patentes. Fonte: WIPO (2024).](/images/imagem5m.jpg)


Figura 5 – Países inventores e anos de publicação das patentes. Fonte: WIPO (2024).


## CONSIDERAÇÕES FINAIS

O breve levantamento bibliográfico realizado deixou claro que a utilização da Internet revolucionou os sistemas e arquiteturas ICS-SCADA e esses se tornaram o cerne das Infraestruturas Críticas. Constatou-se que as melhorias e facilidades criadas pela abordagem IIoT para uma indústria mais inteligente dialogam positivamente com as melhorias propostas pelo desenvolvimento sustentável. No entanto, é urgente que essa nova concepção industrial priorize a segurança cibernética, já que nela se concentram os gargalos tecnológicos do setor, particularmente, durante a concepção dos projetos com interface entre tecnologia operacional e tecnologia da informação. Essa necessidade tem sido incorporada às recentes inovações do setor, apesar de ainda muito recentes (após pandemia de Covid-19) e concentrada nos EUA. 


## REFERÊNCIAS BIBLIOGRÁFICAS

Bhamare, D.; Zolanvari, M.; Erbad, A.; Jain, R.; Khan, K. Cybersecurity for industrial control systems: A survey. Computers & Security 89 (2020) 101-677.

CISA. Cybersecurity & Infrastructure Security Agency. 3RD Annual National Cybersecurity Summit. Recommended Cybersecurity Practices for Industrial Control Systems (2022). 

Kimani, K.; Oduol, V.; Langat, K. Cyber security challenges for IoT-based smart grid networks. International Journal of Critical Infrastructure Protection 25 (2019) 36-49.

Mekala, S.H.; Baig, Z.; Anwar, A.; Zeadally, S. Cybersecurity for Industrial IoT (IIoT): Threats, countermeasures, challenges and future directions. Computer Communications 208 (2023) 294–320.

OECD. Organisation for Economic Co-operation and Development. Case study on Internet of Things in manufacturing. Disponível em: <https://www.oecd-ilibrary.org/sites/c4be2088-en/index.html?itemId=/content/component/c4be2088-en>. Acesso: 05/01/24.

Urquhart, L.; McAuley, D. Avoiding the internet of insecure industrial things. Computer Law & Security Review 34 (2018) 450-466.

WIPO. World Intellectual Property Organization. 

Yadav, G.; Paul, K. Architecture and security of SCADA systems: A review. International Journal of Critical Infrastructure Protection 34 (2021) 100-433.

