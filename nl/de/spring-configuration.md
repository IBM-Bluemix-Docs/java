---

copyright:
  years: 2018, 2019
lastupdated: "2019-06-10"

keywords: spring environment, spring credentials, ibm-cloud-spring-boot-service-bind, service bindings spring, vcap_services spring, access credential spring

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

# Spring-Umgebung konfigurieren
{: #spring-configuration}

Die Verwaltung der Servicekonfiguration und der Berechtigungsnachweise (Servicebindungen) unterscheidet sich zwischen den Plattformen. Cloud Foundry speichert Servicebindungsdetails in einem in Zeichenfolgen konvertierten JSON-Objekt, das an die Anwendung in Form einer Umgebungsvariablen `VCAP_SERVICES` übergeben wird. Kubernetes speichert Servicebindungen als in Zeichenfolgen konvertierte JSON- oder unstrukturierte Attribute in `ConfigMaps` oder `Secrets`, die als Umgebungsvariablen an die containerisierte App übergeben oder als temporäre Datenträger angehängt werden können. Die lokale Entwicklung kann variieren. Diese Variationen auf flexible Art aufzufangen, ohne umgebungsspezifische Codepfade zu verwenden, kann eine Herausforderung sein.

## {{site.data.keyword.cloud_notm}}-Konfiguration zu vorhandenen Spring-Apps hinzufügen
{: #spring-cloud-env}

Die Bibliothek [{{site.data.keyword.cloud_notm}} Spring Service Bindings](https://github.com/ibm-developer/ibm-cloud-spring-bind){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") bietet eine konsistente Methode für den Zugriff Ihrer Spring-App auf die Konfiguration, indem Informationen aus unterschiedlichen Cloudumgebungen abstrahiert werden. Dabei werden die lokale Umgebung, Cloud Foundry, Cloud Foundry Enterprise Environment, Kubernetes, virtuelle Maschinen und {{site.data.keyword.openwhisk}} in ein Array von Suchmustern in einer Datei `mappings.json` eingeschlossen. Wenn die Spring-App gestartet wird, liest die Bibliothek die Datei und wendet die Muster an, um Konfigurationswerte zu erkennen und zuzuordnen. 

Weitere Informationen zur Datei `mappings.json` finden Sie im Abschnitt zur [Arbeit mit injizierten Serviceberechtigungsnachweisen](/docs/java?topic=cloud-native-configuration#portable-credentials).

### Bibliothek zu Ihrer Spring-App hinzufügen
{: #spring-add-service-library}

Um die Bibliothek für {{site.data.keyword.cloud_notm}}-Servicebindungen für Spring Boot zu verwenden, schließen Sie die Abhängigkeit `ibm-cloud-spring-boot-service-bind` in die Datei `pom.xml` oder `build.gradle` ein:

**Maven-Beispiel**:

```xml
<dependency>
   <groupId>com.ibm.cloud</groupId>
   <artifactId>ibm-cloud-spring-boot-service-bind</artifactId>
   <version>1.1.1</version>
</dependency>
```
{: codeblock}

**Gradle-Beispiel**:

```json
dependencies {
    compile group: 'com.ibm.cloud', name: 'ibm-cloud-spring-boot-service-bind', version: '1.1.1'
}
```
{: codeblock}

### Auf Berechtigungsnachweise zugreifen
{: #spring-access-credentials}

Für den Zugriff auf die Servicekonfiguration in Ihrem Code können Sie die Annotation `@Value` oder die Methode `getProperty()` der Klasse 'Environment' des Spring-Frameworks verwenden.

Beispiel: Auf Konfigurationsdaten für einen IBM Cloudant NoSQL-Datenbankservice zugreifen:

```java
   // Annotation '@Value' verwenden
   @Value("cloudant.url")
   String cloudantUrl;

   // Spring-Objekt 'Environment' verwenden
   @Autowired
   Environment env;

   String cloudantUsername = 
        env.getProperty("cloudant.username");
```
{: codeblock}

## Nächste Schritte
{: #spring-config-next-steps notoc}

Weitere Informationen zu Spring Boot finden Sie hier:

* [Spring Boot 1.5.x](https://docs.spring.io/spring-boot/docs/1.5.x/reference/html/){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link")
* [Spring Boot 2.1.x](https://docs.spring.io/spring-boot/docs/2.1.x/reference/html/){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link")

Weitere Informationen zu {{site.data.keyword.IBM}} und Spring: 

* [IBM Developer: Spring](https://developer.ibm.com/technologies/spring/){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link")
* [IBM Cloud-Servicebindungen für Spring Boot](https://github.com/ibm-developer/ibm-cloud-spring-bind){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link")
