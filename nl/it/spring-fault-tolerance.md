---

copyright:
  years: 2018, 2019
lastupdated: "2019-03-15"

keywords: fault tolerance spring, hystrix spring, netflix spring, hystrixcommand spring, bulkhead spring, circuit breaker spring

subcollection: java

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}

# Tolleranza di errore con Spring
{: #spring-tolerance}

La creazione di un sistema resiliente imposta i requisiti su tutti i servizi all'interno di esso. La natura dinamica degli ambienti cloud richiede che i servizi siano progettati per essere pronti per gli imprevisti e per rispondere ad essi in modo corretto.

Spring utilizza la libreria di tolleranza di errore [Netflix Hystrix](https://github.com/Netflix/Hystrix/wiki){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno") per supportare i problemi di tolleranza di errore a livello di applicazione. Hystrix fornisce il supporto per fallback, interruttori, bulkhead e metriche associate. 

Queste informazioni sono sviluppate sulle prassi di tolleranza di errore descritte nella sezione [Cloud-native development: Fault tolerance](/docs/java?topic=cloud-native-fault-tolerance#fault-tolerance).
{: note}

## Utilizzo di Hystrix con un'applicazione Spring
{: #spring-hystrix-starter}

Per rendere Hystrix più facile da utilizzare con un'applicazione Spring, è disponibile uno Spring Boot Starter che abilita un approccio che prevede l'uso di annotazioni all'integrazione di Hystrix nella tua applicazione.

L'aggiunta di questa singola dipendenza alla tua applicazione aggiungerà Hystrix. 

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```
{: codeblock}

Analogamente a molti altri starter Spring, per indicare a Spring che vuoi utilizzare la funzionalità nell'applicazione, aggiungi un'annotazione alla classe dell'applicazione principale. Per abilitare l'elaborazione di Spring delle annotazioni Hystrix per gli interruttori, aggiungi `@EnableCircuitBreaker`.

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

Per aggiungere una modalità di funzionamento con interruttori a un metodo, ne esegui l'annotazione con `@HystrixCommand`. Spring troverà questi metodi all'interno di un'applicazione annotata con `@EnableCircuitBreaker` e ne eseguirà il wrapping con la funzionalità Hystrix per eseguire il monitoraggio per rilevare errori/timeout e richiamerà la modalità di funzionamento alternativa appropriata quando necessario 

`@HystrixCommand`è supportato solo sui metodi in un `@Component` o `@Service` {: note}

La seguente annotazione `@HystrixCommand` esegue il wrapping del richiamo `service()` per fornire la modalità di funzionamento con interruttori. Il proxy richiamerà il metodo `fallback()` se il metodo `service()` non riesce o se il circuito è aperto.

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

Quando si richiama un servizio remoto, potrebbe non rispondere immediatamente; ci potrebbe volere un po' di tempo o un tempo indefinitamente lungo. Hystrix ti consente di definire cosa è accettabile per la tua applicazione. È possibile utilizzare una semplice aggiunta all'annotazione `HystrixCommand` per modificare il valore predefinito del valore di timeout, che è pari a 1 secondo.

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

Hystrix supporta bulkhead di tipo semaforo o basati sulle code. Il seguente frammento di codice mostra come configurare un bulkhead basato sulle code che assegna 4 thread e limita il numero di richieste in sospeso a 10:

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

Lo starter Hystrix Spring può inoltre potenziare l'endpoint `/health` predefinito per l'applicazione (fornito mediante uno Spring Actuator; per ulteriori informazioni, consulta l'[argomento relativo alle metriche](/docs/java?topic=java-spring-metrics#spring-metrics)).

Se l'endpoint di integrità è [configurato per includere dei dettagli supplementari](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-endpoints.html#production-ready-health){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno"), lo stato dell'interruttore verrà incluso con le informazioni del controllo di integrità. (Questa modalità di funzionamento è disabilitata per impostazione predefinita).

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
