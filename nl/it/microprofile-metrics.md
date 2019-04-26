---

copyright:
  years: 2019
lastupdated: "2019-03-27"

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

MicroProfile fornisce una funzionalità Metrics che ti consente di utilizzare delle semplici annotazioni per aggiungere metriche personalizzate alla tua applicazione. Per eseguirne l'abilitazione, aggiungi semplicemente la funzione `mpMetrics-1.1` al tuo `server.xml`. Nota: facoltativamente, puoi anche aggiungere la funzione `monitor-1.0`, se desideri delle metriche specifiche per il server delle applicazioni (come ad esempio quelle relative ai pool di connessioni JDBC).

Importa l'annotazione `@Counted` per creare un semplice contatore.

```java
import org.eclipse.microprofile.metrics.annotation.Counted;
```
{: codeblock}

Utilizza quindi l'annotazione `@Counted` per creare un semplice contatore. Questo esempio conta quante volte è stato richiamato il metodo `createPortfolio`:

```java
@POST
@Path("/{owner}")
@Produces("application/json")
@Counted(monotonic=true, name="portfolios", displayName="Stock Trader portfolios", description="Number of portfolios created in the Stock Trader applications")
@RolesAllowed({"StockTrader"})
public JsonObject createPortfolio(@PathParam("owner") String owner) throws SQLException {
```
{: codeblock}

Per creare questo codice, dobbiamo aggiungere la seguente stanza al nostro `pom.xml` Maven:

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

Quando ne viene eseguita l'implementazione, ogni richiamo del metodo JAX-RS createPortfolio determina un incremento del contatore. Puoi richiamare l'URI `GET /metrics` per visualizzare sia le metriche jvm (statistiche di raccolte di dati obsoleti, heap e caricamento classi) che quelle definite dall'applicazione. L'URI `GET /metrics/application` restituirà solo le metriche definite dall'applicazione. Possiamo raggiungere questa API REST mediante la CLI curl, utilizzando la porta assegnata (32388 in questo esempio).

```
Johns-MacBook-Pro-8:StockTrader jalcorn$ curl http://9.42.2.107:32388/metrics/application

# TYPE application:com_ibm_hybrid_cloud_sample_portfolio_portfolio_portfolios counter

# HELP application:com_ibm_hybrid_cloud_sample_portfolio_portfolio_portfolios Number of portfolios created in the Stock Trader applications

application:com_ibm_hybrid_cloud_sample_portfolio_portfolio_portfolios

Johns-MacBook-Pro-8:StockTrader jalcorn$
```
{: screen}

Come puoi vedere, ha contato che abbiamo creato 2 portfolio. Alcuni elementi degni di nota:

- Questo è un contatore in memoria; se il pod viene riavviato, il valore viene reimpostato su zero; se ci sono più repliche, ciascuna avrà un suo valore (localizzato).
- Il testo "# HELP" è quello che abbiamo specificato come descrizione nell'annotazione `@Counted`.

Puoi anche visualizzare l'output di questo endpoint REST GET nel tuo browser web:

![Browser web dell'endpoint REST GET](images/microprofile-metrics-image1.png "Browser web dell'endpoint REST GET ")

Nota che, per impostazione predefinita, l'endpoint `/metrics` richiede che siano passati https e le credenziali di accesso. Liberty 18.0.0.3 ha introdotto la seguente stanza dove puoi inserire il tuo server.xml per indicare che desideri che questo endpoint consenta http e di non essere autenticato:

```xml
<mpMetrics authentication="false"/>
```

Questo rende molto più facile la configurazione dello scraper Prometheus per accedere a questo endpoint.
