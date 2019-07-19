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

# In Spring protokollieren
{: #spring-logging}

Bei Protokollnachrichten handelt es sich um Zeichenfolgen mit Kontextinformationen, die sich auf den Status und die Aktivitäten eines Mikroservice zum Zeitpunkt der Protokolleintragserstellung beziehen. Mithilfe der Protokollierung können Sie das Auftreten von Servicefehlern sowie die zugrunde liegenden Ursachen diagnostizieren. Darüber hinaus bietet die Protokollierung Unterstützung bei Metriken für die Überwachung des Anwendungsstatus.

Protokolleinträge sollten direkt in die Standardausgabe und die Fehlerdatenströme geschrieben werden, damit sie mithilfe von Befehlszeilentools angezeigt werden können. Anschließend können auf der Ebene der Infrastruktur konfigurierte Protokollweiterleitungsservices wie Logstash, Filebeat oder Fluentd verwendet werden, um die Protokollerfassung zu verwalten. 

## Logback-Unterstützung zu Spring-Apps hinzufügen
{: #spring-log4j}

[Logback](https://logback.qos.ch/){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") ist die Standardprotokollengine, die von Spring Boot verwendet und automatisch eingesetzt wird, falls sie im Klassenpfad enthalten ist. Die meisten Spring Boot-Starter sind über `spring-boot-starter-logging` transitiv von Logback abhängig.

Im folgenden Beispiel (das auf einem [Spring Boot-Beispiel](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-samples/spring-boot-sample-logback/src/main/java/sample/logback/SampleLogbackApplication.java) basiert){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") wird die Protokollierungs-API SLF4J, die von Logback implementiert wird, verwendet, um eine `Protokollfunktion` zu initialisieren und Nachrichten auf verschiedenen Protokollebenen auszugeben:

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

Die Standardprotokollebene lautet `INFO`. Sie können verschiedene Protokollebenen für bestimmte Java&trade;-Pakete in der Datei `application.properties` angeben. Fügen Sie beispielsweise Folgendes zu `application.properties` hinzu, um den Wert `WARN` für die Standardprotokollebene, den Wert `DEBUG` für die Protokollebene des Pakets `sample.logging` und den Wert `ERROR` für die Protokollebene des Spring-Pakets `org.springframework.web` festzulegen:

```properties
logging.level.root=WARN
logging.level.org.springframework.web=ERROR
logging.level. sample.logging=DEBUG
```
{: codeblock}

Standardmäßig generiert Logback Nachrichten mit den folgenden Elementen:

- Datum
- Zeit mit einer Genauigkeit von Millisekunden
- Protokollebene: `ERROR`, `WARN`, `INFO`, `DEBUG` oder `TRACE`
- Prozess-ID
- Trennzeichen "`---`" zur Definition des Beginns der eigentlichen Protokollnachrichten
- [Threadname (kann für die Konsolenausgabe abgeschnitten sein)]
- Protokollfunktionsname: Normalerweise der Name der Quellenklasse (häufig abgekürzt)
- Protokollnachricht

Protokollnachrichten sehen in etwa wie das folgenden Beispiel aus:

```
2018-12-12 09:59:34.751  INFO 11174 --- [           main] sample.logging.SampleLogbackApplication  : Sample INFO Message
2018-12-12 09:59:34.751  WARN 11174 --- [           main] sample.logging.SampleLogbackApplication  : Sample WARN Message
2018-12-12 09:59:34.752 ERROR 11174 --- [           main] sample.logging.SampleLogbackApplication  : Sample ERROR Message
```
{: screen}

### JSON-Protokolle erstellen
{: #spring-json-logs}

Aktivieren Sie die JSON-Protokollierung in Spring-Apps mit `logstash-logback-encoder`. 

Fügen Sie den Logback-Encoder als Abhängigkeit hinzu:

```xml
<dependency>
  <groupId>net.logstash.logback</groupId>
  <artifactId>logstash-logback-encoder</artifactId>
  <version>4.7</version>
</dependency>
```
{: codeblock}

Konfigurieren Sie Logback für die Verwendung des neuen Encoders mit `ConsoleAppender` in `logback.xml`, wie im folgenden Beispiel dargestellt: 

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

Mit dieser Konfiguration werden Protokolle im JSON-Format angezeigt: 

```
{"@timestamp":"2018-10-11T23:48:57.215+00:00","@version":1,"message":"Sample TRACE Message","logger_name":"com.example.demo.LoggingExample","thread_name":"http-nio-8080-exec-1","level":"TRACE","level_value":5000}
{"@timestamp":"2018-10-11T23:48:57.216+00:00","@version":1,"message":"Sample DEBUG Message","logger_name":"com.example.demo.LoggingExample","thread_name":"http-nio-8080-exec-1","level":"DEBUG","level_value":10000}
{"@timestamp":"2018-10-11T23:48:57.216+00:00","@version":1,"message":"Sample INFO Message","logger_name":"com.example.demo.LoggingExample","thread_name":"http-nio-8080-exec-1","level":"INFO","level_value":20000}
{"@timestamp":"2018-10-11T23:48:57.217+00:00","@version":1,"message":"Sample WARN Message","logger_name":"com.example.demo.LoggingExample","thread_name":"http-nio-8080-exec-1","level":"WARN","level_value":30000}
{"@timestamp":"2018-10-11T23:48:57.217+00:00","@version":1,"message":"Sample ERROR Message","logger_name":"com.example.demo.LoggingExample","thread_name":"http-nio-8080-exec-1","level":"ERROR","level_value":40000}
```
{: screen}

[`logstash-logback-encoder`](https://github.com/logstash/logstash-logback-encoder){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") kann zusätzlich angepasst werden, um Änderungen bei der Erfassung und Weiterleitung von JSON-Protokollen vorzunehmen.

Verwenden Sie `jq` zum Filtern der JSON-Protokolle, um die Lesbarkeit zu verbessern. Beispiel:

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

## Nächste Schritte
{: #spring-logging-next-steps notoc}

Weitere Informationen zum Anpassen von Protokollnachrichten mit Appendern, Protokollebenen und Konfigurationsdetails finden Sie in der offiziellen [Spring Boot-Referenz für die Protokollierung](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-logging.html){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link").

Weitere Informationen zum Anzeigen von Protokollen in jeder der folgenden Bereitstellungsumgebungen: 

* [Kubernetes-Protokolle](https://kubernetes.io/docs/concepts/cluster-administration/logging/){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link")
* [Protokolle & Überwachung in {{site.data.keyword.openwhisk}}](/docs/openwhisk?topic=cloud-functions-logs)
* [{{site.data.keyword.cloud_notm}} Log Analysis](/docs/services/CloudLogAnalysis?topic=cloudloganalysis-log_analysis_ov#log_analysis_ov)
* [{{site.data.keyword.cloud_notm}} Private-ELK-Stack](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_2.1.0.2/manage_metrics/logging_elk.html){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link")
