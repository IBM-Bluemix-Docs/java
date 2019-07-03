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

# Consignation dans Spring
{: #spring-logging}

Les messages de journal sont des chaînes d'informations contextuelles qui se rapportent à l'état et l'activité d'un microservice quand une entrée de journal est créée. Vous pouvez utiliser la consignation pour diagnostiquer pourquoi et comment des services échouent, ainsi que comme aide pour des métriques de surveillance de la santé des applications.

Les entrées de journal doivent être écrites directement dans la sortie standard et les flux d'erreurs, ce qui rend les entrées de journal consultables via les outils de ligne de commande. Vous pouvez ensuite utiliser des services d'acheminement des journaux configurés au niveau de l'infrastructure, comme Logstash, Filebeat ou Fluentd, pour gérer la collecte de journaux.

## Ajout d'un support Logback aux applications Spring
{: #spring-log4j}

[Logback](https://logback.qos.ch/){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe") est le moteur de journal par défaut utilisé par Spring Boot. Il est utilisé automatiquement lorsqu'il se trouve dans le chemin de classe. La plupart des modules de démarrage Spring Boot dépendent temporairement de Logback via `spring-boot-starter-logging`.

L'exemple suivant (basé sur un [exemple de Spring Boot](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-samples/spring-boot-sample-logback/src/main/java/sample/logback/SampleLogbackApplication.java)){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe") utilise l'API de consignation SLF4J, que Logback implémente, pour initialiser un `Logger` et émettre des messages à différents niveaux de consignation :

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

Le niveau de consignation par défaut est `INFO`. Vous pouvez spécifier différents niveaux de consignation pour certains modules Java&trade; dans `application.properties`. Ainsi, pour définir le niveau de consignation par défaut sur `WARN` (avertissement), le niveau du package `sample.logging` sur `DEBUG` (débogage) et le niveau du package Spring `org.springframework.web` sur `ERROR` (erreur), ajoutez le code suivant à `application.properties` :

```properties
logging.level.root=WARN
logging.level.org.springframework.web=ERROR
logging.level. sample.logging=DEBUG
```
{: codeblock}

Par défaut, Logback génère des messages avec les éléments suivants :

- Date
- Heure avec une précision à la milliseconde
- Niveau de consignation : `ERROR`, `WARN`, `INFO`, `DEBUG` ou `TRACE`
- ID processus
- Un séparateur "`---`" afin de définir le début des messages de journal réels
- [Nom d'unité d'exécution (peut être tronqué pour la sortie de console)]
- Nom du consignateur : Ce nom est généralement le nom de la classe source (souvent abrégé)
- Message de journal

Les messages de journal sont similaires à l'exemple suivant :

```
2018-12-12 09:59:34.751  INFO 11174 --- [           main] sample.logging.SampleLogbackApplication  : Sample INFO Message
2018-12-12 09:59:34.751  WARN 11174 --- [           main] sample.logging.SampleLogbackApplication  : Sample WARN Message
2018-12-12 09:59:34.752 ERROR 11174 --- [           main] sample.logging.SampleLogbackApplication  : Sample ERROR Message
```
{: screen}

### Création de journaux JSON
{: #spring-json-logs}

Activez la consignation JSON dans des applications Spring en utilisant `logstash-logback-encoder`.

Ajoutez le codeur logback comme dépendance :

```xml
<dependency>
  <groupId>net.logstash.logback</groupId>
  <artifactId>logstash-logback-encoder</artifactId>
  <version>4.7</version>
</dependency>
```
{: codeblock}

Configurez logback pour qu'il utilise le nouveau codeur avec `ConsoleAppender` dans `logback.xml`, comme illustré dans l'exemple suivant :

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

Avec cette configuration, vous pouvez voir des journaux au format JSON : 

```
{"@timestamp":"2018-10-11T23:48:57.215+00:00","@version":1,"message":"Sample TRACE Message","logger_name":"com.example.demo.LoggingExample","thread_name":"http-nio-8080-exec-1","level":"TRACE","level_value":5000}
{"@timestamp":"2018-10-11T23:48:57.216+00:00","@version":1,"message":"Sample DEBUG Message","logger_name":"com.example.demo.LoggingExample","thread_name":"http-nio-8080-exec-1","level":"DEBUG","level_value":10000}
{"@timestamp":"2018-10-11T23:48:57.216+00:00","@version":1,"message":"Sample INFO Message","logger_name":"com.example.demo.LoggingExample","thread_name":"http-nio-8080-exec-1","level":"INFO","level_value":20000}
{"@timestamp":"2018-10-11T23:48:57.217+00:00","@version":1,"message":"Sample WARN Message","logger_name":"com.example.demo.LoggingExample","thread_name":"http-nio-8080-exec-1","level":"WARN","level_value":30000}
{"@timestamp":"2018-10-11T23:48:57.217+00:00","@version":1,"message":"Sample ERROR Message","logger_name":"com.example.demo.LoggingExample","thread_name":"http-nio-8080-exec-1","level":"ERROR","level_value":40000}
```
{: screen}

Vous pouvez personnaliser [`logstash-logback-encoder`](https://github.com/logstash/logstash-logback-encoder){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe") pour modifier la façon dont les journaux JSON sont capturés et transférés.

Utilisez `jq` pour filtrer les journaux JSON afin de simplifier la lecture. Exemple :

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

## Etapes suivantes
{: #spring-logging-next-steps notoc}

Pour en savoir plus sur la personnalisation des messages de journal à l'aide des programmes d'ajout, des niveaux de consignation et des détails de configuration, veuillez consulter les [références Spring Boot officielles relatives à la consignation](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-logging.html){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe").

En savoir plus sur l'affichage des journaux dans chacun des environnements de déploiement :

* [Journaux de Kubernetes](https://kubernetes.io/docs/concepts/cluster-administration/logging/){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe")
* [Journaux & surveillance d'{{site.data.keyword.openwhisk}}](/docs/openwhisk?topic=cloud-functions-logs)
* [{{site.data.keyword.cloud_notm}} - Analyse des journaux](/docs/services/CloudLogAnalysis?topic=cloudloganalysis-log_analysis_ov#log_analysis_ov)
* [{{site.data.keyword.cloud_notm}} - Pile ELK privée](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_2.1.0.2/manage_metrics/logging_elk.html){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe")
