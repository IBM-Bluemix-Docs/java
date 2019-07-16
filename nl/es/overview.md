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

# Guía de aprendizaje de iniciación
{: #getting-started}

Hay aplicaciones Java&trade; de todas las formas y tamaños, desde apps autónomas hasta apps Java&trade; EE/Jakarta EE, MicroProfile o Spring.

## OpenJDK con Open J9
{: #openjdk}

El proyecto OpenJDK&trade; es la implementación de referencia de código abierto del lenguaje Java&trade; y proporciona estabilidad y compatibilidad de API a los programas de usuario. La [plataforma Java&trade; SE](https://docs.oracle.com/javase/8/docs/){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo"), consiste en API y la máquina virtual Java&trade; (JVM). La JVM es el motor de ejecución responsable de la ejecución de la app Java&trade;.

El [proyecto Eclipse OpenJ9](https://www.eclipse.org/openj9/index.html){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo") proporciona una JVM alternativa que sustituye a la JVM Hotspot en OpenJDK. {{site.data.keyword.IBM}} convirtió a código abierto la JVM de OpenJ9 en septiembre de 2017, aunque su linaje se remonta mucho más. OpenJ9 se ha utilizado en más de 3000 {{site.data.keyword.IBM}} productos, incluido {{site.data.keyword.appserver_short}}, durante más de una década.

Juntos, OpenJDK y Eclipse OpenJ9 proporcionan un mejor rendimiento y capacidad de servicio para apps Java&trade; nativas de nube más allá de los tiempos tradicionales de ejecución Java&trade; basados en Hotspot.

### Descarga de OpenJDK con OpenJ9
{: #downloads}

Los binarios de OpenJDK con OpenJ9 están disponibles de forma gratuita en el hub de la comunidad de Java&trade; en [AdoptOpenJDK](https://adoptopenjdk.net/releases.html?variant=openjdk8&jvmVariant=openj9){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo"). AdoptOpenJDK proporciona binarios de OpenJDK precompilados a partir de un conjunto de código totalmente abierto de infraestructura y scripts de compilación. Las imágenes de Docker también están disponibles en [Docker Hub](https://hub.docker.com/u/adoptopenjdk){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo").

## Infraestructuras de Java
{: #frameworks}

OpenJDK y OpenJ9 proporcionan una base sólida para cualquier app Java&trade;, pero las infraestructuras de app se utilizan a menudo para abordar los problemas comunes. Las apps empresariales, por ejemplo, se suelen compilar utilizando [Java&trade; EE (y Jakarta EE)](/docs/java?topic=java-jee-overview#jakarta-ee) o [Spring](/docs/java?topic=java-spring-overview). [MicroProfile](/docs/java?topic=java-jee-overview#microprofile) proporciona un ritmo de desarrollo más rápido y soporte para los conceptos nativos de cloud a Java&trade; EE.

[{{site.data.keyword.runtime_liberty_full}}](/docs/java?topic=java-liberty) es un servidor de aplicaciones Java&trade; rápido, dinámico y fácil de utilizar, basado en el proyecto Open Liberty de código abierto. Proporciona un tiempo de ejecución de grado empresarial flexible y adaptado para las apps Java&trade; EE y Spring.
