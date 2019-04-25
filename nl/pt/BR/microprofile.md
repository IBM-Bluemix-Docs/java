---

copyright:
  years: 2019
lastupdated: "2019-04-04"

keywords: java ee, jakarta ee, microprofile overview, cloud native java, cloud native microprofile

subcollection: java

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}

# Visão geral
{: #jee-overview}



## Java EE e Jakarta EE
{: #jakarta-ee}

As tecnologias introduzidas em Java EE7 e EE8 são bem adequadas para a criação de microsserviços em Java, com suporte para anotações, protocolos leves e padrões de interação assíncrona. Utilitários de simultaneidade no Java EE7 fornecem um mecanismo de encaminhamento direto para o uso de bibliotecas reativas como o RxJava de uma maneira amigável ao contêiner.

Em setembro de 2017, a Oracle, com o suporte da IBM e da Red Hat, anunciou que o Java EE ia se mudar para a Eclipse Foundation, como o Jakarta EE. A Oracle continua a possuir tudo o que está associado ao Java EE, até e incluindo o Java EE 8, mas todo o desenvolvimento futuro dessa plataforma Enterprise Java após o Java EE 8 será na Eclipse Foundation sob a orientação do grupo de trabalho do Jakarta EE. No início de 2018, a Oracle iniciou o grande esforço de licenciar novamente e contribuir com as tecnologias Java EE, incluindo APIs, RIs, TCKs e documentação do projeto associado ao projeto EE4J.

## MicroProfile
{: #microprofile}

Embora o Java EE forneça uma base sólida para criar microsserviços, ele precisava de tecnologias e modelos de programação para atender melhor os aplicativos de microsserviços. A IBM e outras empresas trabalharam juntas para lançar o MicroProfile, uma colaboração aberta entre os desenvolvedores, a comunidade e os fornecedores.

A comunidade MicroProfile está focada na inovação rápida dos microsserviços e do Enterprise Java. Essa comunidade constrói e integra tecnologias que são mais adequadas para aplicativos nativos de nuvem que seguem padrões arquiteturais de microsserviços. Cada liberação do MicroProfile contém um conjunto de tecnologias. O MicroProfile 1.0, liberado em 2016, continha um subconjunto de especificações do Java EE 7 que fornecem um conjunto de recursos principais para microsserviços leves: CDI 1.2, JAX-RS 2.0 e JSON-P 1.0. A plataforma MicroProfile inclui tecnologias para tratar de questões de nuvem exclusivas, como injeção de configuração, tolerância a falhas, verificações de funcionamento, métricas, rastreio aberto. Ela também define os padrões agnósticos do fornecedor para trabalhar com definições OpenAPI e criar clientes REST que tornam mais fácil consumir APIs de REST e para que a propagação do JWT trate das questões de segurança do aplicativo.

Para começar a participar do grupo de software livre, visite [https://microprofile.io/](https://microprofile.io/){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") ou [https://projects.eclipse.org/projects/technology.microprofile](https://projects.eclipse.org/projects/technology.microprofile){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo").
