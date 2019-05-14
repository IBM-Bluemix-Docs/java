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

# Configurazione con MicroProfile
{: #mp-configuration}

L'API MicroProfile Config consente la memorizzazione delle proprietà di configurazione dell'applicazione in più origini, in modo che puoi scrivere il tuo codice senza percorsi di codice specifici per l'ambiente.L'API utilizza [CDI](/docs/java?topic=java-mp-cdi#mp-cdi) per inserire le proprietà con dei valori specificati nella tua applicazione.

## Prerequisiti
{: #mp-config-prereq}

Per utilizzare l'API MicroProfile Config nella tua applicazione, devi aggiungere le seguenti dipendenze al tuo file `pom.xml`:

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

## Inserimento della configurazione
{: #mp-config-inject}

Nella classe bean CDI, utilizza le seguenti importazioni:

```java
//CDI 1.2
import javax.inject.Inject;

//mpConfig 1.2
import org.eclipse.microprofile.config.inject.ConfigProperty;
```
{: codeblock}

Per inserire il valore di una proprietà denominata `WATSON_URL` nella variabile `watsonService`, vedi questo esempio:

```java
@Inject
@ConfigProperty(name = "WATSON_URL")
String watsonService;
```
{: codeblock}

Puoi anche specificare un valore predefinito che viene utilizzato se non è possibile trovare la proprietà denominata:

```java
@Inject
@ConfigProperty(name = "WATSON_URL", defaultValue = "https://gateway.watsonplatform.net/tone-analyzer/api/v3/tone?version=2017-09-21") 
String watsonService;
```
{: codeblock}

E questo è tutto.

Per ulteriori informazioni su MicroProfile Config, vedi:

* [Eclipse MicroProfile Config – what is it?](https://www.eclipse.org/community/eclipse_newsletter/2017/september/article3.php){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno")
* OpenLiberty Guide: [Configuring microservices](https://openliberty.io/guides/microprofile-config.html){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno")
