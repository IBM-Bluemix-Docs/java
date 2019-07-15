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

# Tolleranza di errore con Spring
{: #spring-tolerance}

La creazione di un sistema resiliente imposta i requisiti su tutti i servizi all'interno di esso. La natura dinamica degli ambienti cloud richiede che i servizi siano progettati per rispondere correttamente a un imprevisto.

Spring utilizza la libreria di tolleranza di errore [Netflix Hystrix](https://github.com/Netflix/Hystrix/wiki){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno") per supportare i problemi di tolleranza di errore a livello di applicazione. Con Hystrix puoi creare fallback, interruttori, bulkhead e metriche associate.

Queste informazioni sono sviluppate sulle prassi di tolleranza di errore descritte nella sezione [Cloud-native development: Fault tolerance](/docs/java?topic=cloud-native-fault-tolerance#fault-tolerance).
{: note}

## Utilizzo di Hystrix con un'applicazione Spring
{: #spring-hystrix-starter}

Per rendere Hystrix più facile da utilizzare con un'applicazione Spring, è disponibile uno Spring Boot Starter che abilita un approccio che prevede l'uso di annotazioni all'integrazione di Hystrix nella tua applicazione.

Per aggiungere Hystrix, aggiungi la seguente dipendenza:

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```
{: codeblock}

Per abilitare Spring ad utilizzare questa nuova funzione, aggiungi un'annotazione alla tua classe dell'applicazione principale e abilita l'elaborazione di Spring delle annotazioni Hystrix per gli interruttori aggiungendo `@EnableCircuitBreaker`.

```java
@SpringBootApplication
@EnableCircuitBreaker
public class MyApplication {

	public static void main(String[] args) {
		SpringApplication.run(MyApplication.class, args);
	}
```
{: codeblock}

### Utilizzo di HystrixCommand per definire un fallback
{: #spring-fallback}

Per aggiungere una modalità di funzionamento con interruttori a un metodo, esegui l'annotazione con `@HystrixCommand`. Spring rileva i metodi all'interno di un'applicazione annotata con `@EnableCircuitBreaker` e ne esegue il wrapping con il supporto Hystrix per monitorare gli errori e i timeout. Successivamente, Spring richiama il comportamento alternativo appropriato quando necessario.

L'annotazione `@HystrixCommand` è supportata solo sui metodi in un `@Component` o `@Service`.
{: note}

La seguente annotazione `@HystrixCommand` esegue il wrapping del richiamo `service()` per fornire la modalità di funzionamento con interruttori. Il proxy richiama il metodo `fallback()` se il metodo `service()` non riesce o se il circuito è aperto.

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
        // Do some processing. If an exception is thrown
        // the fallback method will be called
        return "Success";
    }

    public String fallback(Throwable e) {
        return "Fallback! Reason: " + e.getMessage() + "\n";
    }
}
```
{: codeblock}

È possibile trovare ulteriori informazioni controllando l'esempio [Hystrix Circuit Breaker](https://spring.io/guides/gs/circuit-breaker/){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno") basato su Spring.
{: tip}

### Utilizzo dei timeout
{: #spring-timeout}

Come la tua applicazione risponde a un servizio remoto che non risponde? L'attesa può essere breve o durare per sempre. Hystrix ti consente di definire qual è la quantità di tempo accettabile per la tua applicazione.

È possibile utilizzare una semplice aggiunta all'annotazione `HystrixCommand` per modificare il valore di timeout dal valore predefinito di 1 secondo con 30 secondi:

```java
@HystrixCommand(
    fallbackMethod = "fallback",
    commandProperties = {
        @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "30000"),
    }
)
```
{: codeblock}

### Utilizzo dei bulkhead
{: #spring-bulkhead}

Hystrix supporta bulkhead di tipo semaforo o basati sulle code. Il seguente frammento di codice mostra come configurare un bulkhead basato sulle code che assegna quattro thread e limita il numero di richieste in sospeso a 10:

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

### Stato dell'interruttore
{: #spring-breaker-status}

Lo starter Hystrix Spring può inoltre potenziare l'endpoint `/health` predefinito per l'applicazione, fornito mediante uno Spring Actuator. Per ulteriori informazioni, vedi [Metriche con Spring](/docs/java?topic=java-spring-metrics#spring-metrics)).

Se l'endpoint di integrità è [configurato per includere dei dettagli supplementari](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-endpoints.html#production-ready-health){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno"), lo stato dell'interruttore viene incluso con le informazioni del controllo di integrità. (Questa modalità di funzionamento è disabilitata per impostazione predefinita).

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

## Passi successivi
{: #spring-tolerance-next-steps notoc}

Per ulteriori informazioni sulla configurazione di Hystrix, vedi il [wiki relativo alla configurazione di Hystrix](https://github.com/Netflix/Hystrix/wiki/Configuration){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno").

Per ulteriori informazioni su Hystrix con Spring, consulta:

* [A Guide to Spring Cloud Netflix](https://www.baeldung.com/spring-cloud-netflix-hystrix){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno")
* [Spring Circuit Breaker Guide](https://spring.io/guides/gs/circuit-breaker/){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno")
