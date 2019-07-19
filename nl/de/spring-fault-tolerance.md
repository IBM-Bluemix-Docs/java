---

copyright:
  years: 2018, 2019
lastupdated: "2019-04-22"

keywords: fault tolerance spring, hystrix spring, netflix spring, hystrixcommand spring, bulkhead spring, circuit breaker spring

subcollection: java

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}

# Fehlertoleranz mit Spring
{: #spring-tolerance}

Das Erstellen eines ausfallsicheren Systems stellt Anforderungen an alle Services innerhalb des Systems. Der dynamische Charakter von Cloudumgebungen setzt voraus, dass Services angemessen auf nicht erwartete Ereignisse reagieren können.

Spring verwendet die Fehlertoleranzbibliothek [Netflix Hystrix](https://github.com/Netflix/Hystrix/wiki){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link"), um Fehlertoleranzfehler auf Anwendungsebene zu unterstützen. Mit Hystrix können Sie Fallbacks, Sicherungen, Bulkheads und zugehörige Metriken erstellen.

Diese Informationen bauen auf Fehlertoleranzverfahren auf, die unter [Cloudnative Entwicklung: Fehlertoleranz](/docs/java?topic=cloud-native-fault-tolerance#fault-tolerance) beschrieben sind.
{: note}

## Hystrix mit einer Spring-Anwendung verwenden
{: #spring-hystrix-starter}

Damit Hystrix in einer Spring-Anwendung einfacher verwendet werden kann, gibt es einen Spring Boot-Starter, der einen annotierten Ansatz zur Integration von Hystrix mit Ihrer Anwendung bietet.

Fügen Sie die folgende Abhängigkeit hinzu, um Hystrix hinzuzufügen:

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```
{: codeblock}

Damit Spring diese neue Funktion verwendet, fügen Sie Ihrer Hauptanwendungsklasse eine Annotation hinzu und aktivieren Sie die Verarbeitung der Hystrix-Annotationen durch Spring für die Sicherung, indem Sie `@EnableCircuitBreaker` hinzufügen.

```java
@SpringBootApplication
@EnableCircuitBreaker
public class MyApplication {

	public static void main(String[] args) {
		SpringApplication.run(MyApplication.class, args);
	}
```
{: codeblock}

### HystrixCommand zum Definieren eines Fallbacks verwenden
{: #spring-fallback}

Mit der Annotation `@HystrixCommand` können Sie eine Methode in ein Sicherungsverhalten einschließen. Spring erkennt Methoden in einer Anwendung, die mit `@EnableCircuitBreaker` annotiert ist, und schließt sie mit Hystrix-Unterstützung ein, um Fehler und Zeitlimitüberschreitungen zu überwachen. Anschließend ruft Spring das geeignete alternative Verhalten bei Bedarf auf.

Die Annotation `@HystrixCommand` wird nur für Methoden in einer Annotation `@Component` oder `@Service` unterstützt.
{: note}

Die folgende Annotation `@HystrixCommand` schließt den Aufruf `service()` in eine Sicherung ein. Der Proxy ruft die Methode `fallback()` auf, wenn die Methode `service()` fehlschlägt oder wenn die Verbindung geöffnet ist.

```java
@Autowired
private MicroService myService;

@GetMapping("/endpoint")
public String value() throws Exception {
    return "Service returned: " + myService.service();
}

@Service
class MicroService {
    @HystrixCommand(fallbackMethod = "fallback")
    public String service() {
        // Verarbeitung läuft. Wenn eine Ausnahme ausgelöst wird,
        // wird die Fallback-Methode aufgerufen.
        return "Success";
    }

    public String fallback(Throwable e) {
        return "Fallback! Reason: " + e.getMessage() + "\n";
    }
}
```
{: codeblock}

Weitere Informationen erhalten Sie, wenn Sie sich mit dem Spring-basierten Beispiel zur [Hystrix-Sicherung ](https://spring.io/guides/gs/circuit-breaker/){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") eingehender auseinandersetzen.
{: tip}

### Zeitlimits verwenden
{: #spring-timeout}

Wie reagiert Ihre Anwendung auf einen fernen Service, der nicht reagiert? Der Wartestatus kann eine Weile oder ewig lange dauern. Hystrix gibt Ihnen die Möglichkeit, die Wartezeit zu definieren, die für Ihre Anwendung akzeptabel ist.

Mit einer einfachen Ergänzung zur Annotation `HystrixCommand` können Sie den Standardwert von 1 bis 30 Sekunden für das Zeitlimit ändern:

```java
@HystrixCommand(
    fallbackMethod = "fallback",
    commandProperties = {
        @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "30000"),
    }
)
```
{: codeblock}

### Bulkheads verwenden
{: #spring-bulkhead}

Hystrix unterstützt sowohl Semaphor- als auch warteschlangenbasierte Bulkheads. Das folgende Snippet zeigt, wie ein warteschlangenbasierter Bulkhead konfiguriert wird, der vier Threads zuordnet und die Anzahl der ausstehenden Anforderungen auf 10 begrenzt:

```java
@HystrixCommand(
    fallbackMethod = "fallback",
    threadPoolProperties = {
        @HystrixProperty(name = "coreSize", value = "4"),
        @HystrixProperty(name = "maxQueueSize", value = "10")
    }
)
```
{: codeblock}

### Sicherungsstatus
{: #spring-breaker-status}

Der Hystrix Spring-Starter weist eine zusätzliche Besonderheit auf: Er erweitert den Standardendpunkt `/health` für die Anwendung, der über einen Spring-Actuator bereitgestellt wird. Weitere Informationen finden Sie unter [Metrinken mit Spring](/docs/java?topic=java-spring-metrics#spring-metrics)).

Wenn der Statusendpunkt [so konfiguriert ist, dass er zusätzliche Details enthält](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-endpoints.html#production-ready-health){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link"), wird der Sicherungsstatus in die Informationen zur Statusprüfung eingeschlossen. (Dieses Verhalten ist standardmäßig inaktiviert.)

```
{
    "hystrix": {
        "openCircuitBreakers": [
            "MicroService::service"
        ],
        "status": "CIRCUIT_OPEN"
    },
    "status": "UP"
}
```
{: screen}

## Nächste Schritte
{: #spring-tolerance-next-steps notoc}

Weitere Informationen zu Hystrix-Konfigurationen finden Sie im [Wiki zur Hystrix-Konfiguration](https://github.com/Netflix/Hystrix/wiki/Configuration){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link").

Weitere Informationen zu Hystrix mit Spring finden Sie hier:

* [Leitfaden für Spring Cloud Netflix - Hystrix](https://www.baeldung.com/spring-cloud-netflix-hystrix){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link")
* [Leitfaden für Spring-Sicherungen](https://spring.io/guides/gs/circuit-breaker/){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link")
