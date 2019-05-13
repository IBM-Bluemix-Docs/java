---

copyright:
  years: 2019
lastupdated: "2019-04-22"

keywords: microprofile api, microprofile cdi, microprofile config api, config api, store properties multiple sources

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

# Configuración con MicroProfile
{: #mp-configuration}

La API de MicroProfile Config permite almacenar las propiedades de configuración de aplicaciones en varios orígenes, de manera que pueda escribir el código sin vías de acceso de código específicas del entorno. La API utiliza
[CDI](/docs/java?topic=java-mp-cdi#mp-cdi) para inyectar propiedades especificadas por valor en la aplicación.

## Requisitos previos
{: #mp-config-prereq}

Para utilizar la API de MicroProfile Config en la aplicación, debe añadir las siguientes dependencias al archivo `pom.xml`:

```xml
<dependency>
  <groupId>org.eclipse.microprofile</groupId>
  <artifactId>microprofile</artifactId>
  <version>2.1</version>
  <type>pom</type>
  <scope>provided</scope>
</dependency>
```
{: codeblock}

## Inyección de configuración
{: #mp-config-inject}

Dentro de la clase de bean de CDI, utilice las importaciones siguientes:

```java
//CDI 1.2
import javax.inject.Inject;

//mpConfig 1.2
import org.eclipse.microprofile.config.inject.ConfigProperty;
```
{: codeblock}

Para inyectar el valor de una propiedad llamada `WATSON_URL` en la variable `watsonService`, vea este ejemplo:

```java
@Inject 
@ConfigProperty(name = "WATSON_URL") 
String watsonService;
```
{: codeblock}

También puede especificar un valor predeterminado que se utilizará si no se puede encontrar la propiedad determinada:

```java
@Inject 
@ConfigProperty(name = "WATSON_URL", defaultValue = "https://gateway.watsonplatform.net/tone-analyzer/api/v3/tone?version=2017-09-21") 
String watsonService;
```
{: codeblock}

¡Y eso es todo!

Para obtener más información sobre MicroProfile Config, consulte:

* [Eclipse MicroProfile Config – ¿qué es?](https://www.eclipse.org/community/eclipse_newsletter/2017/september/article3.php){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo")
* Guía de OpenLiberty: [Configuración de microservicios](https://openliberty.io/guides/microprofile-config.html){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo")
