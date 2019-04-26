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

# Spring でのロギング
{: #spring-logging}

ログ・メッセージは、ログ・エントリーが生成された時のマイクロサービスの状態やアクティビティーに関するコンテキスト情報を記録したストリングです。 ロギングは、サービスが失敗した状況や理由の診断に活用できます。アプリケーションの正常性をモニターするためのメトリックを補完する情報として利用することもできます。

ログ・エントリーは、標準出力やエラー・ストリームに直接書き込む必要があります。これにより、コマンド・ライン・ツールを使用してログ・エントリーを表示できるようになり、インフラストラクチャー・レベルで構成するログ転送サービス (Logstash、Filebeat、Fluentd など) によるログ収集の管理が可能になります。

## Spring アプリケーションへの Logback サポートの追加
{: #spring-log4j}

[Logback](https://logback.qos.ch/){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン") は、Spring Boot でデフォルトで使用されるログ・エンジンです。クラスパスで検出されると自動的に使用されます。ほとんどの Spring Boot スターターは、過渡的に `spring-boot-starter-logging` の Logback を使用します。

以下の例 ([Spring Boot のサンプル](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-samples/spring-boot-sample-logback/src/main/java/sample/logback/SampleLogbackApplication.java)に基づく例){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン") では、Logback に実装されている SLF4J ロギング API を使用して `Logger` を初期化し、さまざまなログ・レベルでメッセージを生成します。

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

デフォルトのログ・レベルは `INFO` です。 `application.properties` で、特定の Java パッケージ用に別のロギング・レベルを指定できます。 例えば、デフォルトのログ・レベルを `WARN` に、パッケージ `sample.logging` のレベルを `DEBUG` に、Spring パッケージ `org.springframework.web` のレベルを `ERROR` にそれぞれ設定する場合は、以下のコードを `application.properties` に追加します。

```properties
logging.level.root=WARN
logging.level.org.springframework.web=ERROR
logging.level. sample.logging=DEBUG
```
{: codeblock}

デフォルトでは、Logback で生成されるメッセージに以下の要素が含まれています。

- 日付
- 時刻 (ミリ秒の精度)
- ログ・レベル: `ERROR`、`WARN`、`INFO`、`DEBUG`、`TRACE`。
- プロセス ID
- "`---`" セパレーター (実際のログ・メッセージの開始点を定義)
- [スレッド名 (コンソール出力のために切り捨てられる場合もある)]
- ロガー名: 通常はソース・クラスの名前 (省略されることが多い)。
- ログ・メッセージ

ログ・メッセージの例を以下に示します。

```
2018-12-12 09:59:34.751  INFO 11174 --- [           main] sample.logging.SampleLogbackApplication  : Sample INFO Message
2018-12-12 09:59:34.751  WARN 11174 --- [           main] sample.logging.SampleLogbackApplication  : Sample WARN Message
2018-12-12 09:59:34.752 ERROR 11174 --- [           main] sample.logging.SampleLogbackApplication  : Sample ERROR Message
```
{: screen}

### JSON ログの作成
{: #spring-json-logs}

Spring アプリケーションで JSON ロギングを有効にするには、`logstash-logback-encoder` を使用します。

logback エンコーダーを依存項目として追加します。

```xml
<dependency>
  <groupId>net.logstash.logback</groupId>
  <artifactId>logstash-logback-encoder</artifactId>
  <version>4.7</version>
</dependency>
```
{: codeblock}

`logback.xml` で logback を構成し、その新しいエンコーダーと一緒に `ConsoleAppender` を使用するようにします。 以下のように記述します。

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

この構成では、以下のような JSON 形式のログが表示されます。

```
{"@timestamp":"2018-10-11T23:48:57.215+00:00","@version":1,"message":"Sample TRACE Message","logger_name":"com.example.demo.LoggingExample","thread_name":"http-nio-8080-exec-1","level":"TRACE","level_value":5000}
{"@timestamp":"2018-10-11T23:48:57.216+00:00","@version":1,"message":"Sample DEBUG Message","logger_name":"com.example.demo.LoggingExample","thread_name":"http-nio-8080-exec-1","level":"DEBUG","level_value":10000}
{"@timestamp":"2018-10-11T23:48:57.216+00:00","@version":1,"message":"Sample INFO Message","logger_name":"com.example.demo.LoggingExample","thread_name":"http-nio-8080-exec-1","level":"INFO","level_value":20000}
{"@timestamp":"2018-10-11T23:48:57.217+00:00","@version":1,"message":"Sample WARN Message","logger_name":"com.example.demo.LoggingExample","thread_name":"http-nio-8080-exec-1","level":"WARN","level_value":30000}
{"@timestamp":"2018-10-11T23:48:57.217+00:00","@version":1,"message":"Sample ERROR Message","logger_name":"com.example.demo.LoggingExample","thread_name":"http-nio-8080-exec-1","level":"ERROR","level_value":40000}
```
{: screen}

JSON ログを取り込んで転送する方法を変更するために、[`logstash-logback-encoder`](https://github.com/logstash/logstash-logback-encoder){: new_window}
![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン") をさらにカスタマイズすることもできます。

JSON ログをフィルターに掛けて読みやすくするには、`jq` を使用します。 例えば次のようにします。

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

## 次の手順
{: #spring-logging-next-steps notoc}

アペンダー、ログ・レベル、構成の詳細を指定してログ・メッセージをカスタマイズする方法について詳しくは、公式サイトの[Spring Boot 解説書のロギングに関する情報](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-logging.html){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン") を参照してください。

各デプロイメント環境でのログの表示方法について詳しくは、次の各ページを参照してください。

* [Kubernetes のログ](https://kubernetes.io/docs/concepts/cluster-administration/logging/){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン")
* [{{site.data.keyword.openwhisk}} のログおよびモニタリング](/docs/openwhisk?topic=cloud-functions-openwhisk_logs#openwhisk_logs)
* [{{site.data.keyword.cloud_notm}} Log Analysis](/docs/services/CloudLogAnalysis?topic=cloudloganalysis-log_analysis_ov#log_analysis_ov)
* [{{site.data.keyword.cloud_notm}} Private ELK スタック](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_2.1.0.2/manage_metrics/logging_elk.html){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン")
