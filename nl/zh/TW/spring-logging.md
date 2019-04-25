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

# 使用 Spring 進行記載
{: #spring-logging}

日誌訊息是一些字串，其中包含與建立日誌項目時微服務的狀態及活動相關的環境定義資訊。您可以使用記載來診斷服務失敗的方式及原因，以及協助度量值來監視應用程式性能。

日誌項目應該直接寫入標準輸出及錯誤串流。這可讓日誌項目利用指令行工具來進行檢視，並容許在基礎架構層次配置日誌轉遞服務（例如，Logstash、Filebeat 或 Fluentd），以管理日誌集合。

## 將 Logback 支援新增至 Spring 應用程式
{: #spring-log4j}

[Logback](https://logback.qos.ch/){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示") 是 Spring Boot 所使用的預設日誌引擎，在類別路徑中找到時便會自動使用。大部分 Spring Boot 入門範本會透過 `spring-boot-starter-logging` 過渡性地依賴 Logback。

下列範例（以 [Spring Boot 範例](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-samples/spring-boot-sample-logback/src/main/java/sample/logback/SampleLogbackApplication.java)為基礎）{: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示") 使用 Logback 實作的 SLF4J 記載 API，來起始設定 `Logger` 並在各個記載層次發出訊息：

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

預設記載層次為 `INFO`。您可以在 `application.properties` 中指定特定 Java 套件的不同記載層次。例如，若要將預設記載層次設為 `WARN`、將套件 `sample.logging` 的層次設為 `DEBUG`，並將 Spring 套件 `org.springframework.web` 的層次設為 `ERROR`，請新增下列內容至 `application.properties`：

```properties
logging.level.root=WARN
logging.level.org.springframework.web=ERROR
logging.level.sample.logging=DEBUG
```
{: codeblock}

依預設，Logback 會產生具有下列元素的訊息：

- 日期
- 具有毫秒精準度的時間
- 記載層次：`ERROR`、`WARN`、`INFO`、`DEBUG` 或 `TRACE`。
- 程序 ID。
- "`---`" 分隔字元，以定義實際日誌訊息的開始。
- [執行緒名稱（主控台輸出時可能截斷）]
- 日誌程式名稱：名稱通常是來源類別名稱（經常縮寫）。
- 日誌訊息。

日誌訊息類似下列範例：

```
2018-12-12 09:59:34.751  INFO 11174 --- [           main] sample.logging.SampleLogbackApplication  : Sample INFO Message
2018-12-12 09:59:34.751  WARN 11174 --- [           main] sample.logging.SampleLogbackApplication  : Sample WARN Message
2018-12-12 09:59:34.752 ERROR 11174 --- [           main] sample.logging.SampleLogbackApplication  : Sample ERROR Message
```
{: screen}

### 建立 JSON 日誌
{: #spring-json-logs}

請使用 `logstash-logback-encoder` 在 Spring 應用程式中啟用記載。

將 logback 編碼器新增為相依關係：

```xml
<dependency>
  <groupId>net.logstash.logback</groupId>
  <artifactId>logstash-logback-encoder</artifactId>
  <version>4.7</version>
</dependency>
```
{: codeblock}

請在 `logback.xml` 中用 `ConsoleAppender` 配置 logback，以以使用新的編碼器。它應該類似如下：

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

使用此配置，您現在將看到 JSON 格式日誌：

```
{"@timestamp":"2018-10-11T23:48:57.215+00:00","@version":1,"message":"Sample TRACE Message","logger_name":"com.example.demo.LoggingExample","thread_name":"http-nio-8080-exec-1","level":"TRACE","level_value":5000}
{"@timestamp":"2018-10-11T23:48:57.216+00:00","@version":1,"message":"Sample DEBUG Message","logger_name":"com.example.demo.LoggingExample","thread_name":"http-nio-8080-exec-1","level":"DEBUG","level_value":10000}
{"@timestamp":"2018-10-11T23:48:57.216+00:00","@version":1,"message":"Sample INFO Message","logger_name":"com.example.demo.LoggingExample","thread_name":"http-nio-8080-exec-1","level":"INFO","level_value":20000}
{"@timestamp":"2018-10-11T23:48:57.217+00:00","@version":1,"message":"Sample WARN Message","logger_name":"com.example.demo.LoggingExample","thread_name":"http-nio-8080-exec-1","level":"WARN","level_value":30000}
{"@timestamp":"2018-10-11T23:48:57.217+00:00","@version":1,"message":"Sample ERROR Message","logger_name":"com.example.demo.LoggingExample","thread_name":"http-nio-8080-exec-1","level":"ERROR","level_value":40000}
```
{: screen}

[`logstash-logback-encoder`](https://github.com/logstash/logstash-logback-encoder){: new_window}![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示") 可以進一步自訂，以變更如何擷取及轉遞 JSON 日誌。

請使用 `jq` 來過濾 JSON 日誌，以方便閱讀。例如：

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

## 後續步驟
{: #spring-logging-next-steps notoc}

如需使用附加器、記載層次及配置詳細資料來自訂日誌訊息的相關資訊，請參閱官方的 [Spring Boot 記載參考資料](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-logging.html){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")。

進一步瞭解在每一個部署環境中檢視日誌：

* [Kubernetes 日誌](https://kubernetes.io/docs/concepts/cluster-administration/logging/){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")
* [{{site.data.keyword.openwhisk}} Logs & Monitoring](/docs/openwhisk?topic=cloud-functions-openwhisk_logs#openwhisk_logs)
* [{{site.data.keyword.cloud_notm}} Log Analysis](/docs/services/CloudLogAnalysis?topic=cloudloganalysis-log_analysis_ov#log_analysis_ov)
* [{{site.data.keyword.cloud_notm}} Private ELK stack](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_2.1.0.2/manage_metrics/logging_elk.html){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")
