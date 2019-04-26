---

copyright:
  years: 2019
lastupdated: "2019-04-04"

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

# 로깅
{: #mp-logging}

MicroProfile 애플리케이션에서 로깅에 대해 권장되는 방식은 Java의 JSR-47 로깅 표준입니다. 다음 가져오기로 시작하십시오.

```java
import java.util.logging.Level;
import java.util.logging.Logger;
```
{: codeblock}

두 번째로, 클래스 레벨에서 로거의 인스턴스를 인스턴스화하십시오.

```java
private static Logger logger = Logger.getLogger(PortfolioService.class.getName());
```
{: codeblock}

코드에서 오퍼레이션 전후로, 그리고 오퍼레이션 내에서 `logger` 인스턴스에 대한 호출을 추가하십시오. `Logger` 인터페이스의 메소드는 로그되는 정보의 중요도 또는 "레벨"을 표시하도록 자체 이름 지정됩니다.

```java
logger.info("Creating portfolio for "+owner);
logger.warning("Unable to send message to JMS provider. Continuing without notification of change in loyalty level.");
```
{: codeblock}

이러한 메시지가 콘솔에 출력될 때 로그 레벨이 표시됩니다.

```
[INFO] Creating portfolio for John

[WARNING] Unable to send message to JMS provider. Continuing without notification of change in loyalty level.
```
{: screen}

로그 레벨은 애플리케이션이 작성할 로그를 동적으로 선택할 수 있는 유연성을 제공합니다. 이렇게 하면 상위 레벨 애플리케이션 상태와 자세한 디버그 컨텐츠를 미리 모두 설명하는 로그 코드를 작성할 수 있지만 필요할 때까지 자세한 디버그 컨텐츠를 필터링할 수 있습니다. 로그 레벨 `info`는 일반적으로 최소 출력 레벨이며, 그 다음에 `fine`, `finer`, `finest` 및 `debug`가 옵니다.

로그 항목에 여러 코드 행이 필요하거나 이 로그 항목이 문자열 연결과 같은 비용이 많이 드는 오퍼레이션과 관련된 경우, 테스트로 확인하여 로그 레벨이 사용 가능한지 판별해야 합니다. 이렇게 하면 애플리케이션이 결국 필터링으로 제외되는 로그 메시지를 빌드하는 데 중요한 시간을 보내지 않게 됩니다. 다음 예에서는 메시지 출력을 빌드하기 전에 `fine`이라는 의도한 로그 레벨이 사용 가능한지 확인합니다.

```java
if (logger.isLoggable(Level.FINE)) {
    StringWriter writer = new StringWriter();
    exception.printStackTrace(new PrintWriter(writer));
    logger.fine(writer.toString());
}
```
{: codeblock}

로그 레벨 및 구성 세부사항에 대한 자세한 정보는 [WebSphere Liberty 문제점 해결 안내서](https://www.ibm.com/support/knowledgecenter/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/rwlp_logging.html){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘") 및 [java.util.logging API 문서](https://docs.oracle.com/javase/8/docs/api/java/util/logging/package-summary.html){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")를 참조하십시오.

## Liberty로 JSON 로깅
{: #mp-json-logging}

Liberty는 JSON 형식 로깅을 지원합니다. 이를 사용하면 로그 메시지가 콘솔에 JSON 형식으로 기록됩니다. `server.xml`에서 다음 로깅 스탠자를 사용하여 이를 사용으로 설정하십시오.

```xml
<logging consoleLogLevel="INFO" consoleFormat="json" consoleSource="message,trace,accessLog,ffdc" />
```
{: codeblock}

`accessLog`가 위의 콘솔 소스 목록에 포함되어 있을 때 이러한 로그가 콘솔에 기록되기 전에 HTTP 액세스 로깅이 사용 가능해야 합니다. 다음 스니펫에는 `server.xml`의 `httpEndpoint` 요소에 `accessLogging` 하위 요소를 추가하는 방법이 표시되어 있습니다.

```xml
<httpEndpoint id="defaultHttpEndpoint" host="\*" httpPort="9080" httpsPort="9443">
  <accessLogging
    filepath="${server.output.dir}/logs/http_defaultEndpoint_access.log"
    logFormat='%h %u %t "%r" %s %b %D %{User-agent}i'>
  </accessLogging>
</httpEndpoint>
```
{: codeblock}

애플리케이션에 이 코드를 추가하면

```java
if (logger.isLoggable(Level.AUDIT)) {
    logger.audit("Initialization complete");
}
```

로그에 다음과 유사한 내용이 표시됩니다.

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

### JSON 로그 출력 읽기
{: #mp-json-log-output}

전체 JSON 출력은 로그 스토리지 및 검색에 매우 유용하지만 읽기에 쉽지는 않습니다. `kubectl`을 사용하여 터미널 창에서 로그의 컨텐츠를 검사해야 합니다. 다행히도 `jq`라는 명령행 도구가 있어 도움이 됩니다.

`jq`를 사용하면 필요한 필드를 필터링하여 집중할 수 있습니다. 예를 들어, `message` 필드만 표시하고 나머지는 모두 필터링하여 제외하려면 다음을 수행하십시오.

```
kubectl logs trader-54b4d579f7-4zvzk -n stock-trader -c trader | grep message | jq .message -r
```
{: pre}

Liberty에는 JSON 형식이 아닌 몇 가지 기본 콘솔 메시지가 있습니다. `grep`을 사용하면 `jq`로 메시지 필드가 포함된 행만 구문 분석할 수 있습니다.

## 추가 기능
{: #mp-log-features}

`logger.info` 또는 `logger.fine`과 같은 로그 레벨 사용에 대한 가이드라인은 각 조직 또는 프로젝트가 의사결정을 내려야 하는 내용입니다. 하지만 이러한 인터페이스는 거의 모든 프로젝트에 필요하며 유용합니다.

가장 좋은 방법은 `server.xml`의 각 관련 필드에 환경 변수(Kube 구성 맵 또는 시크릿을 통해 팟(Pod)에 피드됨)를 사용하는 것입니다. 이를 통해 Docker 이미지를 다시 빌드하여 재배치하지 않고도 로깅 구성을 변경할 수 있습니다.

예를 들어, 세분화된 로깅 속성을 설정하는 데 환경 변수를 사용하기 위해 이전 예의 스탠자를 변경합니다.

```xml
<logging consoleLogLevel="INFO" consoleFormat="json" consoleSource="message,trace,accessLog,ffdc" />
```
{: codeblock}

다음과 같이 변경합니다.

```xml
<logging consoleLogLevel="${env.LOG_LEVEL}" consoleFormat="${env.LOG_FORMAT}" consoleSource="${env.LOG_SOURCE}" />
```
{: codeblock}

다른 대안은 [로깅 및 추적 문서 ](https://www.ibm.com/support/knowledgecenter/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/rwlp_logging.html){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")에 설명된 대로 `WLP_LOGGING_CONSOLE_FORMAT` 환경 변수를 사용하는 것입니다. 이는 위의 예제와 유사합니다. `WLP_LOGGING_CONSOLE_FORMAT` 변수를 `basic`(기본값) 또는 `json`으로 설정할 수 있습니다.

## Liberty를 위한 Kibana 대시보드
{: #liberty-kibana}

새 JSON 로깅 기능과 함께 Liberty는 [GitHub에서 다운로드할 수 있는](https://www.ibm.com/support/knowledgecenter/en/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/twlp_icp_json_logging.html){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘") 사전 빌드된 Kibana 대시보드를 제공합니다. 링크의 지시사항에 따라 이 대시보드를 설치하십시오. 두 개의 새 대시보드를 사용할 수 있습니다.

![Kibana 대시보드](images/microprofile-logging-image4.png "Kibana 대시보드")

문제점 판별을 위해 대시보드를 클릭하면 다음을 확인할 수 있습니다.

![Kibana 대시보드 세부사항](images/microprofile-logging-image5.png "Kibana 대시보드 세부사항")

대시보드는 대화식입니다. 예를 들어, **Liberty 메시지** 위젯에 대한 범례에서 **INFO**를 클릭하면 다음 **Liberty 메시지 검색** 위젯이 `loglevel=INFO` 메시지에 대해 자체 필터링합니다. 대시보드는 모든 Liberty 기반 마이크로서비스의 로그 데이터를 연합하여 다른 시스템 로그를 필터링합니다.

Liberty helm 차트와 연관된 추가 Kibana 및 Grafana 대시보드가 있습니다. 이러한 대시보드는 [Liberty 클라우드 pak 확장](https://github.com/IBM/charts/tree/master/stable/ibm-websphere-liberty/ibm_cloud_pak/pak_extensions/dashboards){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")으로 사용 가능합니다.

## 다음 단계
{: #mp-logging-next-steps notoc}

어펜더, 로그 레벨 및 구성 세부사항으로 로그 메시지를 사용자 정의하는 데 대한 자세한 정보는 공식 [로깅을 위한 Spring Boot 참조](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-logging.html){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")를 참조하십시오.

각 배치 환경에서 로그를 보는 방법에 대해 자세히 알아보십시오.

* [Kubernetes 로그](https://kubernetes.io/docs/concepts/cluster-administration/logging/){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")
* [{{site.data.keyword.openwhisk}} 로그 및 모니터링](/docs/openwhisk?topic=cloud-functions-openwhisk_logs#openwhisk_logs)
* [{{site.data.keyword.cloud_notm}} Log Analysis](/docs/services/CloudLogAnalysis?topic=cloudloganalysis-log_analysis_ov#log_analysis_ov)
* [{{site.data.keyword.cloud_notm}} Private ELK 스택](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_2.1.0.2/manage_metrics/logging_elk.html){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")
