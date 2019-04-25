---

copyright:
  years: 2018, 2019
lastupdated: "2019-03-15"

keywords: spring logging, spring logger, logback spring, debug spring, json log spring, consoleappender spring, spring boot log

subcollection: java

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}

# Criação de log no Spring
{: #spring-logging}

As mensagens de log são sequências de informações contextuais que pertencem ao estado e à atividade de um microsserviço quando uma entrada de log é criada. É possível usar a criação de log para diagnosticar como e por que os serviços falham e para ajudar com as métricas para monitorar o funcionamento do aplicativo.

As entradas de log devem ser gravadas diretamente na saída padrão e nos fluxos de erro. Isso torna as entradas de log visualizáveis usando ferramentas de linha de comandos e permite que os serviços de encaminhamento de log sejam configurados no nível de infraestrutura, como Logstash, Filebeat ou Fluentd, para gerenciar a coleção de logs.

## Incluindo suporte de Logback nos aplicativos Spring
{: #spring-log4j}

[Logback](https://logback.qos.ch/){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") é o mecanismo de log padrão que é usado pelo Spring Boot e é usado automaticamente quando localizado no caminho de classe. A maioria dos iniciadores do Spring Boot depende transitoriamente do Logback por meio de `spring-boot-starter-logging`.

O exemplo a seguir (com base em uma [Amostra de Spring Boot](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-samples/spring-boot-sample-logback/src/main/java/sample/logback/SampleLogbackApplication.java)){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") usa a API de criação de log SLF4J, que implementa o Logback, para inicializar um `Logger` e emite mensagens em vários níveis de log:

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

O nível de log padrão é `INFO`. É possível especificar diferentes níveis de criação de log para pacotes Java específicos em `application.properties`. Por exemplo, para configurar o nível de log padrão como `WARN`, o nível do pacote `sample.logging` como `DEBUG` e o nível do pacote do Spring `org.springframework.web` como `ERROR`, inclua o seguinte em `application.properties`:

```properties
logging.level.root=WARN
logging.level.org.springframework.web=ERROR
logging.level. sample.logging=DEBUG
```
{: codeblock}

Por padrão, o Logback produz mensagens com os elementos a seguir:

- Date
- Horário com precisão de milissegundo
- Nível de log: `ERROR`, `WARN`, `INFO`, `DEBUG` ou `TRACE`.
- ID do processo.
- Um separador "`---`" para definir o início de mensagens de log reais.
- [O nome do encadeamento (pode ser truncado para saída do console)]
- Nome do criador de logs: o nome é geralmente o nome da classe de origem (geralmente abreviado).
- A mensagem de log.

As mensagens de log são semelhantes ao exemplo a seguir:

```
2018-12-12 09:59:34.751  INFO 11174 --- [           main] sample.logging.SampleLogbackApplication  : Sample INFO Message
2018-12-12 09:59:34.751  WARN 11174 --- [           main] sample.logging.SampleLogbackApplication  : Sample WARN Message
2018-12-12 09:59:34.752 ERROR 11174 --- [           main] sample.logging.SampleLogbackApplication  : Sample ERROR Message
```
{: screen}

### Criando logs do JSON
{: #spring-json-logs}

Ative a criação de log de JSON em aplicativos Spring usando o `logstash-logback-encoder`.

Inclua o codificador de logback como uma dependência:

```xml
<dependency>
  <groupId>net.logstash.logback</groupId>
  <artifactId>logstash-logback-encoder</artifactId>
  <version>4.7</version>
</dependency>
```
{: codeblock}

Configure o logback para usar o novo codificador com um `ConsoleAppender` em `logback.xml`. Ele deve ser semelhante ao seguinte:

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

Com essa configuração, agora você verá os logs formatados por JSON:

```
{"@timestamp":"2018-10-11T23:48:57.215+00:00","@version":1,"message":"Sample TRACE Message","logger_name":"com.example.demo.LoggingExample","thread_name":"http-nio-8080-exec-1","level":"TRACE","level_value":5000}
{"@timestamp":"2018-10-11T23:48:57.216+00:00","@version":1,"message":"Sample DEBUG Message","logger_name":"com.example.demo.LoggingExample","thread_name":"http-nio-8080-exec-1","level":"DEBUG","level_value":10000}
{"@timestamp":"2018-10-11T23:48:57.216+00:00","@version":1,"message":"Sample INFO Message","logger_name":"com.example.demo.LoggingExample","thread_name":"http-nio-8080-exec-1","level":"INFO","level_value":20000}
{"@timestamp":"2018-10-11T23:48:57.217+00:00","@version":1,"message":"Sample WARN Message","logger_name":"com.example.demo.LoggingExample","thread_name":"http-nio-8080-exec-1","level":"WARN","level_value":30000}
{"@timestamp":"2018-10-11T23:48:57.217+00:00","@version":1,"message":"Sample ERROR Message","logger_name":"com.example.demo.LoggingExample","thread_name":"http-nio-8080-exec-1","level":"ERROR","level_value":40000}
```
{: screen}

O [`logstash-logback-encoder`](https://github.com/logstash/logstash-logback-encoder){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") pode ser customizado ainda mais para alterar como os logs JSON são capturados e encaminhados.

Use `jq` para filtrar os logs JSON a fim de facilitar a leitura. Por exemplo:

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

## Próximas etapas
{: #spring-logging-next-steps notoc}

Para obter mais informações sobre como customizar mensagens de log com anexadores, níveis de log e detalhes de configuração, consulte a [Referência oficial do Spring Boot para criação de log](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-logging.html){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo").

Saiba mais sobre como visualizar os logs em cada um dos nossos ambientes de implementação:

* [Logs do Kubernetes](https://kubernetes.io/docs/concepts/cluster-administration/logging/){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo")
* [{{site.data.keyword.openwhisk}} Logs & Monitoramento](/docs/openwhisk?topic=cloud-functions-openwhisk_logs#openwhisk_logs)
* [{{site.data.keyword.cloud_notm}} Log Analysis](/docs/services/CloudLogAnalysis?topic=cloudloganalysis-log_analysis_ov#log_analysis_ov)
* [Pilha ELK do {{site.data.keyword.cloud_notm}} Private](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_2.1.0.2/manage_metrics/logging_elk.html){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo")
