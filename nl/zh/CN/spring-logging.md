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

# Spring 中的日志记录
{: #spring-logging}

日志消息是上下文信息（与创建日志条目时微服务的状态和活动相关）的字符串。您可以使用日志记录来诊断服务发生故障的方式和原因，并帮助提供用于监视应用程序运行状况的度量值。

日志条目应该直接写入到标准输出和错误流。这使日志条目可使用命令行工具进行查看，并允许使用在基础架构级别配置的日志转发服务（例如，Logstash、Filebeat 或 Fluentd）来管理日志收集。

## 向 Spring 应用程序添加 Logback 支持
{: #spring-log4j}

[Logback](https://logback.qos.ch/){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标") 是 Spring Boot 使用的缺省日志引擎，如果位于类路径中，会自动加以使用。大多数 Spring Boot 入门模板通过 `spring-boot-starter-logging` 以及物方式依赖于 Logback。

以下示例（基于 [Spring Boot 样本](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-samples/spring-boot-sample-logback/src/main/java/sample/logback/SampleLogbackApplication.java)）{: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标") 使用 Logback 实现的 SLF4J 日志记录 API 来初始化 `Logger` 并在各个日志级别发出消息：

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

缺省日志级别为 `INFO`。您可以在 `application.properties` 中为特定 Java 包指定不同的日志记录级别。例如，要将缺省日志级别设置为 `WARN`，将软件包 `sample.logging` 的级别设置为 `DEBUG`，并且将 Spring 软件包 `org.springframework.web` 的级别设置为 `ERROR`，请将以下行添加到 `application.properties`：

```properties
logging.level.root=WARN
logging.level.org.springframework.web=ERROR
logging.level. sample.logging=DEBUG
```
{: codeblock}

缺省情况下，Logback 生成包含以下元素的消息：

- 日期
- 精确到毫秒的时间
- 日志级别：`ERROR`、`WARN`、`INFO`、`DEBUG` 或 `TRACE`。
- 进程标识。
- “`---`”分隔符，用于定义实际日志消息的开始。
- [线程名称（可能被控制台输出截断）]
- 记录器名称：名称通常是源类名称（通常为缩写）。
- 日志消息。

日志消息类似于以下示例：

```
2018-12-12 09:59:34.751  INFO 11174 --- [           main] sample.logging.SampleLogbackApplication  : Sample INFO Message
2018-12-12 09:59:34.751  WARN 11174 --- [           main] sample.logging.SampleLogbackApplication  : Sample WARN Message
2018-12-12 09:59:34.752 ERROR 11174 --- [           main] sample.logging.SampleLogbackApplication  : Sample ERROR Message
```
{: screen}

### 创建 JSON 日志
{: #spring-json-logs}

使用 `logstash-logback-encoder` 在 Spring 应用程序中启用 JSON 日志记录。

添加 logback 编码器作为依赖项：

```xml
<dependency>
  <groupId>net.logstash.logback</groupId>
  <artifactId>logstash-logback-encoder</artifactId>
  <version>4.7</version>
</dependency>
```
{: codeblock}

配置 logback 以将新的编码器用于 `logback.xml` 中的 `ConsoleAppender`。应该类似于以下内容：

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

使用此配置，现在您将看到 JSON 格式化日志：

```
{"@timestamp":"2018-10-11T23:48:57.215+00:00","@version":1,"message":"Sample TRACE Message","logger_name":"com.example.demo.LoggingExample","thread_name":"http-nio-8080-exec-1","level":"TRACE","level_value":5000}
{"@timestamp":"2018-10-11T23:48:57.216+00:00","@version":1,"message":"Sample DEBUG Message","logger_name":"com.example.demo.LoggingExample","thread_name":"http-nio-8080-exec-1","level":"DEBUG","level_value":10000}
{"@timestamp":"2018-10-11T23:48:57.216+00:00","@version":1,"message":"Sample INFO Message","logger_name":"com.example.demo.LoggingExample","thread_name":"http-nio-8080-exec-1","level":"INFO","level_value":20000}
{"@timestamp":"2018-10-11T23:48:57.217+00:00","@version":1,"message":"Sample WARN Message","logger_name":"com.example.demo.LoggingExample","thread_name":"http-nio-8080-exec-1","level":"WARN","level_value":30000}
{"@timestamp":"2018-10-11T23:48:57.217+00:00","@version":1,"message":"Sample ERROR Message","logger_name":"com.example.demo.LoggingExample","thread_name":"http-nio-8080-exec-1","level":"ERROR","level_value":40000}
```
{: screen}

可进一步定制 [`logstash-logback-encoder`](https://github.com/logstash/logstash-logback-encoder){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标") 以更改捕获和转发 JSON 日志的方式。

使用 `jq` 来过滤 JSON 日志以易于阅读。例如：

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

## 后续步骤
{: #spring-logging-next-steps notoc}

有关使用附加器、日志级别和配置详细信息来定制日志消息的更多信息，请参阅官方的 [Spring Boot 日志记录参考指南](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-logging.html){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")。

了解有关查看每个部署环境中日志的详细信息：

* [Kubernetes 日志](https://kubernetes.io/docs/concepts/cluster-administration/logging/){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")
* [{{site.data.keyword.openwhisk}} 日志和监视](/docs/openwhisk?topic=cloud-functions-openwhisk_logs#openwhisk_logs)
* [{{site.data.keyword.cloud_notm}} Log Analysis](/docs/services/CloudLogAnalysis?topic=cloudloganalysis-log_analysis_ov#log_analysis_ov)
* [{{site.data.keyword.cloud_notm}} Private ELK 堆栈](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_2.1.0.2/manage_metrics/logging_elk.html){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")
