---

copyright:
  years: 2019
lastupdated: "2019-06-10"

keywords: openjdk, open j9, download openjdk, java frameworks, adoptopenjdk, eclipse openj9, openj9 binaries, openjdk binaries, microprofile framework, jakarta

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

# Tutorial de introdução
{: #getting-started}

Os aplicativos do Java&trade; são fornecidos em todas as formas e tamanhos, tudo desde apps independentes até apps do Java&trade; EE /do Jakarta EE, do MicroProfile ou do Spring.

## OpenJDK com Open J9
{: #openjdk}

O projeto do OpenJDK&trade; é a implementação de referência de software livre da linguagem Java&trade; e fornece estabilidade e compatibilidade de API para programas do usuário. A plataforma [Java&trade; Platform, Standard Edition](https://docs.oracle.com/javase/8/docs/){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo"), consiste em APIs e na máquina virtual (JVM) do Java&trade;. A JVM é o mecanismo de execução responsável pela execução do app do Java&trade;.

O [projeto Eclipse OpenJ9](https://www.eclipse.org/openj9/index.html){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") fornece uma JVM alternativa que substitui o Ponto de acesso da JVM em OpenJDK. A OpenJ9 JVM foi colocada como software livre pela {{site.data.keyword.IBM}} em setembro de 2017, embora a sua linhagem volte para muito mais longe. O OpenJ9 capacita mais de 3000 produtos {{site.data.keyword.IBM}}, incluindo o {{site.data.keyword.appserver_short}}, por mais de uma década.

Juntos, o OpenJDK e o Eclipse OpenJ9 fornecem melhor desempenho e capacidade de manutenção para apps do Java&trade; nativos de nuvem acima e além dos tempos de execução do Java&trade; baseados em Hotspot tradicional.

### Fazendo download do OpenJDK com OpenJ9
{: #downloads}

O OpenJDK com binários do OpenJ9 estão disponíveis gratuitamente por meio do hub de comunidade do Java&trade; em [AdoptOpenJDK](https://adoptopenjdk.net/releases.html?variant=openjdk8&jvmVariant=openj9){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo"). O AdoptOpenJDK fornece binários OpenJDK pré-construídos por meio de um conjunto de software totalmente livre de scripts de construção e de infraestrutura. As imagens do Docker também estão disponíveis no [hub do Docker](https://hub.docker.com/u/adoptopenjdk){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo").

## Estruturas Java
{: #frameworks}

O OpenJDK e o OpenJ9 fornecem uma base sólida para qualquer app do Java&trade;, mas as estruturas de app são frequentemente usadas para tratar de interesses comuns. Os apps corporativos, por exemplo, são geralmente construídos usando [Java&trade; EE (e Jakarta EE)](/docs/java?topic=java-jee-overview#jakarta-ee) ou o [Spring](/docs/java?topic=java-spring-overview). O [MicroProfile](/docs/java?topic=java-jee-overview#microprofile) traz um ritmo de desenvolvimento mais rápido e suporte para conceitos nativos de nuvem para o Java&trade; EE.

O [{{site.data.keyword.runtime_liberty_full}}](/docs/java?topic=java-liberty) é um servidor de aplicativos do Java&trade; rápido, dinâmico e fácil de usar, com base no projeto do Open Liberty de software livre. Ele fornece um tempo de execução flexível, adequado à finalidade e de nível corporativo para apps do Java&trade; EE e do Spring.
