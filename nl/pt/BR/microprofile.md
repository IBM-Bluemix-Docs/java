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

As tecnologias que são introduzidas no Java&trade; EE7 e EE8 são bem adequadas para criar microsserviços no Java&trade;, com suporte para anotações, protocolos leves e padrões de interação assíncrona. Os utilitários de conversão no Java&trade; EE7 fornecem um mecanismo direto para o uso de bibliotecas reativas como RxJava de uma maneira fácil para contêiner.

Em setembro de 2017, a Oracle, suportada pelo {{site.data.keyword.IBM}} e pelo Red Hat, anunciou que o Java&trade; EE ia se mudar para a Eclipse Foundation, como Jakarta EE. A Oracle continua a ter tudo o que está associado ao Java&trade; EE, até e incluindo o Java&trade; EE 8. Mas todo o desenvolvimento futuro da plataforma do Enterprise Java&trade; após o Java&trade; EE 8 é manipulado pela Eclipse Foundation sob a orientação do grupo de trabalho do Jakarta EE. No início de 2018, a Oracle iniciou o grande esforço de relicenciar e contribuir com as tecnologias do Java&trade; EE que incluem APIs, RIs, TCKs e documentação associada do projeto para o projeto EE4J.

## MicroProfile
{: #microprofile}

Embora o Java&trade; EE forneça uma base sólida para criar microsserviços, ele precisava de tecnologias e modelos de programação para atender melhor aos aplicativos de microsserviço. A IBM e outras empresas trabalharam juntas para lançar o MicroProfile, uma colaboração aberta entre os desenvolvedores, a comunidade e os fornecedores.

A comunidade do MicroProfile está focada na inovação rápida em torno dos microsserviços e no Enterprise Java&trade;. Essa comunidade constrói e integra tecnologias que são mais adequadas para apps nativos de nuvem que seguem padrões arquiteturais de microsserviços. Cada liberação do MicroProfile contém um conjunto de tecnologias. O MicroProfile 1.0, liberado em 2016, continha um subconjunto de especificações do Java&trade; EE 7 que fornecem um conjunto de recursos principais para microsserviços leves: CDI 1.2, JAX-RS 2.0 e JSON-P 1.0. A plataforma MicroProfile inclui tecnologias para tratar de questões de nuvem exclusivas, como injeção de configuração, tolerância a falhas, verificações de funcionamento, métricas, rastreio aberto. Ela também define os padrões agnósticos do fornecedor para trabalhar com definições do OpenAPI e criar clientes de REST que tornam mais fácil consumir APIs de REST e para a propagação de JWT para atender a interesses de segurança do app.

Para começar a participar do grupo de software livre, visite [https://microprofile.io/](https://microprofile.io/){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") ou [https://projects.eclipse.org/projects/technology.microprofile](https://projects.eclipse.org/projects/technology.microprofile){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo").
