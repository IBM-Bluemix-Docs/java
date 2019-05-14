---

copyright:
  years: 2019
lastupdated: "2019-04-23"

keywords: health check jax-rs, jax-rs endpoint, jax-rs status, readiness jax-rs, liveness jax-rs, microprofile health

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

# Controlli di integrità con JAX-RS
{: #jaxrs-healthcheck}

Come discusso nella sezione precedente, Kubernetes fornisce diversi metodi per integrare i probe di integrità, compresa l'esecuzione di un comando, e il controllo di rete tramite gli endpoint TCP o HTTP. Anche se è possibile implementare uno qualsiasi di questi probe nel linguaggio Java&trade, un'implementazione JAX-RS di probe HTTP e MicroProfile Health viene qui discussa in modo dettagliato.

## Definizione di un endpoint JAX-RS di disponibilità
{: #mp-readiness}

Un endpoint di disponibilità minimo è qualcosa di simile al seguente esempio:

```java
@Path("ready")
public class ReadinessEndpoint {
  @GET
  @Produces(MediaType.APPLICATION_JSON)
  public Response testReadiness() {
    if ( ! ready ) {
      return Response.status(503).entity("{\"status\":\"DOWN\"}").build();
    }
    return Response.ok("{\"status\":\"UP\"}").build();
  }
}
```
{: codeblock}

Un endpoint come questo in un'applicazione distribuita in WebSphere Liberty raggiunge un certo livello di modalità di funzionamento di disponibilità senza ulteriori sforzi. Liberty indica come non riuscite le richieste all'endpoint di integrità con una risposta 503 appropriata finché l'applicazione è in esecuzione. Questa implementazione di base deve essere ampliata per includere le funzionalità richieste. Prendi ad esempio in considerazione la valutazione delle risorse CDI `@ApplicationScoped` per garantire che l'elaborazione di inserimento delle dipendenze sia stata completata. Dopo essere stato implementato, il probe può essere abilitato configurando il tipo di probe HTTP, come indicato nella sezione precedente.

Ricordati sempre il ruolo e lo scopo della tolleranza di errore: ci si aspetta che i microservizi gestiscano gli errori e le interruzioni nelle comunicazioni con i servizi di downstream. Includi i controlli solo per le funzionalità che non hanno alcun fallback fattibile all'interno di un controllo di disponibilità.

## Definizione di un endpoint JAX-RS di attività
{: #jaxrs-liveness}

Un probe di attività deve essere cauto in merito a cosa controlla, perché un errore causa una terminazione immediata del processo. Un semplice endpoint di attività può essere simile al seguente:

```java
@Path("liveness")
public class LivenessEndpoint {
    @GET
    @Produces(MediaType.APPLICATION_JSON)
    public Response testLiveness() {
      return Response.ok("{\"status\":\"UP\"}").build();
    }
}
```
{: codeblock}

L'implementazione del probe di attività può essere estesa per consultare le condizioni locali del processo, che indicano uno stato del processo irreversibile. Tuttavia la sfida spesso rappresentata da questo tipo di controlli consiste nel fatto che, se è possibile eseguire una tale elaborazione, il server è probabilmente sufficientemente integro per eseguire altre richieste. Per tale motivo, all'implementazione del controllo di attività dovrebbe essere prontamente applicato un principio di semplicità. Dopo che le origini di attività sono state codificate per abilitare l'endpoint di probe, l'endpoint di attività http può essere abilitato nelle origini di orchestrazione del contenitore, come spiegato nell'ultimo capitolo.

Per evitare cicli di riavvio, l'attributo initialDelaySeconds per il controllo di attività deve essere più lungo del tempo di avvio del server massimo previsto. Per un server delle applicazioni Java&trade che di norma impiega 30 secondi per l'avvio, scegli un valore maggiore, come ad esempio 60 secondi.

## Utilizzo dell'API MicroProfile Health
{: #mp-health-api}

L'[API MicroProfile Health](https://www.ibm.com/support/knowledgecenter/en/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/twlp_microprofile_healthcheck.html){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno") (mpHealth) definisce un framework per semplificare l'implementazione di controlli di integrità. La versione 1.0 dell'API `mpHealth` definisce un singolo endpoint. La prossima versione ne fornirà due, uno per l'attività e uno per la disponibilità.

Per utilizzare l'API MicroProfile Health con Liberty, aggiungi la funzione `mpHealth` a `server.xml`:

```xml
<featureManager>
   <feature>mpHealth-1.0</feature>
</featureManager>
```
{: codeblock}

Puoi controllare lo stato dell'applicazione con un browser accedendo all'endpoint `/health`. Il codice restituisce un payload `{"outcome": "UP"}` quando va tutto bene. Il seguente codice mostra come utilizzare le API MicroProfile per indicare se il pod è integro o disponibile. Il metodo `call` consente agli sviluppatori di fornire un algoritmo di controllo per indicare lo stato di integrità di un pod. Riuscire a identificare se un pod sta esaurendo la memoria è un buon indicatore della sua integrità. Controllare la connessione al database di downstream può essere un buon controllo per la disponibilità del pod. Se non c'è alcun controllo specifico, puoi utilizzare quanto segue per indicare se un pod è attivo o disponibile.

```java
import org.eclipse.microprofile.health.Health;
import org.eclipse.microprofile.health.HealthCheck;
import org.eclipse.microprofile.health.HealthCheckResponse;

import javax.enterprise.context.ApplicationScoped;

@Health
@ApplicationScoped
public class HealthResource implements HealthCheck {

    @Override
    public HealthCheckResponse call() {
        return HealthCheckResponse.named("service-a").withData("service-a", "ok").up().build();
    }
}
```
{: codeblock}

Devi quindi specificare `livenessProbe` e `readinessProbe` per Kubernetes, come mostrato di seguito.
```yaml
livenessProbe:
  exec:
    command: ["sh", "-c", "curl -s http://localhost:9080/health| grep -q service-a"]
  initialDelaySeconds: 60
readinessProbe:
  exec:
    command: ["sh", "-c", "curl -s http://localhost:9080/health | grep -q service-a"]
  initialDelaySeconds: 40
```
{: codeblock}

Come menzionato in precedenza, puoi utilizzare delle logiche differenti per indicare disponibilità e attività. Puoi utilizzare MicroProfile Health per indicare un controllo di disponibilità e per creare un endpoint JAX-RS per il controllo di disponibilità o viceversa. La prossima versione di MicroProfile Health fornirà due endpoint, uno per l'attività e l'altro per la disponibilità.

OpenLiberty ha una guida pratica che fornisce un esempio di come aggiungere controlli personalizzati all'endpoint MicroProfile Health: https://openliberty.io/guides/microprofile-health.html

Puoi sovrascrivere la modalità di funzionamento predefinita dell'endpoint MicroProfile Health come descritto in [Performing MicroProfile health checks](https://www.ibm.com/support/knowledgecenter/en/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/twlp_microprofile_healthcheck.html){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno").
