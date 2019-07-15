---

copyright:
  years: 2018, 2019
lastupdated: "2019-06-10"

keywords: spring logging, spring logger, logback spring, debug spring, json log spring, consoleappender spring, spring boot log

subcollection: java

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}

# Accesso a Spring
{: #spring-logging}

I messaggi di log sono stringhe di informazioni contestuali che riguardano lo stato e l'attività del microservizio nel momento in cui viene creata una voce di log. Puoi utilizzare la registrazione per diagnosticare modalità e causa dei malfunzionamenti dei servizi e per un ausilio con le metriche per il monitoraggio dell'integrità delle applicazioni.

Le voci di log devono essere scritte direttamente nei flussi di errore e nell'output standard che rende le voci di log visualizzabili tramite gli strumenti di riga di comando. Per gestire la raccolta di log, puoi quindi utilizzare i log tramite l'inoltro dei servizi configurati a livello dell'infrastruttura, come Logstash, Filebeat o Fluentd.

## Aggiunta del supporto Logback alle applicazioni Spring
{: #spring-log4j}

[Logback](https://logback.qos.ch/){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno") è il motore di log predefinito utilizzato da Spring Boot e viene utilizzato automaticamente quando viene rilevato nel percorso classi. La maggior parte degli starter Spring Boot dipende transitivamente da Logback tramite `spring-boot-starter-logging`.

Il seguente esempio (basato su un [esempio di Spring Boot](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-samples/spring-boot-sample-logback/src/main/java/sample/logback/SampleLogbackApplication.java)){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno") utilizza l'API di registrazione SLF4J, implementata da Logback, per inizializzare un `Logger` ed emettere messaggi a diversi livelli di log.

```java
package sample.logging;

import javax.annotation.PostConstruct;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SampleLogbackApplication {
  private static final Logger logger = LoggerFactory.getLogger(SampleLogbackApplication.class);

  @PostConstruct
  public void logSomething() {
    logger.trace("Sample TRACE Message");
    logger.debug("Sample DEBUG Message");
    logger.info("Sample INFO Message");
    logger.warn("Sample WARN Message");
    logger.error("Sample ERROR Message");
  }

  public static void main(String[] args) {
    SpringApplication.run(SampleLogbackApplication.class, args).close();
  }
}
```
{: codeblock}

Il livello di log predefinito è `INFO`. Puoi specificare diversi livelli di registrazione per pacchetti Java&trade; specifici in `application.properties`. Ad esempio, per impostare il livello di log predefinito su `WARN`, il livello di log per il pacchetto `sample.logging` su `DEBUG` e il livello di log per il pacchetto Spring `org.springframework.web` su `ERROR`, aggiungi quanto segue a `application.properties`:

```properties
logging.level.root=WARN
logging.level.org.springframework.web=ERROR
logging.level. sample.logging=DEBUG
```
{: codeblock}

Per impostazione predefinita, Logback produce messaggi con i seguenti elementi:

- Data
- Tempo con precisione al millisecondo
- Livello di log: `ERROR`, `WARN`, `INFO`, `DEBUG` o `TRACE`.
- ID processo.
- Un separatore "`---`" per definire l'inizio degli effettivi messaggi di log.
- [Nome thread (potrebbe essere troncato per l'output della console)]
- Nome logger: il nome di norma è il nome della classe di origine (spesso abbreviato).
- Il messaggio di log.

I messaggi di log sono simili al seguente esempio:

```
2018-12-12 09:59:34.751  INFO 11174 --- [           main] sample.logging.SampleLogbackApplication  : Sample INFO Message
2018-12-12 09:59:34.751  WARN 11174 --- [           main] sample.logging.SampleLogbackApplication  : Sample WARN Message
2018-12-12 09:59:34.752 ERROR 11174 --- [           main] sample.logging.SampleLogbackApplication  : Sample ERROR Message
```
{: screen}

### Creazione di log JSON
{: #spring-json-logs}

Abilita la registrazione JSON nelle applicazioni Spring utilizzando `logstash-logback-encoder`.

Aggiungi il codificatore di logback come una dipendenza:

```xml
<dependency>
  <groupId>net.logstash.logback</groupId>
  <artifactId>logstash-logback-encoder</artifactId>
  <version>4.7</version>
</dependency>
```
{: codeblock}

Configura il logback per utilizzare il nuovo codificatore con un `ConsoleAppender` in `logback.xml` come mostrato nel seguente esempio. 

```xml
<configuration>
    <appender name="consoleAppender" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="net.logstash.logback.encoder.LogstashEncoder"/>
    </appender>
    <logger name="jsonLogger" additivity="false" level="DEBUG">
        <appender-ref ref="consoleAppender"/>
    </logger>
    <root level="INFO">
        <appender-ref ref="consoleAppender"/>
    </root>
</configuration>
```
{: codeblock}

Con questa configurazione, puoi vedere i log con formattazione JSON:

```
{"@timestamp":"2018-10-11T23:48:57.215+00:00","@version":1,"message":"Sample TRACE Message","logger_name":"com.example.demo.LoggingExample","thread_name":"http-nio-8080-exec-1","level":"TRACE","level_value":5000}
{"@timestamp":"2018-10-11T23:48:57.216+00:00","@version":1,"message":"Sample DEBUG Message","logger_name":"com.example.demo.LoggingExample","thread_name":"http-nio-8080-exec-1","level":"DEBUG","level_value":10000}
{"@timestamp":"2018-10-11T23:48:57.216+00:00","@version":1,"message":"Sample INFO Message","logger_name":"com.example.demo.LoggingExample","thread_name":"http-nio-8080-exec-1","level":"INFO","level_value":20000}
{"@timestamp":"2018-10-11T23:48:57.217+00:00","@version":1,"message":"Sample WARN Message","logger_name":"com.example.demo.LoggingExample","thread_name":"http-nio-8080-exec-1","level":"WARN","level_value":30000}
{"@timestamp":"2018-10-11T23:48:57.217+00:00","@version":1,"message":"Sample ERROR Message","logger_name":"com.example.demo.LoggingExample","thread_name":"http-nio-8080-exec-1","level":"ERROR","level_value":40000}
```
{: screen}

Il [`logstash-logback-encoder`](https://github.com/logstash/logstash-logback-encoder){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno") può essere personalizzato ulteriormente per modificare il modo in cui i log JSON vengono acquisiti e inoltrati.

Utilizza `jq` per filtrare i log JSON per una lettura più facile. Ad esempio:

```
$ docker logs --tail 1 -f <container-id> | jq .
{
  "@timestamp": "2018-10-11T23:48:57.215+00:00",
  "@version": 1,
  "message": "Sample TRACE Message",
  "logger_name": "com.example.demo.LoggingExample",
  "thread_name": "http-nio-8080-exec-1",
  "level": "TRACE",
  "level_value": 5000
}

$ docker logs --tail 5 -f <container-id> | jq .message
"Sample TRACE Message"
"Sample DEBUG Message"
"Sample INFO Message"
"Sample WARN Message"
"Sample ERROR Message"
```
{: screen}

## Passi successivi
{: #spring-logging-next-steps notoc}

Per ulteriori informazioni sulla personalizzazione dei messaggi di log con gli appender, i livelli di log e i dettagli di configurazione, consulta la [guida di riferimento di Spring Boot per la registrazione](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-logging.html){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno").ufficiale.

Ulteriori informazioni sulla visualizzazione dei log in ognuno dei seguenti ambienti di distribuzione:

* [Log Kubernetes](https://kubernetes.io/docs/concepts/cluster-administration/logging/){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno")
* [{{site.data.keyword.openwhisk}} Logs & Monitoring](/docs/openwhisk?topic=cloud-functions-logs)
* [{{site.data.keyword.cloud_notm}} Log Analysis](/docs/services/CloudLogAnalysis?topic=cloudloganalysis-log_analysis_ov#log_analysis_ov)
* [{{site.data.keyword.cloud_notm}} Stack ELK privato stack](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_2.1.0.2/manage_metrics/logging_elk.html){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno")
