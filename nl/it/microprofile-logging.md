---

copyright:
  years: 2019
lastupdated: "2019-05-20"

keywords: java logging, log level java, debug java, json log java, json log help, kibana liberty, liberty messages

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

# Registrazione
{: #mp-logging}

L'approccio consigliato per la registrazione con le applicazioni MicroProfile è lo standard di registrazione JSR-47 di Java&trade;. Puoi iniziare con le seguenti importazioni:

```java
import java.util.logging.Level;
import java.util.logging.Logger;
```
{: codeblock}

Istanzia quindi un'istanza di un logger a livello di classe:

```java
private static Logger logger = Logger.getLogger(PortfolioService.class.getName());
```
{: codeblock}

Aggiungi le chiamate all'istanza `logger` prima, dopo e all'interno delle operazioni del tuo codice. I metodi dell'interfaccia `Logger` sono essi stessi denominati per indicare l'importanza, o il "livello", delle informazioni che si stanno registrando.

```java
logger.info("Creating portfolio for "+owner);
logger.warning("Unable to send message to JMS provider. Continuing without notification of change in loyalty level.");
```
{: codeblock}

Il livello di log viene visualizzato quando questi messaggi vengono generati in output sulla console.

```
[INFO] Creating portfolio for John

[WARNING] Unable to send message to JMS provider. Continuing without notification of change in loyalty level.
```
{: screen}

I livelli di log ti offrono la flessibilità di scegliere dinamicamente quali log scrive la tua applicazione. Questa funzione ti consente di scrivere del codice di log che descrive sia lo stato delle applicazioni di alto livello che il contenuto di debug dettagliato senza molto sforzo. Per cui puoi escludere mediante filtro il contenuto di debug più dettagliato finché non ne hai bisogno. Il livello di log `info` è di norma il livello di output minimo, seguito da `fine`, `finer`, `finest` e `debug`.

Se una voce di log richiede più righe di codice, o comporta delle operazioni che utilizzano molta potenza di elaborazione come la concatenazione di stringhe, devi valutarne la protezione con un test per determinare se il livello di log è abilitato. Aggiungendo il controllo si garantisce che la tua applicazione non dedichi tempo prezioso a creare dei messaggi di log che finiscono per essere esclusi mediante filtro. Nel seguente esempio, il livello di log previsto di `fine` viene abilitato prima di provare a creare l'output del messaggio.

```java
if (logger.isLoggable(Level.FINE)) {
    StringWriter writer = new StringWriter();
    exception.printStackTrace(new PrintWriter(writer));
    logger.fine(writer.toString());
}
```
{: codeblock}

Per ulteriori informazioni sui livelli di log e i dettagli della configurazione, vedi il manuale [WebSphere Liberty troubleshooting guide](https://www.ibm.com/support/knowledgecenter/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/rwlp_logging.html){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno") e la [documentazione dell'API di registrazione java.util.logging](https://docs.oracle.com/javase/8/docs/api/java/util/logging/package-summary.html){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno").

## Registrazione JSON con Liberty
{: #mp-json-logging}

Liberty supporta la registrazione con formato JSON. Quando è abilitata, i messaggi di log vengono scritti sulla console in formato JSON. Abilita questa opzione utilizzando la seguente stanza di registrazione nel tuo `server.xml`:

```xml
<logging consoleLogLevel="INFO" consoleFormat="json" consoleSource="message,trace,accessLog,ffdc" />
```
{: codeblock}

Sebbene `accessLog` è incluso nell'elenco di origini della console, la registrazione dell'accesso HTTP deve essere abilitata prima che questi log vengono scritti sulla console. Il seguente frammento di codice mostra come aggiungere l'elemento secondario `accessLogging` all'elemento `httpEndpoint` nel tuo `server.xml`:

```xml
<httpEndpoint id="defaultHttpEndpoint" host="\*" httpPort="9080" httpsPort="9443">
  <accessLogging
    filepath="${server.output.dir}/logs/http_defaultEndpoint_access.log"
    logFormat='%h %u %t "%r" %s %b %D %{User-agent}i'>
  </accessLogging>
</httpEndpoint>
```
{: codeblock}

Ora, quando aggiungi questo codice alla tua applicazione:

```java
if (logger.isLoggable(Level.AUDIT)) {
    logger.audit("Initialization complete");
}
```

Puoi visualizzare il seguente output nei log:

```json
{ "type":"liberty_message",
  "host":"trader-54b4d579f7-4zvzk",
  "ibm_userDir":"\/opt\/ol\/wlp\/usr\/",
  "ibm_serverName":"defaultServer",
  "ibm_datetime":"2018-06-21T19:23:21.356+0000",
  "ibm_threadId":"00000028",
  "module":"com.trader.Main",
  "loglevel":"AUDIT",
  "message":"Initialization complete"}
```
{: codeblock}

### Lettura dell'output di log JSON
{: #mp-json-log-output}

L'output JSON completo è utile per l'archiviazione e le ricerche nei log ma non molto facile da leggere. Puoi esaminare il contenuto del log in una finestra di terminale utilizzando il comando `kubectl`. Fortunatamente è disponibile come ausilio uno strumento di riga di comando denominato `jq`.

Con il comando `jq`, puoi filtrare ed evidenziare il campo o i campi di cui hai bisogno. Se vuoi vedere il campo `message` ed escludere dal filtro tutto il resto, vedi il seguente esempio:

```
kubectl logs trader-54b4d579f7-4zvzk -n stock-trader -c trader | grep message | jq .message -r
```
{: pre}

Liberty ha alcuni messaggi della console primitiva che non hanno una formattazione JSON. Puoi utilizzare il comando `grep` per assicurarti che `jq` analizzi specificatamente le righe che contengono un campo message.

## Funzioni aggiuntive
{: #mp-log-features}

Le linee guida per l'utilizzo dei livelli di log, come quando utilizzi `logger.info` o `logger.fine`, sono elementi che devono essere decisi da ciascun progetto o da ciascuna organizzazione. In generale, queste interfacce sono necessarie e utili in quasi tutti i progetti. 

Una prassi ottimale consiste nell'utilizzare le variabili di ambiente (fornite al pod tramite le mappe di configurazione Kubernetes o i segreti) in ogni campo pertinente nel file `server.xml`. Utilizzando questo metodo, puoi modificare la configurazione della registrazione senza dover ricreare e ridistribuire la tua immagine Docker. 

Ad esempio, per utilizzare le variabili di ambiente per impostare gli attributi di registrazione dettagliati, devi modificare la stanza proveniente dall'esempio precedente:

```xml
<logging consoleLogLevel="INFO" consoleFormat="json" consoleSource="message,trace,accessLog,ffdc" />
```
{: codeblock}

Per la seguente voce:

```xml
<logging consoleLogLevel="${env.LOG_LEVEL}" consoleFormat="${env.LOG_FORMAT}" consoleSource="${env.LOG_SOURCE}" />
```
{: codeblock}

Un'altra alternativa consiste nell'utilizzare la variabile di ambiente `WLP_LOGGING_CONSOLE_FORMAT`, come descritto nella [documentazione Logging and Trace](https://www.ibm.com/support/knowledgecenter/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/rwlp_logging.html){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno"). Questo metodo è simile all'esempio precedente e puoi impostare la variabile `WLP_LOGGING_CONSOLE_FORMAT` su `basic` (il valore predefinito) o su `json`.

## Dashboard Kibana per Liberty
{: #liberty-kibana}

Insieme alla nuova funzione di registrazione JSON, Liberty fornisce dashboard Kibana precostruiti [che puoi scaricare da GitHub](https://www.ibm.com/support/knowledgecenter/en/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/twlp_icp_json_logging.html){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno"). Segui le istruzioni presenti nel link per installarli. Sono ora disponibili due nuovi dashboard: 

![Dashboard Kibana](images/microprofile-logging-image4.png "Dashboard Kibana")

Quando selezioni il dashboard per la determinazione dei problemi, puoi vedere: 

![Dettagli dashboard Kibana](images/microprofile-logging-image5.png "Dettagli dashboard Kibana")

Il dashboard è interattivo. Ad esempio, se scegli **INFO** nella legenda del widget **Liberty Message**, il widget **Liberty Messages Search** si filtrerà alla ricerca dei soli messaggi `loglevel=INFO`. Il dashboard federa i dati di log da tutti i tuoi microservizi basati su Liberty, escludendo dal filtro gli altri log di sistema. 

Ci sono altri dashboard Kibana e Grafana associati al grafico helm Liberty. Sono disponibili come [estensioni al pacchetto cloud Liberty](https://github.com/IBM/charts/tree/master/stable/ibm-websphere-liberty/ibm_cloud_pak/pak_extensions/dashboards){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno").

## Passi successivi
{: #mp-logging-next-steps notoc}

Per ulteriori informazioni sulla personalizzazione dei messaggi di log con gli appender, i livelli di log e i dettagli di configurazione, consulta la [guida di riferimento di Spring Boot per la registrazione](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-logging.html){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno").ufficiale.

Ulteriori informazioni sulla visualizzazione dei log in ognuno dei seguenti ambienti di distribuzione:

* [Log Kubernetes](https://kubernetes.io/docs/concepts/cluster-administration/logging/){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno")
* [{{site.data.keyword.openwhisk}} Logs & Monitoring](/docs/openwhisk?topic=cloud-functions-openwhisk_logs#openwhisk_logs)
* [{{site.data.keyword.cloud_notm}} Log Analysis](/docs/services/CloudLogAnalysis?topic=cloudloganalysis-log_analysis_ov#log_analysis_ov)
* [{{site.data.keyword.cloud_notm}} Stack ELK privato stack](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_2.1.0.2/manage_metrics/logging_elk.html){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno")
