---

copyright:
  years: 2019
lastupdated: "2019-06-10"

keywords: mpmetrics microprofile, mpmetrics, prometheus java, metrics java, microprofile metrics

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

# Metriche con MicroProfile
{: #mp-metrics}

MicroProfile fornisce una funzionalità Metrics per l'aggiunta di metriche personalizzate alla tua applicazione utilizzando delle semplici annotazioni. Per abilitare questa funzione, aggiungi la funzione `mpMetrics-1.1` al tuo `server.xml`. Puoi facoltativamente aggiungere la funzione `monitor-1.0`, se desideri visualizzare ulteriori metriche specifiche per il server delle applicazioni (come ad esempio quelle relative ai pool di connessioni JDBC).

Importa l'annotazione `@Counted` per creare un semplice contatore.

```java
import org.eclipse.microprofile.metrics.annotation.Counted;
```
{: codeblock}

Successivamente, utilizza l'annotazione `@Counted` per creare un semplice contatore, come mostrato nel seguente esempio che conta quante volte viene richiamato il metodo `createPortfoloio`: 

```java
@POST
@Path("/{owner}")
@Produces("application/json")
@Counted(monotonic=true, name="portfolios", displayName="Stock Trader portfolios", description="Number of portfolios created in the Stock Trader applications")
@RolesAllowed({"StockTrader"})
public JsonObject createPortfolio(@PathParam("owner") String owner) throws SQLException {
```
{: codeblock}

Per creare questo codice, aggiungi la seguente stanza al file `pom.xml` di Maven:

```xml
<dependency>
  <groupId>org.eclipse.microprofile</groupId>
  <artifactId>microprofile</artifactId>
  <version>2.0.1</version>
  <type>pom</type>
  <scope>provided</scope>
</dependency>
```
{: codeblock}

Quando ne viene eseguita l'implementazione, il contatore viene incrementato ogni volta che viene richiamato il metodo JAX-RS `createPortfolio`. 

Puoi richiamare l'URI `GET /metrics` per visualizzare sia le metriche JVM (statistiche di raccolte di dati obsoleti, heap e caricamento classi) che quelle definite dall'applicazione. L'URI `GET /metrics/application` restituisce solo le metriche definite dall'applicazione. 

Puoi accedere all'API REST mediante la CLI curl utilizzando la porta assegnata (32388 in questo esempio).

```
Johns-MacBook-Pro-8:StockTrader jalcorn$ curl http://9.42.2.107:32388/metrics/application

# TYPE application:com_ibm_hybrid_cloud_sample_portfolio_portfolio_portfolios counter

# HELP application:com_ibm_hybrid_cloud_sample_portfolio_portfolio_portfolios Number of portfolios created in the Stock Trader applications

application:com_ibm_hybrid_cloud_sample_portfolio_portfolio_portfolios

Johns-MacBook-Pro-8:StockTrader jalcorn$
```
{: screen}

Come puoi vedere, vengono contati due portofli come previsto. 

Alcuni elementi degni di nota:
- Viene utilizzato un contatore in memoria; se il pod viene riavviato, il valore viene reimpostato su zero; se ci sono più repliche, ciascuna ha un suo valore univoco.
- Il testo "# HELP" è quello che viene specificato come descrizione nell'annotazione `@Counted`.

Puoi anche visualizzare l'output di questo endpoint REST GET nel tuo browser web:

![Browser web dell'endpoint REST GET](images/microprofile-metrics-image1.png "Browser web dell'endpoint REST GET ")

Per impostazione predefinita, l'endpoint `/metrics` richiede che siano passati https e le credenziali di accesso. Liberty 18.0.0.3 ha introdotto la seguente stanza che puoi inserire nel tuo `server.xml` per definire questo endpoint in modo che consenta http e di non essere autenticato:

```xml
<mpMetrics authentication="false"/>
```

La configurazione dello scraper Prometheus è stata semplificata per rendere l'accesso all'endpoint più semplice.
