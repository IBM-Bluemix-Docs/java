---

copyright:
  years: 2019
lastupdated: "2019-04-23"

keywords: spring metrics, configure metrics spring, micrometer spring, micrometer, spring boot 2, spring actuator, prometheus spring

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

# Spring의 메트릭
{: #spring-metrics}

Spring Framework 5부터 Spring의 메트릭은 Micrometer에서 처리됩니다. Micrometer는 "메트릭의 SLF4J"로 자체 설명되는 프레임워크입니다. 다양한 로깅 백엔드에 연결할 수 있는 로깅을 위한 벤더 중립적 API 역할을 하는 SLF4J와 같이 Micrometer는 코드를 도구화하고 측정한 다음 Prometheus, DataDog 또는 Influx/Telegraph와 같은 다양한 메트릭 집계기에 계속 해당 메트릭을 제공하는 데 사용할 벤더 중립적 API를 제공합니다.

Micrometer 프레임워크를 사용하면 Spring이 다양한 클라우드 고유 아키텍처로 통합될 수 있습니다. Prometheus 또는 Statsd에 대한 지원을 추가하는 것은 종속성을 수정하는 간단한 문제이며, 메트릭 콜렉터가 푸시 기반인 경우 `application.properties`에 대상 정보를 제공합니다. Spring 및 Micrometer는 애플리케이션의 클래스 경로에 있는 종속성을 기반으로 런타임 시 수행할 작업을 결정합니다.

## 메트릭 사용 설정
{: #spring-metrics-enabling}

기본 메트릭이 Spring Boot Actuator에서 제공되며, 여기에는 `pom.xml` 파일의 `spring-boot-starter-actuator` 종속성이 필요합니다. 

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-actuator</artifactId>
</dependency>
```
{: codeblock}

여전히 Spring Boot 1.5.x에 있는 경우, 메트릭(및 Actuator)은 약간 다르게 작동합니다. Micrometer가 Spring Boot 2.0의 Spring 메트릭에 대한 표준이 되었지만 Boot 1.5.x에 백포트를 사용할 수 있게 되어, 클라우드 고유 Spring Boot 애플리케이션에서 메트릭을 작성하고 수집하기 위한 일관된 방식이 가능하게 됩니다. 더 중요하게는 Micrometer가 Prometheus를 포함한 다양한 모니터링 시스템을 지원하며, 이는 클라우드 배치를 위한 Boot 1.5.x 메트릭 시스템보다 나은 선택이 됩니다.
{: note}

## Micrometer 메트릭
{: #spring-metrics-micrometer}

Micrometer는 해당 인터페이스 `MeterRegistry`를 통해 메트릭에 대한 대상 개념을 추상화합니다. `MeterRegistry`에서는 애플리케이션이 사용자 정의 카운터, 게이지 및 타이머를 확보하는 방법을 제공합니다. 여러 대상이 필요한 경우 Micrometer에서는 `CompositeMeterRegistry` 구현을 제공하므로, 메트릭의 실제 구성 대상에서 애플리케이션을 격리할 수 있습니다.

Spring Boot의 어느 버전에서든 Micrometer 기반 메트릭 엔드포인트가 사용되면 Spring Boot가 자동으로 `CompositeMeterRegistry`를 작성하며 애플리케이션을 `Bean`으로 사용할 수 있게 됩니다. 레지스트리는 클래스 경로에 있는 지원되는 `MeterRegistry` 구현으로 자동으로 채워집니다. 여러 모니터링 시스템을 지원하거나 이러한 시스템 사이를 전환하는 것은 pom.xml에서 종속성을 변경하는 간단한 문제입니다. 

Spring은 그 자체로 bean인 `MeterRegistryCustomizer`를 통해 `MeterRegistry` bean을 사용자 정의할 수 있도록 허용합니다. Spring에서 이와 같은 bean의 존재를 식별하게 되면 메트릭을 보고하기 전에 글로벌 `MeterRegistry`를 구성할 수 있습니다. 이는 종종 `MeterRegistry`, DataCenter 또는 EnvironmentType에 일반 태그를 추가하는 데 사용됩니다.

메트릭의 이름을 지정할 때 Spring 및 Micrometer에서는 카멜 케이스(camel case) 또는 `_`를 사용하거나 다른 방법을 사용하는 대신 단어를 `.`으로 분할하는 이름 지정 규칙을 따르도록 권장합니다. 이는 Micrometer가 구성된 대상에 해당 메트릭을 릴레이하고 대상의 요구사항 충족에 필요할 때 이름 변환을 수행하기 때문입니다.

## Spring Boot를 사용한 메트릭 구성
{: #spring-metrics-configuration}

최신 버전인 Spring Boot 2부터 다음 절을 통해 Micrometer로 Spring Boot Actuator 메트릭을 사용하여 메트릭을 수집하고 각 릴리스에서 Prometheus의 엔드포인트를 제공하는 방법을 설명합니다.

### Spring Boot 2에서 메트릭 구성
{: #spring-metrics-boot2}

Spring Boot 2 Actuator 메트릭 엔드포인트는 웹 액세스에 사용 가능하고 노출되어야 합니다. 기본적으로 엔드포인트는 사용 가능하지만 노출되지는 않습니다(자세한 정보는 [Spring 엔드포인트 문서](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-endpoints.html){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘") 참조).  

다음을 `application.properties`에 추가하여 메트릭 엔드포인트를 노출하십시오.

```properties
management.endpoints.web.exposure.include=health,metrics
```
{: codeblock}

`/actuator` 엔드포인트를 확인하여 메트릭 엔드포인트가 사용 가능한지 확인하십시오. 로컬에서 애플리케이션을 실행하고 `jq`를 사용하여 json 응답을 pretty print하면 다음과 같이 출력됩니다.

```
$ curl -s localhost:8080/actuator | jq .
{
  "_links": {
    "self": {
      "href": "http://localhost:8080/actuator",
      "templated": false
    },
    ...
    "metrics": {
      "href": "http://localhost:8080/actuator/metrics",
      "templated": false
    },
    "metrics-requiredMetricName": {
      "href": "http://localhost:8080/actuator/metrics/{requiredMetricName}",
      "templated": true
    },
    ...
  }
}
```
{: screen}

`/actuator/metrics` 엔드포인트에 액세스하면 다음과 비슷하게 보이는 JSON 형식의 사용 가능한 메트릭 목록을 표시합니다.

```
$ curl -s localhost:8080/actuator/metrics | jq .
{
  "names": [
    "jvm.memory.max",
    "jvm.threads.states",
    "process.files.max",
    "jvm.gc.memory.promoted",
    ...
  ]
}
```
{: screen}

개별 메트릭의 실제 값을 확인하려면 추가 요청이 필요합니다(값이 Actuator URL 목록에서 추정될 수 있기 때문에). JVM 스레드 상태를 검색하려면 예를 들어, 다음을 수행하십시오.

```
$ curl -s localhost:8080/actuator/metrics/jvm.threads.states | jq .
{
  "name": "jvm.threads.states",
  "description": "The current number of threads having TERMINATED state",
  "baseUnit": "threads",
  "measurements": [
    {
    "statistic": "VALUE",
    "value": 24
    }
  ],
  "availableTags": [
    {
      "tag": "state",
      "values": [
        "timed-waiting",
        "new",
        "runnable",
        "blocked",
        "waiting",
        "terminated"
      ]
    }
  ]
}
```
{: screen}

메트릭 Actuator의 동작을 사용자 정의하는 방법에 대한 자세한 정보는 [공식 문서](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-metrics.html){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")를 참조하십시오.

### Spring Boot 2에서 Prometheus 지원 사용
{: #spring-prometheus-boot2}

Prometheus에서는 애플리케이션이 Prometheus 서버가 메트릭을 확보하기 위해 읽을 엔드포인트를 호스팅해야 합니다. Spring의 이 엔드포인트 호스팅은 데이터를 제공하는 기능이 이미 있는 애플리케이션에 의존하며, 이는 Prometheus 엔드포인트가 Spring MVC, Spring WebFlux 또는 Jersey를 사용하는 Spring Boot 애플리케이션만 지원함을 의미합니다.

Prometheus 엔드포인트를 사용하려면 `pom.xml`에 다음 종속성을 추가하십시오.

```xml
<dependency>
  <groupId>io.micrometer</groupId>
  <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```
{: codeblock}

Spring은 종속성을 인식하고 PrometheusMeterRegistry를 글로벌 MeterRegistry Bean의 구성된 레지스트리 중 하나로 자동 구성합니다. 하지만 메트릭 엔드포인트의 경우와 같이, application.properties에 다음을 추가하여 웹 액세스를 위해 Prometheus 엔드포인트를 사용으로 설정하고 노출해야 합니다.

```properties
management.endpoints.web.exposure.include=health,metrics,prometheus
```
{: codeblock}

`/actuator` 엔드포인트를 확인하여 Prometheus 엔드포인트가 사용 가능한지 확인하십시오. 애플리케이션을 로컬로 실행할 때 출력은 다음과 유사합니다.

```
$ curl -s localhost:8080/actuator | jq .
{
  "_links": {
    "self": {
      "href": "http://localhost:8080/actuator",
      "templated": false
    },
    ...
    "prometheus": {
      "href": "http://localhost:8080/actuator/prometheus",
      "templated": false
    },
    ...
  }
}
```
{: screen}

`/actuator/prometheus` 엔드포인트는 Prometheus 스크레이프 형식으로 구성된 모든 메트릭을 생성합니다.

```
$ curl -s localhost:8080/actuator/prometheus
# HELP system_cpu_usage The "recent cpu usage" for the whole system
# TYPE system_cpu_usage gauge
system_cpu_usage 0.046231032221805926
...
```
{: screen}

서비스 정의에 다음 어노테이션을 추가하여 메트릭 엔드포인트에 대해 Prometheus 스크레이핑을 사용으로 설정하십시오.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: ...
  annotations:
    prometheus.io/scrape: 'true'
    prometheus.io/scheme: http
    prometheus.io/path: /actuator/prometheus
    prometheus.io/port: "8080"
  ...
```
{: codeblock}

### Spring Boot 1에서 메트릭 구성
{: #spring-metrics-boot1}

Spring Boot Actuator의 `/metrics` 엔드포인트는 Spring Boot 1에서 기본적으로 사용 가능하지만 “민감한” 것으로 간주되고 권한이 필요합니다. 일부 환경에서는 메트릭 엔드포인트를 보안하는 것이 올바른 답이 될 수 있지만 메트릭 엔드포인트에 대한 각 요청에 권한을 부여하면 Kubernetes로 자동화된 검사의 대기 시간이 너무 길어집니다. application.properties에 다음을 추가하여 권한 요구사항을 제거하도록 메트릭 엔드포인트의 민감한 속성을 사용 안함으로 설정하십시오.

```properties
endpoints.metrics.sensitive=false
```
{: codeblock}

이렇게 하면 기본 Spring Boot 1 메트릭 지원이 가능하며, 이를 통해 다음과 같은 출력이 생성됩니다.

```
$ curl -s localhost:8080/metrics | jq .
{
  "names": [
    "jvm.memory.max",
    "jvm.threads.states",
    "process.files.max",
    "jvm.gc.memory.promoted",
    ...
  ]
}
```
{: screen}

Spring Boot 1 메트릭 Actuator의 동작을 사용자 정의하는 방법에 대한 자세한 정보는 [공식 문서](https://docs.spring.io/spring-boot/docs/1.5.x/reference/html/production-ready-metrics.html){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")를 참조하십시오.

### Spring Boot 1에서 Prometheus 지원 사용
{: #spring-prometheus-boot1}

Prometheus 지원은 Spring Boot 1 Actuator 메트릭에 적용되지 않지만, Micrometer를 통해 이에 대한 지원을 추가할 수 있습니다.

[개요](#spring-metrics)에 언급된 대로, Micrometer는 다양한 미터 기본 요소 세트와 Prometheus 및 StatsD와 같은 모니터링 시스템에 대한 향상된 지원을 포함하고 있으므로 Spring Boot 1으로 백포트되었습니다. Spring Boot 1에서 Micrometer를 사용하는 것은 매우 간단합니다. 이 경우 특정 어댑터 라이브러리와 하나 이상의 레지스트리가 필요합니다. `pom.xml`에 다음을 추가하여 어댑터 라이브러리 및 Prometheus 레지스트리에 대한 종속성을 포함하십시오.

```xml
<properties>
  ...
  <micrometer.version>1.1.1</micrometer.version>
</properties>
<dependencies>
  <dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-spring-legacy</artifactId>
    <version>${micrometer.version}</version>
  </dependency>
  <dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
    <version>${micrometer.version}</version>
  </dependency>
</dependencies>
```
{: codeblock}

이러한 종속성은 `/prometheus` 메트릭 엔드포인트를 작성하고 노출합니다. 기본 `/metrics` 엔드포인트와 같이 `/prometheus` 엔드포인트는 기본적으로 민감한 것으로 간주됩니다. 권한 요구사항을 제거하려면 application.properties에 다음을 추가하십시오.

```properties
endpoints.prometheus.sensitive=false
```
{: codeblock}

`/prometheus` 엔드포인트에서 생성된 출력은 Spring Boot 2의 경우와 유사합니다.

```
$ curl -s localhost:8080/prometheus
# HELP jvm_memory_used_bytes The amount of used memory
# TYPE jvm_memory_used_bytes gauge
jvm_memory_used_bytes{area="heap",id="PS Survivor Space",} 2.2594152E7
...
```
{: screen}

서비스 정의에 다음 어노테이션을 추가하여 메트릭 엔드포인트에 대해 Prometheus 스크레이핑을 사용으로 설정하십시오.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: …
  annotations:
    prometheus.io/scrape: 'true'
    prometheus.io/scheme: http
    prometheus.io/path: /prometheus
    prometheus.io/port: "8080"
  ...
```
{: codeblock}

Spring Boot 1에서 Micrometer를 사용하는 방법에 대한 자세한 정보는 [https://micrometer.io/docs/ref/spring/1.5](https://micrometer.io/docs/ref/spring/1.5){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")를 참조하십시오.

## Micrometer의 타이밍 호출
{: #spring-metrics-timing}

`management.metrics.web.server.auto-time-requests` 특성이 `true`로 설정되면 Spring MVC, WebFlux 및 Jersey 서버는 각각 모든 요청에 대한 타이밍 정보를 자동으로 수집하는 Spring의 자동 구성을 제공합니다. 이는 기본값입니다.

```properties
management.metrics.web.server.auto-time-requests=true
```
{: codeblock}

Micrometer는 지원되는 메소드 호출에 대한 타이밍 정보를 수집하는 데 사용할 수 있는 `@Timed` 어노테이션을 지원합니다. 데이터 수집을 위해 컨테이너 지원이 필요하므로 이 어노테이션은 임의 메소드의 시간을 맞추는 데 사용할 수 없습니다. `@Timed` 어노테이션은 서블릿 컨테이너에서 지원되는 오퍼레이션(예: Spring MVC, Webflux 또는 Jersey 서비스 메소드)에 지원됩니다.

수집되는 타이밍 정보를 선별하려면 `management.metrics.web.server.auto-time-requests` 특성을 `false`로 설정하고, 개별 서비스 메소드 또는 제어기에서 `@Timed` 어노테이션을 사용하십시오.

```java
@Timed
@GetMapping("/myendpoint")
public String getSomething() { ... }
```
{: codeblock}

`management.metrics.web.server.auto-time-requests`를 `true`로 설정한 경우 `@Timed` 어노테이션을 사용하여 보고된 지정 메트릭에 태그를 추가할 수 있습니다. 예를 들어, 다음과 같습니다.

```java
@Timed(extraTags = { "mytagname", "mytagvalue"})
@GetMapping("/myendpoint")
public String getSomething() { ... }
```
{: codeblock}

## Spring 및 Micrometer의 사용자 정의 메트릭
{: #spring-metrics-custom}

애플리케이션 도메인 특정 측정을 수행하려면 사용자 정의 메트릭을 작성해야 합니다. Micrometer는 몇 가지 기본 `Meter` 유형을 정의합니다.

* `Counter`는 증가만 가능한 값을 추적합니다.
* `Gauge`는 미터가 공개(또는 조회)될 때 관찰된 값을 측정하고 리턴합니다.
* `Timer`는 이벤트 발행 횟수 및 이 이벤트의 누적 경과 시간을 추적합니다. 
* `DistributionSummary`는 `Timer`와 유사하며, 이벤트 분포를 추적하지만 시간을 측정하지는 않습니다.

새 미터는 `MeterRegistry`의 헬퍼 메소드 또는 미터의 유연한 빌더를 사용하여 작성됩니다. 

애플리케이션 코드로 작업하는 경우, `MeterRegistry`가 삽입된 후 `@PostConstruct` 메소드에서 또는 생성자(`MeterRegistry`를 매개변수로서 전달)에서 `Meter`를 작성하십시오.

예를 들어, `RestController` 내에서 `Counter`를 사용하기 위해 다음과 같이 수행할 수 있습니다.

```java
@RestController
public class MyController {
    
    private final Counter myCounter;
    
    public MyController(MeterRegistry meterRegistry) {
    
        // Create the counter using the helper method on the builder
        myCounter = meterRegistry.counter("my.counter", "mytagname", "mytagvalue");
        
    }
```
{: codeblock}

그런 다음 서비스 메소드 내에서 미터 값을 업데이트할 수 있습니다.

```java
@GetMapping("/")
public String getSomething() {
    myCounter.increment();
    ...
}
```
{: codeblock}


Spring에서 사용자 정의 메트릭 작성에 대해 자세히 보려면 다음을 참조하십시오.

* [Spring Boot Metrics](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-metrics.html){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")
* [Micrometer concepts](https://micrometer.io/docs/concepts){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")
