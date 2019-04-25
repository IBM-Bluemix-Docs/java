---

copyright:
  years: 2019
lastupdated: "2019-04-01"

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

# Java on {{site.data.keyword.cloud_notm}}
{: #overview}

Os aplicativos Java&trade; vêm em todas as formas e tamanhos, tudo de aplicativos independentes para os aplicativos Java EE/Jakarta EE, MicroProfile ou Spring.

## OpenJDK com Open J9
{: #openjdk}

O projeto OpenJDK&trade; é a implementação de referência de software livre da linguagem Java e fornece estabilidade e compatibilidade de API para os programas do usuário.  A [Plataforma Java SE](https://docs.oracle.com/javase/8/docs/){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo"), consiste em APIs e na Java virtual machine (JVM). A JVM é o mecanismo de execução responsável pela execução do aplicativo Java.

O [projeto Eclipse OpenJ9](https://www.eclipse.org/openj9/index.html){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") fornece uma JVM alternativa que substitui o Ponto de acesso da JVM em OpenJDK. A JVM OpenJ9 tornou-se um software livre pela {{site.data.keyword.IBM}} em setembro de 2017, embora sua linhagem remonte muito mais longe do que isso. O OpenJ9 tem fortalecido mais de 3.000 produtos {{site.data.keyword.IBM}}, incluindo o {{site.data.keyword.appserver_short}}, por mais de uma década.

Juntos, o OpenJDK e o Eclipse OpenJ9 fornecem desempenho e capacidade de manutenção aprimorados para aplicativos Java nativos de nuvem acima e além dos tempos de execução Java baseados em Pontos de acesso tradicionais.

### Fazendo download do OpenJDK com OpenJ9
{: #downloads}

Os binários OpenJDK com OpenJ9 estão disponíveis gratuitamente por meio do hub da comunidade Java em [AdoptOpenJDK](https://adoptopenjdk.net/releases.html?variant=openjdk8&jvmVariant=openj9){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo"). O AdoptOpenJDK fornece binários OpenJDK pré-construídos por meio de um conjunto de software totalmente livre de scripts de construção e de infraestrutura. As imagens do Docker também estão disponíveis no [Dockerhub](https://hub.docker.com/u/adoptopenjdk){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo").

## Estruturas Java
{: #frameworks}

O OpenJDK e o OpenJ9 fornecem uma base sólida para qualquer aplicativo Java, mas as estruturas do aplicativo são frequentemente usadas para tratar de questões comuns. Os aplicativos corporativos, por exemplo, são geralmente construídos usando o [Java EE (e o Jakarta EE)](/docs/java?topic=java-jee-overview#jakarta-ee) ou o [Spring](/docs/java?topic=java-spring-overview). O [MicroProfile](/docs/java?topic=java-jee-overview#microprofile) traz um ritmo de desenvolvimento mais rápido e suporte para conceitos nativos da nuvem para Java EE.

O [{{site.data.keyword.runtime_liberty_full}}](/docs/java?topic=java-liberty) é um servidor de aplicativos Java rápido, dinâmico e fácil de usar, com base no projeto Open Liberty de software livre. Ele fornece um tempo de execução flexível e adequado para o uso corporativo para aplicativos Java EE e Spring.
