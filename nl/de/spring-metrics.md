---

copyright:
  years: 2019
lastupdated: "2019-06-10"

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

# Metriken mit Spring
{: #spring-metrics}

Ab Spring Framework 5 werden Metriken von Micrometer verwaltet. Micrometer ist ein Framework, das sich selbst als "SLF4J für Metriken" beschreibt. Genau wie SLF4J als anbieterneutrale API für die Protokollierung agiert, die eine Verbindung zu verschiedenen Protokollierungs-Back-Ends herstellt, stellt Micrometer eine anbieterneutrale API bereit, mit der Sie Ihren Code instrumentieren und messen können. Diese Metriken können dann einer Reihe von Metrikaggregatoren wie Prometheus, DataDog, InfluxDB und Telegraf zur Verfügung gestellt werden. 

Mit dem Micrometer-Framework kann Spring mit einer Vielzahl von cloudnativen Architekturen integriert werden. Wenn Sie Unterstützung für Prometheus oder Statsd hinzufügen möchten, müssen Sie nur einige Abhängigkeiten ändern und ggf. Zielinformationen in der Datei `application.properties` angeben, falls der Metrikcollector pushbasiert sein sollte. Spring und Micrometer legen fest, was während der Laufzeit auf der Basis der Abhängigkeiten, die im Klassenpfad der Anwendung gefunden werden, ausgeführt werden soll.

## Metriken aktivieren
{: #spring-metrics-enabling}

Grundlegende Metriken werden vom Spring Boot Actuator bereitgestellt, der die Abhängigkeit `spring-boot-starter-actuator` in der Datei `pom.xml` erfordert: 

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-actuator</artifactId>
</dependency>
```
{: codeblock}

Wenn Sie immer noch Spring Boot 1.5.x verwenden, verhalten sich Metriken (und Actuators) etwas anders. Micrometer wurde in Spring Boot 2.0 als Standard für Spring-Metriken verwendet. Für Boot 1.5.x wurden jedoch Backports zur Verfügung gestellt, die konsistente Verfahren für die Erstellung und Erfassung von Metriken ermöglichen. Darüber hinaus unterstützt Micrometer verschiedene Überwachungssysteme, darunter Prometheus, und ist somit für Cloudbereitstellungen die bessere Wahl als das Metriksystem von Boot 1.5.x.
{: note}

## Micrometer-Metriken
{: #spring-metrics-micrometer}

Micrometer abstrahiert das Konzept eines Ziels für Metriken über seine Schnittstelle `MeterRegistry`. Über die `MeterRegistry` kann die App angepasste Zähler, Messanzeigen und Zeitgeber abrufen. Werden mehrere Ziele benötigt, bietet Micrometer die Implementierung einer `CompositeMeterRegistry`, mit der die App von den tatsächlichen konfigurierten Zielen für Metriken isoliert werden kann. 

Wenn ein Micrometer-basierter Metrikendpunkt aktiviert ist, erstellt Spring Boot in jeder Spring Boot-Version automatisch eine `CompositeMeterRegistry` und stellt diese Registry der App als `Bean` zur Verfügung. Die Registry wird automatisch mit den unterstützten Implementierungen von `MeterRegistry` gefüllt, die im Klassenpfad gefunden wurden. Für die Unterstützung mehrere Überwachungssysteme oder das Umschalten zwischen solchen Systemen müssen Sie dann einfach nur die entsprechenden Abhängigkeiten in der Datei 'pom.xml' ändern. 

Spring ermöglicht die Anpassung der Bean `MeterRegistry` über `MeterRegistryCustomizer`, die selbst eine Bean ist. Wenn Spring eine solche Bean erkennt, kann das Framework die globale `MeterRegistry` konfigurieren, bevor Metriken gemeldet werden. Diese Anpassung wird häufig verwendet, um allgemeine Tags zu `MeterRegistry`, 'DataCenter' oder 'EnvironmentType' hinzuzufügen. 

Bei der Benennung von Metriken wird bei Spring und Micrometer die folgende Namenskonvention empfohlen, bei der Wörter durch einen Punkt (`.`) und nicht durch die Kamelschreibweise, einen Unterstrich (`_`) oder sonstige Methoden getrennt werden. Die Benennung ist wichtig, weil Micrometer diese Metriken an die konfigurierten Ziele überträgt und alle erforderlichen Namensumwandlungen ausführt, um den Anforderungen der Ziele zu entsprechen. 

## Metriken mit Spring Boot konfigurieren
{: #spring-metrics-configuration}

In den folgenden Abschnitten wird beschrieben, wie Metriken für Spring Boot Actuator mit Micrometer aktiviert werden. Sie erfahren, wie Sie (in beiden Releases) beginnend mit dem aktuellen Release 2 von Spring Boot Metriken erfassen und einen Endpunkt für Prometheus bereitstellen. 

### Metriken in Spring Boot 2 konfigurieren
{: #spring-metrics-boot2}

Der Spring Boot 2 Actuator-Metrikendpunkt muss aktiviert und für den Webzugriff verfügbar gemacht werden. Standardmäßig ist der Endpunkt aktiviert, aber nicht verfügbar (siehe die [Dokumentation zu Spring-Endpunkten](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-endpoints.html){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link")).  

Fügen Sie folgenden Code zur Datei `application.properties` hinzu, um den Endpunkt für Metriken bereitzustellen:

```properties
management.endpoints.web.exposure.include=health,metrics
```
{: codeblock}

Überprüfen Sie den Endpunkt `/actuator`, um sicherzustellen, dass der Metrikendpunkt aktiviert ist. Die Ausgabe sieht in etwa wie folgt aus, wenn Sie die App lokal ausführen und `jq` verwenden, um die Lesbarkeit der JSON-Antwort zu verbessern: 

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

Greifen Sie auf den Endpunkt `/actuator/metrics` zu, um eine Liste der verfügbaren Metriken im JSON-Format anzuzeigen, die in etwa folgendermaßen aussieht: 
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

Eine zusätzliche Anforderung ist erforderlich, um den tatsächlichen Wert einer einzelnen Metrik anzuzeigen (diese kann aus der Liste der Actuator-URLs abgeleitet werden). Führen Sie beispielsweise den folgenden Befehl aus, um die JVM-Theadstatusangaben abzurufen: 

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

Weitere Informationen darüber, wie Sie das Verhalten des Metrik-Actuators anpassen können, finden Sie in der [offiziellen Dokumentation](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-metrics.html){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link").

### Prometheus-Unterstützung in Spring Boot 2 aktivieren
{: #spring-prometheus-boot2}

Prometheus erfordert, dass die App einen Endpunkt hostet, von dem der Prometheus-Server Daten liest, um die Metriken abzurufen. Das Hosting dieses Endpunkts in Spring setzt voraus, dass die App die Daten bereitstellt. Dies bedeutet, dass der Prometheus-Endpunkt nur Spring Boot-Apps unterstützt, die Spring MVC, Spring WebFlux oder Jersey verwenden. 

Um den Prometheus-Endpunkt zu aktivieren, fügen Sie die folgende Abhängigkeit in der Datei `pom.xml` hinzu:
```xml
<dependency>
  <groupId>io.micrometer</groupId>
  <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```
{: codeblock}

Spring erkennt die Abhängigkeit und konfiguriert `PrometheusMeterRegistry` automatisch als eine der konfigurierten Registrys in der globalen `MeterRegistry`-Bean. Wie der Metrikendpunkt muss der Prometheus-Endpunkt jedoch aktiviert und für den Webzugriff zugänglich gemacht werden, indem der Datei `application.properties` Folgendes hinzugefügt wird: 

```properties
management.endpoints.web.exposure.include=health,metrics,prometheus
```
{: codeblock}

Überprüfen Sie den Endpunkt `/actuator`, um sicherzustellen, dass der Prometheus-Endpunkt aktiviert wurde. Die Ausgabe sieht in etwa wie folgt aus, wenn Sie die App lokal ausführen: 

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

Der Endpunkt `/actuator/prometheus` gibt alle konfigurierten Metriken in Prometheus-Scrape-Format aus: 

```
$ curl -s localhost:8080/actuator/prometheus
# HELP system_cpu_usage Die "letzte CPU-Belastung" für das gesamte System
# TYPE system_cpu_usage gauge
system_cpu_usage 0.046231032221805926
...
```
{: screen}

Fügen Sie der Servicedefinition die folgenden Annotationen hinzu, um das Scraping in Prometheus für den Metrikendpunkt zu aktivieren:

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

### Metriken in Spring Boot 1 konfigurieren
{: #spring-metrics-boot1}

Der `/metrics`-Endpunkt von Spring Boot Actuator ist standardmäßig in Spring Boot 1 aktiviert, wird jedoch als "sensibel" betrachtet und muss autorisiert werden. In einigen Umgebungen kann die Sicherung des Metrikendpunkts die richtige Antwort sein. Das Autorisieren jeder Anforderung an den Metrikendpunkt führt jedoch zu zu viel Latenzzeit für die automatisierte Überprüfung mit Kubernetes.

Inaktivieren Sie das Attribut 'sensitive' des Metrikendpunkts, um die Berechtigungsanforderung zu entfernen, indem Sie der Datei `application.properties` Folgendes hinzufügen: 
```properties
endpoints.metrics.sensitive=false
```
{: codeblock}

Nun ist die standardmäßige Metrikunterstützung in Spring Boot 1 aktiviert. Sie erzeugt eine Ausgabe ähnlich der folgenden: 
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

Weitere Informationen darüber, wie Sie das Verhalten des Actuators für Spring Boot 1-Metriken anpassen können, finden Sie in der [offiziellen Dokumentation](https://docs.spring.io/spring-boot/docs/1.5.x/reference/html/production-ready-metrics.html){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link").

### Prometheus-Unterstützung in Spring Boot 1 aktivieren
{: #spring-prometheus-boot1}

Die Prometheus-Unterstützung ist zwar nicht in die Metriken für Spring Boot 1 Actuator integriert; Sie können jedoch eine entsprechende Unterstützung über Micrometer hinzufügen. 

Wie in der [Übersicht](#spring-metrics) erwähnt, wurde Micrometer auf Spring Boot 1 zurückportiert, da es über umfangreichere Metrikprimmitive verfügt und eine bessere Unterstützung für Überwachungssysteme wie Prometheus und Statsd bietet. Die Verwendung von Micrometer mit Spring Boot 1 ist relativ einfach: Erforderlich sind eine bestimmte Adapterbibliothek und mindestens eine Registry. Fügen Sie Folgendes zur Datei `pom.xml` hinzu, um Abhängigkeiten für die Adapterbibliothek und die Prometheus-Registry einzuschließen:

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

Durch diese Abhängigkeiten wird ein Metrikendpunkt `/prometheus` erstellt und zugänglich gemacht. Wie der Standardendpunkt `/metrics` wird auch der Endpunkt `/prometheus` standardmäßig als sensibel betrachtet. Um die Berechtigungsanforderung zu entfernen, fügen Sie der Datei `application.properties` Folgendes hinzu: 

```properties
endpoints.prometheus.sensitive=false
```
{: codeblock}

Die vom Endpunkt `/prometheus` erzeugte Ausgabe ähnelt der Ausgabe für Spring Boot 2: 

```
$ curl -s localhost:8080/prometheus
# HELP jvm_memory_used_bytes Der Umfang des belegten Speichers
# TYPE jvm_memory_used_bytes gauge
jvm_memory_used_bytes{area="heap",id="PS Survivor Space",} 2.2594152E7
...
```
{: screen}

Fügen Sie der Servicedefinition die folgenden Annotationen hinzu, um das Scraping in Prometheus für den Metrikendpunkt zu aktivieren:

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

Weitere Informationen zur Verwendung von Micrometer mit Spring Boot 1 finden Sie unter [https://micrometer.io/docs/ref/spring/1.5](https://micrometer.io/docs/ref/spring/1.5){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link").

## Ablaufsteuerung für Aufrufe mit Micrometer
{: #spring-metrics-timing}

Sowohl Spring MVC als auch WebFlux und Jersey Server bieten eine automatische Konfiguration für Spring, die automatisch Ablaufsteuerungsinformationen für alle Anforderungen erfasst, wenn die Eigenschaft `management.metrics.web.server.auto-time-requests` auf `true` gesetzt ist. Die Standardeinstellung ist `true`. 

```properties
management.metrics.web.server.auto-time-requests=true
```
{: codeblock}

Micrometer unterstützt auch die Annotation `@Timed`, die zum Erfassen von Ablaufsteuerungsinformationen für unterstützte Methodenaufrufe verwendet werden kann. Es sollte darauf hingewiesen werden, dass diese Annotation nicht für beliebige Methoden verwendet werden kann, da sie die Containerunterstützung benötigt, um die Daten zu erfassen. `@Timed`-Annotationen werden für Operationen unterstützt, die von einem Servlet-Container unterstützt werden, z. B. Spring-MVC-, Webflux- oder Jersey-Servicemethoden.

Wenn Sie genauer definieren möchten, welche Ablaufsteuerungsinformationen erfasst werden, setzen Sie die Eigenschaft `management.metrics.web.server.auto-time-requests` auf `false` und verwenden Sie die Annotation `@Timed` für einzelne Servicemethoden oder Controller:

```java
@Timed
@GetMapping("/myendpoint")
public String getSomething() { ... }
```
{: codeblock}

Wenn `management.metrics.web.server.auto-time-requests` den Wert `true` hat, kann die Annotation `@Timed` verwendet werden, um zusätzliche Tags zu einer angegebenen Metrik hinzuzufügen. Beispiel:

```java
@Timed(extraTags = { "mytagname", "mytagvalue"})
@GetMapping("/myendpoint")
public String getSomething() { ... }
```
{: codeblock}

## Angepasste Metriken bei Spring und Micrometer
{: #spring-metrics-custom}

Um anwendungsdomänenspezifische Messungen vornehmen zu können, müssen Sie eigene angepasste Metriken erstellen. Micrometer definiert hierbei einige grundlegende Zählertypen``:

* Ein Zähler (`Counter`) verfolgt einen Wert, der sich erhöhen kann. 
* Eine Messanzeige (`Gauge`) misst den beobachteten Wert und gibt ihn zurück, wenn die Messung veröffentlicht (oder abgefragt) wird.
* Ein Zeitgeber (`Timer`) verfolgt die Häufigkeit eines eingetretenen Ereignisses und die kumulative abgelaufene Zeit für dieses Ereignis.  
* Eine Verteilungszusammenfassung (`DistributionSummary`) ähnelt einem `Timer`; die Verteilung von Ereignissen wird auch überwacht, es erfolgt jedoch keine Zeitmessung.

Neue Zähler werden mithilfe von Helper-Methoden in der `MeterRegistry` oder mithilfe des Fluent-Builders des Zählers erstellt. 

Erstellen Sie Ihr Messinstrument (`Meter`) im Konstruktor (geben Sie dabei die `MeterRegistry` als Parameter an) oder in einer `@PostConstruct`-Methode, nachdem die `MeterRegistry` eingefügt wurde. 

Um beispielsweise einen `Counter` in einem `RestController` zu verwenden, könnten Sie beispielsweise folgendermaßen vorgehen:

```java
@RestController
public class MyController {
    
    private final Counter myCounter;
    
    public MyController(MeterRegistry meterRegistry) {
    
        // Counter mithilfe der Helper-Methode im Builder erstellen
        myCounter = meterRegistry.counter("my.counter", "mytagname", "mytagvalue");
        
    }
```
{: codeblock}

Innerhalb der Servicemethoden können Sie dann den Zählerwert entsprechend aktualisieren:

```java
@GetMapping("/")
public String getSomething() {
    myCounter.increment();
    ...
}
```
{: codeblock}


Weitere Informationen zum Erstellen von angepassten Metriken in Spring finden Sie hier:

* [Spring Boot-Metriken](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-metrics.html){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link")
* [Micrometer-Konzepte](https://micrometer.io/docs/concepts){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link")
