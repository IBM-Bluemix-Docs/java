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

# Spring의 로깅
{: #spring-logging}

로그 메시지는 로그 항목이 작성된 시점의 마이크로서비스 상태 및 활동과 관련된 컨텍스트 정보의 문자열입니다. 로깅을 사용하여 서비스가 실패한 이유와 과정을 진단하고 애플리케이션 상태 모니터링에서 메트릭을 지원할 수 있습니다.

로그 항목은 명령행 도구를 사용하여 볼 수 있는 표준 출력 및 오류 스트림에 직접 기록됩니다. 그런 다음 Logstash, Filebeat 또는 Fluentd와 같은 인프라 레벨에서 구성된 로그 전달 서비스를 사용하여 로그 콜렉션을 관리할 수 있습니다.

## Spring 앱에 Logback 지원 추가
{: #spring-log4j}

[Logback](https://logback.qos.ch/){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")은 Spring Boot에서 사용되는 기본 로그 엔진이고 클래스 경로에 있으면 자동으로 사용됩니다. 대부분의 Spring Boot 스타터는 `spring-boot-starter-logging`을 통해 Logback에 이행적으로 종속됩니다.

다음 예([Spring Boot 샘플](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-samples/spring-boot-sample-logback/src/main/java/sample/logback/SampleLogbackApplication.java){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘") 기반)에서는 Logback이 구현하는 SLF4J 로깅 API를 사용하여 `Logger`를 초기화하고 다양한 로그 레벨에서 메시지를 생성합니다.

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

기본 로그 레벨은 `INFO`입니다.  `application.properties`에서 특정 Java&trade; 패키지에 대한 여러 로깅 레벨을 지정할 수 있습니다. 예를 들어, 기본 로그 레벨을 `WARN`으로, 패키지 `sample.logging`의 레벨을 `DEBUG`로, Spring 패키지 `org.springframework.web`의 레벨을 `ERROR`로 설정하려면 다음을 `application.properties`에 추가하십시오.

```properties
logging.level.root=WARN
logging.level.org.springframework.web=ERROR
logging.level. sample.logging=DEBUG
```
{: codeblock}

기본적으로 Logback은 다음 요소로 메시지를 생성합니다.

- 날짜.
- 시간(정밀도로 밀리초 사용).
- 로그 레벨: `ERROR`, `WARN`, `INFO`, `DEBUG` 또는 `TRACE`.
- 프로세스 ID.
- 실제 로그 메시지의 시작을 정의하는 "`---`" 구분 기호.
- [스레드 이름(콘솔 출력의 경우 잘릴 수 있음)]
- 로거 이름: 이름은 보통 소스 클래스 이름입니다(종종 축약됨).
- 로그 메시지.

로그 메시지는 다음 예와 유사합니다.

```
2018-12-12 09:59:34.751  INFO 11174 --- [           main] sample.logging.SampleLogbackApplication  : Sample INFO Message
2018-12-12 09:59:34.751  WARN 11174 --- [           main] sample.logging.SampleLogbackApplication  : Sample WARN Message
2018-12-12 09:59:34.752 ERROR 11174 --- [           main] sample.logging.SampleLogbackApplication  : Sample ERROR Message
```
{: screen}

### JSON 로그 작성
{: #spring-json-logs}

`logstash-logback-encoder`를 사용하여 Spring 앱에서 JSON 로깅을 사용으로 설정하십시오.

Logback 인코더를 종속성으로 추가하십시오.

```xml
<dependency>
  <groupId>net.logstash.logback</groupId>
  <artifactId>logstash-logback-encoder</artifactId>
  <version>4.7</version>
</dependency>
```
{: codeblock}

다음 예와 같이 `logback.xml`에서 `ConsoleAppender`와 함께 새 인코더를 사용하도록 Logback을 구성하십시오.

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

이 구성을 사용하면 다음과 같은 JSON 형식 로그가 표시됩니다.

```
{"@timestamp":"2018-10-11T23:48:57.215+00:00","@version":1,"message":"Sample TRACE Message","logger_name":"com.example.demo.LoggingExample","thread_name":"http-nio-8080-exec-1","level":"TRACE","level_value":5000}
{"@timestamp":"2018-10-11T23:48:57.216+00:00","@version":1,"message":"Sample DEBUG Message","logger_name":"com.example.demo.LoggingExample","thread_name":"http-nio-8080-exec-1","level":"DEBUG","level_value":10000}
{"@timestamp":"2018-10-11T23:48:57.216+00:00","@version":1,"message":"Sample INFO Message","logger_name":"com.example.demo.LoggingExample","thread_name":"http-nio-8080-exec-1","level":"INFO","level_value":20000}
{"@timestamp":"2018-10-11T23:48:57.217+00:00","@version":1,"message":"Sample WARN Message","logger_name":"com.example.demo.LoggingExample","thread_name":"http-nio-8080-exec-1","level":"WARN","level_value":30000}
{"@timestamp":"2018-10-11T23:48:57.217+00:00","@version":1,"message":"Sample ERROR Message","logger_name":"com.example.demo.LoggingExample","thread_name":"http-nio-8080-exec-1","level":"ERROR","level_value":40000}
```
{: screen}

[`logstash-logback-encoder`](https://github.com/logstash/logstash-logback-encoder){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")는 JSON 로그를 캡처하고 전달하는 방법을 변경하도록 사용자 정의할 수 있습니다.

더 쉽게 읽을 수 있도록 JSON 로그를 필터링하려면 `jq`를 사용하십시오. 예를 들어, 다음과 같습니다.

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

## 다음 단계
{: #spring-logging-next-steps notoc}

어펜더, 로그 레벨 및 구성 세부사항으로 로그 메시지를 사용자 정의하는 데 대한 자세한 정보는 공식 [로깅을 위한 Spring Boot 참조](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-logging.html){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")를 참조하십시오.

다음 각 배치 환경에서 로그를 보는 방법에 대해 자세히 알아보십시오.

* [Kubernetes 로그](https://kubernetes.io/docs/concepts/cluster-administration/logging/){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")
* [{{site.data.keyword.openwhisk}} 로그 및 모니터링](/docs/openwhisk?topic=cloud-functions-logs)
* [{{site.data.keyword.cloud_notm}} Log Analysis](/docs/services/CloudLogAnalysis?topic=cloudloganalysis-log_analysis_ov#log_analysis_ov)
* [{{site.data.keyword.cloud_notm}} Private ELK 스택](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_2.1.0.2/manage_metrics/logging_elk.html){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")
