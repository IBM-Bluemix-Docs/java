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

# Configurazione dell'ambiente Spring
{: #spring-configuration}

La gestione della configurazione e delle credenziali del servizio (bind dei servizi) varia tra le piattaforme. Cloud Foundry memorizza i dettagli di associazione del servizio in un oggetto JSON convertito in stringa che viene passato all'applicazione come una variabile di ambiente `VCAP_SERVICES`. Kubernetes memorizza i bind del servizio come JSON convertito in stringa o attributi semplici in `ConfigMaps` o `Secrets`, che possono essere passati all'applicazione inserita in un contenitore come variabili di ambiente o montati come volumi temporanei. Lo sviluppo locale può variare. Lavorare su queste variazioni in modo trasportabile senza avere percorsi di codice specifici per l'ambiente può essere una sfida.

## Aggiunta della configurazione {{site.data.keyword.cloud_notm}} a applicazioni Spring esistenti
{: #spring-cloud-env}

La libreria [{{site.data.keyword.cloud_notm}} Spring Service Bindings](https://github.com/ibm-developer/ibm-cloud-spring-bind){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno") fornisce un modo coerente per le tue applicazioni Spring di accedere alla configurazione separando le informazioni da ambienti cloud diversi. Inclusi l'ambiente locale, le macchine virtuali, Cloud Foundry, Cloud Foundry Enterprise Environment, Kubernetes e {{site.data.keyword.openwhisk}} in un array di modelli di ricerca in un file `mappings.json`. Quando viene avviata l'applicazione Spring, la libreria legge il file e applica i modelli per rilevare e assegnare i valori di configurazione.

Per ulteriori informazioni sul file `mappings.json`, vedi la sezione relativa alla [gestione delle credenziali del servizio inserite](/docs/java?topic=cloud-native-configuration#portable-credentials).

### Aggiunta della libreria alla tua applicazione Spring
{: #spring-add-service-library}

Per utilizzare la libreria {{site.data.keyword.cloud_notm}} Spring Service Bindings, includi la dipendenza `ibm-cloud-spring-boot-service-bind` nel tuo file `pom.xml` o `build.gradle`:

**Esempio Maven**:

```xml
<dependency>
   <groupId>com.ibm.cloud</groupId>
   <artifactId>ibm-cloud-spring-boot-service-bind</artifactId>
   <version>1.1.1</version>
</dependency>
```
{: codeblock}

**Esempio Gradle**:

```json
dependencies {
    compile group: 'com.ibm.cloud', name: 'ibm-cloud-spring-boot-service-bind', version: '1.1.1'
}
```
{: codeblock}

### Accesso alle credenziali
{: #spring-access-credentials}

Per accedere alla configurazione del servizio nel tuo codice puoi utilizzare l'annotazione `@Value` oppure il metodo Spring Framework Environment class `getProperty()`.

Esempio: accesso ai dati di configurazione per un servizio IBM Cloudant NoSQL DB:

```java
   // Utilizza l'annotazione @Value
   @Value("cloudant.url")
   String cloudantUrl;

   // Utilizza l'oggetto Spring Environment
   @Autowired
   Environment env;

   String cloudantUsername = 
        env.getProperty("cloudant.username");
```
{: codeblock}

## Passi successivi
{: #spring-config-next-steps notoc}

Per ulteriori informazioni su Spring Boot:

* [Spring Boot 1.5.x](https://docs.spring.io/spring-boot/docs/1.5.x/reference/html/){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno")
* [Spring Boot 2.1.x](https://docs.spring.io/spring-boot/docs/2.1.x/reference/html/){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno")

Per ulteriori informazioni su {{site.data.keyword.IBM}} e Spring:

* [IBM Developer: Spring](https://developer.ibm.com/technologies/spring/){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno")
* [IBM Spring Service Bindings](https://github.com/ibm-developer/ibm-cloud-spring-bind){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno")
