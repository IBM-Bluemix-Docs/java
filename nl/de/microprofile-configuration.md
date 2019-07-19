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

# Konfiguration mit MicroProfile
{: #mp-configuration}

Mit der API 'MicroProfile Config' können Anwendungskonfigurationseigenschaften in mehreren Quellen gespeichert werden, sodass Sie Ihren Code ohne umgebungsspezifische Codepfade schreiben können. Die API verwendet [CDI](/docs/java?topic=java-mp-cdi#mp-cdi), um die angegebenen Werte in Ihre Anwendung einzufügen.

## Voraussetzungen
{: #mp-config-prereq}

Um die API 'MicroProfile Config' in Ihrer Anwendung verwenden zu können, müssen Sie die folgenden Abhängigkeiten zu Ihrer Datei `pom.xml` hinzufügen:

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

## Konfiguration einfügen
{: #mp-config-inject}

Verwenden Sie in Ihrer CDI-Bean-Klasse die folgenden Importe:

```java
//CDI 1.2
import javax.inject.Inject;

//mpConfig 1.2
import org.eclipse.microprofile.config.inject.ConfigProperty;
```
{: codeblock}

Um den Wert einer Eigenschaft mit dem Namen `WATSON_URL` in die Variable `watsonService` einzufügen, verwenden Sie dieses Beispiel:

```java
@Inject
@ConfigProperty(name = "WATSON_URL")
String watsonService;
```
{: codeblock}

Sie können auch einen Standardwert angeben, der verwendet wird, wenn die benannte Eigenschaft nicht gefunden werden kann:

```java
@Inject 
@ConfigProperty(name = "WATSON_URL", defaultValue = "https://gateway.watsonplatform.net/tone-analyzer/api/v3/tone?version=2017-09-21") 
String watsonService;
```
{: codeblock}

Und so einfach ist es!

Weitere Informationen zu 'MicroProfile Config' finden Sie unter:

* [Eclipse 'MicroProfile Config' – Was ist das?](https://www.eclipse.org/community/eclipse_newsletter/2017/september/article3.php){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link")
* Leitfaden für OpenLiberty: [Mikroservices konfigurieren](https://openliberty.io/guides/microprofile-config.html){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link")
