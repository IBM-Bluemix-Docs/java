---

copyright:
  years: 2018, 2019
lastupdated: "2019-06-10"

keywords: health check spring, spring health endpoint, spring-boot-actuator, liveness probe spring, readiness probe spring, spring kubernetes probe

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

# Controlli di integrità con Spring
{: #spring-healthcheck}

L'endpoint di integrità fornito da Spring Boot Actuator è legato al ciclo di vita dell'applicazione e non è raggiungibile finché l'applicazione non viene avviata. Altrimenti viene disabilitato appena l'applicazione viene arrestata (cosa che avviene prima dell'arresto del processo). Spring Boot Actuator aggiunge automaticamente gli indicatori di integrità per le tecnologie rilevate sul percorso classi. Ad esempio, se l'applicazione utilizza JDBC, l'endpoint di integrità include dei test di base per garantire che sia possibile raggiungere l'archivio dati di supporto. L'endpoint di integrità definito dall'Actuator è particolarmente indicato per l'utilizzo come un probe di disponibilità per tale motivo.

Abilita Spring Boot Actuator aggiungendo la dipendenza `spring-boot-actuator` al tuo file `pom.xml`.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-actuator</artifactId>
</dependency>
```
{: codeblock}

Ci sono alcune differenze nella modalità di funzionamento con Spring Boot Actuator tra le versioni. Le prossime due sezioni esplorano come creare controlli di attività e disponibilità in entrambe le versioni di Spring Boot, iniziando con la 2 come la più corrente.

## Controlli di integrità di Spring Boot 2
{: #spring-health-boot2}

L'Actuator di Spring Boot 2 definisce un endpoint `/actuator/health`, che restituisce un payload `{"status": "UP"}` quando va tutto bene. Questo endpoint è abilitato per impostazione predefinita e non richiede alcun codice applicativo.

Vedi il seguente esempio di un controllo di integrità riuscito e non autenticato utilizzando l'Actuator di Spring Boot 2:
<!-- Spring Boot 2 test project: https://github.com/IBM/spring-alarm-application -->

```
$ curl -i localhost:8080/actuator/health
HTTP/1.1 200
Content-Type: application/vnd.spring-boot.actuator.v2+json;charset=UTF-8
Transfer-Encoding: chunked
Date: Fri, 07 Dec 2018 23:09:09 GMT

{"status":"UP"}
```
{: screen}

Come mostrato, l'endpoint di integrità restituisce un semplice stato UP o DOWN per impostazione predefinita. Per un controllo dell'integrità automatizzato, questo semplice payload è di solito sufficiente. Delle informazioni sull'integrità dettagliate possono essere abilitate impostando la seguente proprietà in `application.properties`:

```properties
management.endpoint.health.show-details=always
```
{: codeblock}

```
$ curl -i localhost:8080/actuator/health
HTTP/1.1 200
Content-Type: application/vnd.spring-boot.actuator.v2+json;charset=UTF-8
Transfer-Encoding: chunked
Date: Fri, 07 Dec 2018 23:09:09 GMT

{"status":"UP","details":{"diskSpace":{"status":"UP","details":{"total":500068036608,"free":407611420672,"threshold":10485760}},"db":{"status":"UP","details":{"database":"H2","hello":1}}}}
```
{: screen}

Nota l'inclusione di alcune informazioni del database H2. Questo esempio di Spring Actuator aggiunge automaticamente dei controlli per i servizi di supporto. In questo caso, l'applicazione sta utilizzando JDBC e il driver H2 è stato rilevato nel percorso classi.

Puoi sovrascrivere la modalità di funzionamento predefinita dell'endpoint di integrità con le proprietà o il codice, come descritto nel manuale [Spring Boot Reference Guide for 2.1.x](https://docs.spring.io/spring-boot/docs/2.1.x/reference/html/production-ready-endpoints.html#production-ready-health){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno").

### Probe di attività in Spring Boot 2
{: #spring-liveness-boot2}

Il framework Actuator in Spring Boot 2 è il suo stesso mini-framework REST che può essere esteso con degli endpoint personalizzati. Questo significa che puoi aggiungere un semplice endpoint di attività gestito nello stesso modo in cui viene gestito l'indicatore di integrità incorporato. Un endpoint di attività personalizzato può essere banalmente dichiarato come nel seguente esempio:

```java
@Endpoint(id="liveness")
@Component
public class Liveness {
    @ReadOperation
    public String testLiveness() {
            return "{\"status\":\"UP\"}";
    }
}
```
{: codeblock}

Questo endpoint personalizzato devo essere esposto come un endpoint web:

```properties
management.endpoints.web.exposure.include=health,liveness,...
```
{: codeblock}

```
$ curl -i localhost:8080/actuator/liveness
HTTP/1.1 200
Content-Type: application/vnd.spring-boot.actuator.v2+json;charset=UTF-8
Content-Length: 15
Date: Thu, 07 Feb 2019 22:38:27 GMT

{"status":"UP"}
```
{: screen}

### Configurazione di probe di disponibilità e di attività Spring Boot 2 in Kubernetes
{: #spring-probe-config-2}

Aggiungi la configurazione di probe di disponibilità e attività alla definizione del contenitore in un manifest di distribuzione Kubernetes (nota i valori di percorso e porta).

```yaml
spec:
  containers:
  - name: ...
    image: ...
    readinessProbe:
      httpGet:
        path: /actuator/health
        port: 8080
      initialDelaySeconds: 60
      timeoutSeconds: 5
    livenessProbe:
      httpGet:
        path: /actuator/liveness
        port: 8080
      initialDelaySeconds: 130
      timeoutSeconds: 10
      failureThreshold: 10
```
{: codeblock}

Per evitare i cicli di riavvio, imposta `livenessProbe.initialDelaySeconds` in modo da essere tranquillamente più lungo del tempo necessario al tuo servizio per l'inizializzazione. Puoi poi utilizzare un valore più corto per `readinessProbe.initialDelaySeconds` per instradare le richieste al servizio come è pronto.

## Controlli di integrità in Spring Boot 1
{: #spring-health-boot1}

L'Actuator di Spring Boot 1 definisce un endpoint `/health`, che restituisce un payload `{"status": "UP"}` quando va tutto bene. L'endpoint non richiede l'autorizzazione, ma se viene configurata e il chiamante è autorizzato, vengono incluse nella risposta ulteriori informazioni.

Vedi il seguente esempio di un controllo di integrità riuscito e non autenticato utilizzando l'Actuator di Spring Boot 1:
```
$ curl -i localhost:8080/health
HTTP/1.1 200
X-Application-Context: application
Content-Type: application/vnd.spring-boot.actuator.v1+json;charset=UTF-8
Content-Length: 15

{"status":"UP"}
```
{: screen}

Un esempio di un controllo di integrità riuscito autenticato:

```
$ curl -i localhost:8080/health
HTTP/1.1 200 OK
X-Application-Context: application
Content-Type: application/vnd.spring-boot.actuator.v1+json;charset=UTF-8
Content-Length: 221

{
    "status" : "UP",
  "diskSpace" : {
        "status" : "UP",
    "total" : 63251804160,
    "free" : 31316164608,
    "threshold" : 10485760
    },
  "db" : {
        "status" : "UP",
    "database" : "H2",
    "hello" : 1
    }
}
```
{: screen}

Puoi sovrascrivere la modalità di funzionamento predefinita dell'endpoint di integrità con le proprietà o il codice, come descritto nel manuale [Spring Boot Reference v1.5.x](https://docs.spring.io/spring-boot/docs/1.5.9.RELEASE/reference/htmlsingle/#_auto_configured_healthindicators){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno").

### Probe di attività in Spring Boot 1
{: #spring-liveness-boot1}

Per Spring Boot 1, un semplice endpoint HTTP per l'attività che restituisce sempre {"status": "UP"} con il codice di stato 200 avrà un aspetto simile al seguente frammento di codice:

```java
@RestController
public class Liveness {
    private static Map<String, String> alwaysOk = Collections.singletonMap("status", "OK");

    @GetMapping("/liveness")
    public Map<String,String> testLiveness() {
        return alwaysOk;
    }
}
```
{: codeblock}

```
$ curl -i localhost:8080/liveness
HTTP/1.1 200
Content-Type: application/json;charset=UTF-8
Transfer-Encoding: chunked
Date: Thu, 07 Feb 2019 22:43:32 GMT

{"status":"OK"}
```
{: screen}

### Configurazione di probe di disponibilità e di attività Spring Boot 1 in Kubernetes
{: #spring-probe-config-1}

Aggiungi la configurazione di probe di disponibilità e attività alla definizione del contenitore in un manifest di distribuzione Kubernetes (nota i valori di percorso e porta).

```yaml
spec:
  containers:
  - name: ...
    image: ...
    readinessProbe:
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 60
      timeoutSeconds: 5
    livenessProbe:
      httpGet:
        path: /liveness
        port: 8080
      initialDelaySeconds: 130
      timeoutSeconds: 10
      failureThreshold: 10
```
{: codeblock}

Per evitare i cicli di riavvio, imposta `livenessProbe.initialDelaySeconds` in modo da essere tranquillamente più lungo del tempo necessario al tuo servizio per l'inizializzazione. Puoi poi utilizzare un valore più corto per `readinessProbe.initialDelaySeconds` per instradare le richieste al servizio come è pronto.
