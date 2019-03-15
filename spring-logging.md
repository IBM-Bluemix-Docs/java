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

# Logging in Spring
{: #spring-logging}

Log messages are strings of contextual information that pertain to the state and activity of a microservice when a log entry is created. You can use logging to diagnose how and why services fail, and to assist with metrics for monitoring application health.

Log entries should be written directly to standard output and error streams. This makes log entries viewable using command line tools, and allows log forwarding services configured at the infrastructure level, like Logstash, Filebeat, or Fluentd, to manage log collection.

## Adding Logback support to Spring applications
{: #spring-log4j}

[Logback](https://logback.qos.ch/){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon") is the default log engine that is used by Spring Boot, and is used automatically when found in the class path. Most Spring Boot starters transitively depend on Logback through `spring-boot-starter-logging`.

The following example (based on a [Spring Boot sample](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-samples/spring-boot-sample-logback/src/main/java/sample/logback/SampleLogbackApplication.java)){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon") uses the SLF4J logging API, which Logback implements, to initialize a `Logger` and emit messages at various log levels:

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

The default log level is `INFO`. You can specify different logging levels for specific Java packages in `application.properties`. For example, to set the default log level to `WARN`, the level for package `sample.logging` to `DEBUG`, and the level for the Spring package `org.springframework.web` to `ERROR`, add the following to `application.properties`:

```properties
logging.level.root=WARN
logging.level.org.springframework.web=ERROR
logging.level. sample.logging=DEBUG
```
{: codeblock}

By default, Logback produces messages with the following elements:

- Date
- Time with millisecond precision
- Log Level: `ERROR`, `WARN`, `INFO`, `DEBUG`, or `TRACE`.
- Process ID.
- A "`---`" separator to define the start of actual log messages.
- [Thread name (may be truncated for console output)]
- Logger name: The name is usually the source class name (often abbreviated).
- The log message.

Log messages look something like the following example:

```
2018-12-12 09:59:34.751  INFO 11174 --- [           main] sample.logging.SampleLogbackApplication  : Sample INFO Message
2018-12-12 09:59:34.751  WARN 11174 --- [           main] sample.logging.SampleLogbackApplication  : Sample WARN Message
2018-12-12 09:59:34.752 ERROR 11174 --- [           main] sample.logging.SampleLogbackApplication  : Sample ERROR Message
```
{: screen}

### Creating JSON Logs
{: #spring-json-logs}

Enable JSON logging in Spring applications using the `logstash-logback-encoder`.

Add the logback encoder as a dependency:

```xml
<dependency>
  <groupId>net.logstash.logback</groupId>
  <artifactId>logstash-logback-encoder</artifactId>
  <version>4.7</version>
</dependency>
```
{: codeblock}

Configure logback to use the new encoder with a `ConsoleAppender` in `logback.xml`. It should look something like the following:

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

With this configuration, you'll now see JSON-formatted logs:

```
{"@timestamp":"2018-10-11T23:48:57.215+00:00","@version":1,"message":"Sample TRACE Message","logger_name":"com.example.demo.LoggingExample","thread_name":"http-nio-8080-exec-1","level":"TRACE","level_value":5000}
{"@timestamp":"2018-10-11T23:48:57.216+00:00","@version":1,"message":"Sample DEBUG Message","logger_name":"com.example.demo.LoggingExample","thread_name":"http-nio-8080-exec-1","level":"DEBUG","level_value":10000}
{"@timestamp":"2018-10-11T23:48:57.216+00:00","@version":1,"message":"Sample INFO Message","logger_name":"com.example.demo.LoggingExample","thread_name":"http-nio-8080-exec-1","level":"INFO","level_value":20000}
{"@timestamp":"2018-10-11T23:48:57.217+00:00","@version":1,"message":"Sample WARN Message","logger_name":"com.example.demo.LoggingExample","thread_name":"http-nio-8080-exec-1","level":"WARN","level_value":30000}
{"@timestamp":"2018-10-11T23:48:57.217+00:00","@version":1,"message":"Sample ERROR Message","logger_name":"com.example.demo.LoggingExample","thread_name":"http-nio-8080-exec-1","level":"ERROR","level_value":40000}
```
{: screen}

The [`logstash-logback-encoder`](https://github.com/logstash/logstash-logback-encoder){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon") can be further customized to alter how JSON logs are captured and forwarded.

Use `jq` to filter the JSON logs for easier reading. For example:

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

## Next Steps
{: #next_steps notoc}

For more information about customizing log messages with appenders, log levels, and configuration details, see the official [Spring Boot reference for logging](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-logging.html){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon").

Learn more about viewing the logs in each of our deployment environments:

* [Kubernetes Logs](https://kubernetes.io/docs/concepts/cluster-administration/logging/){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon")
* [{{site.data.keyword.openwhisk}} Logs & Monitoring](/docs/openwhisk?topic=cloud-functions-openwhisk_logs#openwhisk_logs)
* [{{site.data.keyword.cloud_notm}} Log Analysis](/docs/services/CloudLogAnalysis?topic=cloudloganalysis-log_analysis_ov#log_analysis_ov)
* [{{site.data.keyword.cloud_notm}} Private ELK stack](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_2.1.0.2/manage_metrics/logging_elk.html){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon")
