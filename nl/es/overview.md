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

# Java en {{site.data.keyword.cloud_notm}}
{: #overview}

Hay aplicaciones Java&trade; de todas las formas y tamaños, desde aplicaciones autónomas hasta aplicaciones Java EE/Jakarta EE, MicroProfile o Spring.

## OpenJDK con Open J9
{: #openjdk}

El proyecto OpenJDK&trade; es la implementación de referencia de código abierto del lenguaje Java y proporciona estabilidad y compatibilidad de API a los programas de usuario.  La [plataforma Java SE](https://docs.oracle.com/javase/8/docs/){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo"), consiste en API y la máquina virtual Java (JVM).  La JVM es el motor de ejecución responsable de la ejecución de la aplicación Java.

El [proyecto Eclipse OpenJ9](https://www.eclipse.org/openj9/index.html){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo") proporciona una JVM alternativa que sustituye a la JVM Hotspot en OpenJDK. {{site.data.keyword.IBM}} convirtió la JVM OpenJ9 en código abierto en septiembre de 2017, aunque su origen se remonta mucho más allá. OpenJ9 ha impulsado más de 3000 productos de {{site.data.keyword.IBM}}, incluido {{site.data.keyword.appserver_short}}, durante más de una década.

Juntos, OpenJDK y Eclipse OpenJ9 proporcionan un mejor rendimiento y capacidad de servicio para aplicaciones Java nativas de nube más allá de los tiempos tradicionales de ejecución Java basados en Hotspot.

### Descarga de OpenJDK con OpenJ9
{: #downloads}

Los binarios de OpenJDK con OpenJ9 están disponibles de forma gratuita en el hub de la comunidad de Java en [AdoptOpenJDK](https://adoptopenjdk.net/releases.html?variant=openjdk8&jvmVariant=openj9){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo"). AdoptOpenJDK proporciona binarios de OpenJDK precompilados a partir de un conjunto de código totalmente abierto de infraestructura y scripts de compilación. También hay imágenes de Docker disponibles en [Dockerhub](https://hub.docker.com/u/adoptopenjdk){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo").

## Infraestructuras de Java
{: #frameworks}

OpenJDK y OpenJ9 proporcionan una base sólida para cualquier aplicación Java, pero las infraestructuras de aplicación se utilizan a menudo para abordar los problemas comunes. Las aplicaciones empresariales, por ejemplo, se suelen compilar utilizando [Java EE (y Jakarta EE)](/docs/java?topic=java-jee-overview#jakarta-ee) o [Spring](/docs/java?topic=java-spring-overview).  [MicroProfile](/docs/java?topic=java-jee-overview#microprofile) proporciona un ritmo de desarrollo más rápido y soporte para los conceptos nativos de cloud a Java EE.

[{{site.data.keyword.runtime_liberty_full}}](/docs/java?topic=java-liberty) es un servidor de aplicaciones Java rápido, dinámico y fácil de utilizar, basado en el proyecto Open Liberty de código abierto. Proporciona un tiempo de ejecución de grado empresarial flexible y adaptado para las aplicaciones Java EE y Spring.
