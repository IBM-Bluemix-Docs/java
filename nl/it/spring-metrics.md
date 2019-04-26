---

copyright:
  years: 2019
lastupdated: "2019-04-09"

keywords: spring metrics, configure metrics spring, micrometer spring, micrometer, spring boot 2, spring actuator, prometheus spring

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

# Metriche con Spring
{: #spring-metrics}

A partire da Spring Framework 5, le metriche in Spring sono ora gestite da Micrometer. Micrometer è un framework che descrive se stesso come "SLF4J per le metriche". Proprio come SLF4J funge da API indipendente dal fornitore per la registrazione che può connettersi a diversi backend di registrazione, Micrometer fornisce un'API indipendente dal fornitore con cui strumentare e misurare il tuo codice e inoltrare quindi tali metriche a una gamma di aggregatori di metriche, come Prometheus, DataDog o Influx/Telegraph. 

Il framework Micrometer consente a Spring di integrarsi in un'ampia gamma di architetture native del cloud. L'aggiunta del supporto per Prometheus o Statsd è una semplice questione di modifica delle dipendenze e, se il raccoglitore di metriche è basato sui push, fornendo le informazioni sulla destinazione in `application.properties`, Spring and Micrometer determinano cosa fare al runtime in base a quali dipendenze trovano nel percorso classi dell'applicazione.

## Abilitazione delle metriche
{: #spring-metrics-enabling}

Spring Boot Actuator fornisce delle metriche predefinite di base, cosa che richiede la dipendenza `spring-boot-starter-actuator` nel tuo file `pom.xml`:  

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-actuator</artifactId>
</dependency>
```
{: codeblock}

Se sei ancora su Spring Boot 1.5.x, le metriche (e gli actuator) funzionano in modo leggermente diverso. Micrometer è stato indicato come standard per le metriche Spring in Spring Boot 2.0, ma sono stati resi disponibili dei backport per Boot 1.5.x, abilitando delle prassi congruenti per la creazione e la raccolta di metriche nelle applicazioni Spring Boot native del cloud. Ancora più importante, Micrometer supporta una gamma di sistemi di monitoraggio, compreso Prometheus, che lo rende la scelta migliore rispetto al sistema di metriche Boot 1,5,x per le distribuzioni cloud.
{: note}

## Metriche di Micrometer
{: #spring-metrics-micrometer}

Micrometer astrae il concetto di una destinazione per le metriche mediante la sua interfaccia `MeterRegistry`. Il `MeterRegistry` consente all'applicazione di ottenere contatori, misuratori e timer personalizzati ecc. Se sono necessarie più destinazioni, Micrometer fornisce un'implementazione `CompositeMeterRegistry`, consentendo all'applicazione di essere isolata dalle effettive destinazioni configurate per le metriche.

In entrambe le versioni di Spring Boot, quando un endpoint delle metriche basato su Micrometer viene abilitato, Spring Boot crea automaticamente un `CompositeMeterRegistry` e lo rende disponibile all'applicazione come un `Bean`. Il registro viene popolato automaticamente con le implementazioni `MeterRegistry` supportate rilevate nel percorso classi. Supportare più sistemi di monitoraggio, o passare dall'uno all'altro, è quindi una semplice questione di modifica delle dipendenze in pom.xml. 

Spring consente la personalizzazione del Bean `MeterRegistry` mediante un `MeterRegistryCustomizer`, esso stesso un Bean. Quando Spring identifica la presenza di un tale Bean, avrà l'opportunità di configurare il `MeterRegistry` globale prima che venga notifica alcuna metrica. Ciò viene spesso utilizzato per aggiungere tag comuni al `MeterRegistry`, ad es. DataCenter o EnvironmentType.

Quando si denominano le metriche, Spring e Micrometer raccomandano di rispettare una convenzione di denominazione che suddivide le parole con un `.` invece di utilizzare la notazione a cammello (ossia le parole unite tra di loro lasciando però le loro lettere iniziali in maiuscolo), `_` o altri approcci. Questo perché Micrometer inoltrerà queste metriche alla destinazione o alle destinazioni configurate ed eseguirà le eventuali conversioni di nome come necessario per soddisfare i requisiti delle destinazioni.

## Configurazione delle metriche con Spring Boot
{: #spring-metrics-configuration}

Le seguenti sezioni descriveranno come abilitare le metriche Spring Boot utilizzando Micrometer per raccogliere le metriche e fornire un endpoint per Prometheus per ciascuna release, iniziando con Spring Boot 2 come la più corrente.

### Configurazione delle metriche in Spring Boot 2
{: #spring-metrics-boot2}

L'endpoint delle metriche dell'Actuator di Spring Boot 2 deve essere sia abilitato che esposto per l'accesso web. Per impostazione predefinita, l'endpoint è abilitato ma non esposto (per ulteriori informazioni, consulta [la documentazione di Spring Endpoint](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-endpoints.html){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno")).  

Aggiungi quanto segue a `application.properties` per esporre l'endpoint delle metriche:

```properties
management.endpoints.web.exposure.include=health,metrics
```
{: codeblock}

Verifica che l'endpoint delle metriche sia stato abilitato controllando l'endpoint `/actuator`. L'output sarà simile al seguente, in caso di esecuzione dell'applicazione in locale (utilizzando `jq` per una riproduzione in formato pretty print della risposta json).

```
$ curl -s localhost:8080/actuator | jq .
{
  "_links": {
    "self": {
      "href": "http://localhost:8080/actuator",
      "templated": false
    },
    ...
    "metrics": {
      "href": "http://localhost:8080/actuator/metrics",
      "templated": false
    },
    "metrics-requiredMetricName": {
      "href": "http://localhost:8080/actuator/metrics/{requiredMetricName}",
      "templated": true
    },
    ...
  }
}
```
{: screen}

L'accesso all'endpoint `/actuator/metrics` visualizza un elenco con formattazione json di metriche disponibili che ha un aspetto simile al seguente:

```
$ curl -s localhost:8080/actuator/metrics | jq .
{
  "names": [
    "jvm.memory.max",
    "jvm.threads.states",
    "process.files.max",
    "jvm.gc.memory.promoted",
    ...
  ]
}
```
{: screen}

È necessaria una richiesta aggiuntiva per visualizzare il valore effettivo di una singola metrica (come si può dedurre dall'elenco di URL degli actuator). Per richiamare gli stati dei thread jvm, ad esempio:

```
$ curl -s localhost:8080/actuator/metrics/jvm.threads.states | jq .
{
  "name": "jvm.threads.states",
  "description": "The current number of threads having TERMINATED state",
  "baseUnit": "threads",
  "measurements": [
    {
    "statistic": "VALUE",
    "value": 24
    }
  ],
  "availableTags": [
    {
      "tag": "state",
      "values": [
        "timed-waiting",
        "new",
        "runnable",
        "blocked",
        "waiting",
        "terminated"
      ]
    }
  ]
}
```
{: screen}

Per ulteriori informazioni su come personalizzare la modalità di funzionamento dell'actuator delle metriche, vedi la [documentazione ufficiale](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-metrics.html){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno").

### Abilitazione del supporto Prometheus in Spring Boot 2
{: #spring-prometheus-boot2}

Prometheus richiede che l'applicazione ospiti un endpoint da cui leggerà il server Prometheus per ottenere le metriche. L'hosting di questo endpoint in Spring si basa sul fatto che l'applicazione sia già in grado di servire i dati, il che significa che l'endpoint Prometheus supporta solo le applicazioni Spring Boot che utilizzano Spring MVC, Spring WebFlux o Jersey.

Per abilitare l'endpoint Prometheus, aggiungi la seguente ulteriore dipendenza in `pom.xml`:

```xml
<dependency>
  <groupId>io.micrometer</groupId>
  <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```
{: codeblock}

Spring rileverà la dipendenza e configurerà automaticamente PrometheusMeterRegistry come uno dei registri configurati nel bean MeterRegistry globale. Tuttavia, come per l'endpoint delle metriche, l'endpoint Prometheus deve essere abilitato ed esposto per l'accesso web aggiungendo quanto segue alle proprietà dell'applicazione.

```properties
management.endpoints.web.exposure.include=health,metrics,prometheus
```
{: codeblock}

Verifica che l'endpoint Prometheus sia stato abilitato controllando l'endpoint `/actuator`. L'output sarà simile al seguente, in caso di esecuzione dell'applicazione in locale:

```
$ curl -s localhost:8080/actuator | jq .
{
  "_links": {
    "self": {
      "href": "http://localhost:8080/actuator",
      "templated": false
    },
    ...
    "prometheus": {
      "href": "http://localhost:8080/actuator/prometheus",
      "templated": false
    },
    ...
  }
}
```
{: screen}

L'endpoint `/actuator/prometheus` emetterà tutte le metriche configurate in formato di scraping Prometheus.

```
$ curl -s localhost:8080/actuator/prometheus
# HELP system_cpu_usage The "recent cpu usage" for the whole system
# TYPE system_cpu_usage gauge
system_cpu_usage 0.046231032221805926
...
```
{: screen}

Aggiungi le seguenti annotazioni alla tua definizione del servizio per abilitare lo scraping Prometheus per l'endpoint delle metriche:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: ...
  annotations:
    prometheus.io/scrape: 'true'
    prometheus.io/scheme: http
    prometheus.io/path: /actuator/prometheus
    prometheus.io/port: "8080"
  ...
```
{: codeblock}

### Configurazione delle metriche in Spring Boot 1
{: #spring-metrics-boot1}

L'endpoint `/metrics` di Spring Boot Actuator è abilitato per impostazione predefinita in Spring Boot 1, ma è considerato sensibile (“sensitive”) e richiede l'autorizzazione. Per alcuni ambienti, la protezione dell'endpoint delle metriche potrebbe essere la risposta giusta, ma autorizzare ogni richiesta all'endpoint delle metriche introduce una latenza eccessiva per il controllo automatizzato con Kubernetes. Disabilita l'attributo sensitive dell'endpoint delle metriche per rimuovere il requisito di autorizzazione aggiungendo quanto segue ad application.properties:

```properties
endpoints.metrics.sensitive=false
```
{: codeblock}

Questo abilita il supporto delle metriche Spring Boot 1 predefinito, che produce un output simile al seguente:

```
$ curl -s localhost:8080/metrics | jq .
{
  "names": [
    "jvm.memory.max",
    "jvm.threads.states",
    "process.files.max",
    "jvm.gc.memory.promoted",
    ...
  ]
}
```
{: screen}

Per ulteriori informazioni su come personalizzare la modalità di funzionamento dell'actuator delle metriche Spring Boot 1, vedi la [documentazione ufficiale](https://docs.spring.io/spring-boot/docs/1.5.x/reference/html/production-ready-metrics.html){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno").

### Abilitazione del supporto Prometheus in Spring Boot 1
{: #spring-prometheus-boot1}

Il supporto Prometheus non è integrato nelle metriche di Spring Boot 1 Actuator ma possiamo aggiungerne il supporto mediante Micrometer.

Come menzionato nella [panoramica](#spring-metrics), è stata eseguita una modifica della versione di Micrometer a Spring Boot 1 perché ha un insieme più ricco di primitive delle metriche e un migliore supporto per i sistemi di monitoraggio come Prometheus e StatsD. L'utilizzo di Micrometer con Spring Boot 1 è piuttosto semplice: richiede una specifica libreria di adattatori e almeno un registro. Aggiungi quanto segue al tuo `pom.xml` per includere le dipendenze per le librerie di adattatori e il registro Prometheus:

```xml
<properties>
  ...
  <micrometer.version>1.1.1</micrometer.version>
</properties>
<dependencies>
  <dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-spring-legacy</artifactId>
    <version>${micrometer.version}</version>
  </dependency>
  <dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
    <version>${micrometer.version}</version>
  </dependency>
</dependencies>
```
{: codeblock}

Queste dipendenze creeranno ed esporranno un endpoint delle metriche `/prometheus`. Come succede con l'endpoint `/metrics` predefinito, l'endpoint `/prometheus` è, per impostazione predefinita, considerato sensibile (sensitive). Per rimuovere il requisito di autorizzazione, aggiungi quanto segue ad application.properties:

```properties
endpoints.prometheus.sensitive=false
```
{: codeblock}

L'output prodotto dall'endpoint /prometheus è simile a quello per Spring Boot 2:

```
$ curl -s localhost:8080/prometheus
# HELP jvm_memory_used_bytes The amount of used memory
# TYPE jvm_memory_used_bytes gauge
jvm_memory_used_bytes{area="heap",id="PS Survivor Space",} 2.2594152E7
...
```
{: screen}

Aggiungi le seguenti annotazioni alla tua definizione del servizio per abilitare lo scraping Prometheus per l'endpoint delle metriche:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: …
  annotations:
    prometheus.io/scrape: 'true'
    prometheus.io/scheme: http
    prometheus.io/path: /prometheus
    prometheus.io/port: "8080"
  ...
```
{: codeblock}

Per ulteriori informazioni sull'utilizzo di Micrometer con Spring Boot 1, vedi [https://micrometer.io/docs/ref/spring/1.5](https://micrometer.io/docs/ref/spring/1.5){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno").

## Richiami della tempistica con Micrometer
{: #spring-metrics-timing}

Spring MVC, WebFlux e Jersey Server forniscono ognuno una configurazione automatica per Spring che raccoglie automaticamente le informazioni sulla tempistica per tutte le richieste quando la proprietà `management.metrics.web.server.auto-time-requests` è impostata su `true`. Questo è il valore predefinito.

```properties
management.metrics.web.server.auto-time-requests=true
```
{: codeblock}

Micrometer supporta anche l'annotazione `@Timed`, che può essere utilizzata per raccogliere le informazioni sulla tempistica per le chiamate di metodo supportate. Vale la pena notare che questa annotazione non può essere utilizzata per metodi arbitrari di tempo poiché richiede il supporto contenitore per raccogliere i dati. Le annotazioni `@Timed` sono supportate per le operazioni supportate da un contenitore servlet, ad esempio i metodi del servizio Spring MVC, Webflux o Jersey.

Per essere più selettivo in merito a quali informazioni di tempistica vengono raccolte, imposta la proprietà `management.metrics.web.server.auto-time-requests` su `false` e utilizza l'annotazione `@Timed` su singoli metodi del servizio o controller.

```java
@Timed
@GetMapping("/myendpoint")
public String getSomething() { ... }
```
{: codeblock}

Se `management.metrics.web.server.auto-time-requests` è `true`, l'annotazione `@Timed` può essere utilizzata per aggiungere delle tag supplementari a una specifica metrica notificata, ad esempio.

```java
@Timed(extraTags = { "mytagname", "mytagvalue"})
@GetMapping("/myendpoint")
public String getSomething() { ... }
```
{: codeblock}

## Metriche personalizzate con Spring e Micrometer
{: #spring-metrics-custom}

Per effettuare delle misurazioni specifiche per il dominio dell'applicazione, dovrai creare delle tue metriche personalizzate. Micrometer definisce alcuni tipi `Meter` di base.

* Un `Counter` traccia un valore che può solo aumentare.
* Un `Gauge` misura e restituisce il valore osservato quando il contatore viene pubblicato (o ne viene eseguita la query).
* A `Timer` tiene traccia di quante volte si è verificato un evento e il tempo trascorso cumulativo per tale evento. 
* A `DistributionSummary` è simile a un `Timer`, tiene anche traccia di una distribuzione di eventi ma non misura il tempo.

I nuovi contatori vengono creati utilizzando i metodi helper sul `MeterRegistry` oppure il fluent builder di Meter. 

Quando lavori nel tuo codice applicativo, crea il tuo `Meter` nel constructor (passando il `MeterRegistry` come un parametro) oppure in un metodo `@PostConstruct` dopo che è stato inserito il `MeterRegistry`.

Ad esempio, per utilizzare un `Counter` in un `RestController` potresti fare qualcosa di simile a quanto segue:

```java
@RestController
public class MyController {
    
    private final Counter myCounter;
    
    public MyController(MeterRegistry meterRegistry) {
    
        // Create the counter using the helper method on the builder
        myCounter = meterRegistry.counter("my.counter", "mytagname", "mytagvalue");
        
    }
```
{: codeblock}

Quindi, nei tuoi metodi di servizio, puoi aggiornare il valore del contatore in modo appropriato:

```java
@GetMapping("/")
public String getSomething() {
    myCounter.increment();
    ...
}
```
{: codeblock}


Per ulteriori informazioni sulla creazione di metriche personalizzate in Spring:

* [Metriche di Spring Boot](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-metrics.html){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno")
* [Concetti di Micrometer](https://micrometer.io/docs/concepts){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno")
