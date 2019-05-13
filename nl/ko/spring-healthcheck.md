---

copyright:
  years: 2018, 2019
lastupdated: "2019-04-22"

keywords: health check spring, spring health endpoint, spring-boot-actuator, liveness probe spring, readiness probe spring, spring kubernetes probe

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

# Spring의 상태 검사
{: #spring-healthcheck}

Spring Boot Actuator에서 제공된 상태 엔드포인트는 애플리케이션 라이프사이클에 연결됩니다. 애플리케이션이 시작되기 전까지는 접속할 수 없으며 애플리케이션이 중지하면 바로 사용할 수 없게 됩니다(프로세스가 종료되기 전에 발생). Spring Boot Actuator는 클래스 경로에서 발견된 기술에 대한 상태 지표를 자동으로 추가합니다. 예를 들어, 애플리케이션이 JDBC를 사용하는 경우 상태 엔드포인트에는 지원 데이터 저장소에 도달할 수 있는지 확인하는 몇 가지 기본 테스트가 포함됩니다. 이러한 이유로 Actuator 정의 상태 엔드포인트는 준비 상태(readiness) 프로브로 사용하기 적합합니다.

`pom.xml` 파일에 `spring-boot-actuator` 종속성을 추가하여 Spring Boot Actuator를 사용으로 설정하십시오.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-actuator</artifactId>
</dependency>
```
{: codeblock}

Spring Boot Actuator 버전 사이에는 약간의 동작 차이가 있습니다. 최신 버전인 2부터 다음 두 절을 통해 Spring Boot의 두 버전에서 활동 상태(liveness) 및 준비 상태 검사를 작성하는 방법을 알아봅니다.

## 상태 검사 Spring Boot 2
{: #spring-health-boot2}

Spring Boot 2 Actuator는 `/actuator/health endpoint`를 정의하고, 모두가 정상이면 `{"status": "UP"}` 페이로드를 리턴합니다. 이 엔드포인트는 기본적으로 사용 가능하며 애플리케이션 코드가 필요하지 않습니다.

Spring Boot 2 Actuator를 사용하는 인증되지 않은 성공적 검사의 예:
<!-- Spring Boot 2 test project: https://github.com/IBM/spring-alarm-application -->

```
$ curl -i localhost:8080/actuator/health
HTTP/1.1 200
Content-Type: application/vnd.spring-boot.actuator.v2+json;charset=UTF-8
Transfer-Encoding: chunked
Date: Fri, 07 Dec 2018 23:09:09 GMT

{"status":"UP"}
```
{: screen}

표시된 대로 상태 엔드포인트는 기본적으로 단순한 작동 또는 작동 중지 상태를 리턴합니다. 자동화된 상태 검사에서는 대개 이 단순한 페이로드로 충분합니다. 자세한 상태 정보는 `application.properties`에서 다음 특성을 설정하여 사용할 수 있습니다.

```properties
management.endpoint.health.show-details=always
```
{: codeblock}

```
$ curl -i localhost:8080/actuator/health
HTTP/1.1 200
Content-Type: application/vnd.spring-boot.actuator.v2+json;charset=UTF-8
Transfer-Encoding: chunked
Date: Fri, 07 Dec 2018 23:09:09 GMT

{"status":"UP","details":{"diskSpace":{"status":"UP","details":{"total":500068036608,"free":407611420672,"threshold":10485760}},"db":{"status":"UP","details":{"database":"H2","hello":1}}}}
```
{: screen}

일부 H2 데이터 기본 정보 포함을 참고하십시오. 이는 지원 서비스에 대한 검사를 자동으로 추가하는 Spring Actuator의 예입니다. 이 경우 애플리케이션은 JDBC를 사용 중이고 H2 드라이버가 클래스 경로에서 발견되었습니다.

[2.1.x의 Spring Boot 참조 안내서](https://docs.spring.io/spring-boot/docs/2.1.x/reference/html/production-ready-endpoints.html#production-ready-health){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")에 설명된 대로 상태 엔드포인트의 기본 동작을 특성 또는 코드로 대체할 수 있습니다.

### Spring Boot 2의 활동 상태 프로브
{: #spring-liveness-boot2}

Spring Boot 2의 Actuator 프레임워크는 사용자 정의 엔드포인트로 확장할 수 있는 고유 미니 REST 프레임워크입니다. 즉, 기본 제공 상태 표시기와 같은 방식으로 관리되는 단순한 활동 상태 엔드포인트를 추가할 수 있습니다. 사용자 정의 활동 상태 엔드포인트는 다음과 같이 간단하게 선언할 수 있습니다.

```java
@Endpoint(id="liveness")
@Component
public class Liveness {
    @ReadOperation
    public String testLiveness() {
            return "{\"status\":\"UP\"}";
    }
}
```
{: codeblock}

이 사용자 정의 엔드포인트는 웹 엔드포인트로 노출해야 합니다.

```properties
management.endpoints.web.exposure.include=health,liveness,...
```
{: codeblock}

```
$ curl -i localhost:8080/actuator/liveness
HTTP/1.1 200
Content-Type: application/vnd.spring-boot.actuator.v2+json;charset=UTF-8
Content-Length: 15
Date: Thu, 07 Feb 2019 22:38:27 GMT

{"status":"UP"}
```
{: screen}

### Kubernetes에서 Spring Boot 2 준비 상태 및 활동 상태 프로브 구성
{: #spring-probe-config-2}

준비 상태 및 활동 상태 프로브 구성을 Kubernetes 배치 Manifest의 컨테이너 정의에 추가하십시오(경로 및 포트 값 참고).

```yaml
spec:
  containers:
  - name: ...
    image: ...
    readinessProbe:
      httpGet:
        path: /actuator/health
        port: 8080
      initialDelaySeconds: 60
      timeoutSeconds: 5
    livenessProbe:
      httpGet:
        path: /actuator/liveness
        port: 8080
      initialDelaySeconds: 130
      timeoutSeconds: 10
      failureThreshold: 10
```
{: codeblock}

주기를 다시 시작하지 않으려면 서비스를 초기화하는 데 소요되는 시간보다 길게 `livenessProbe.initialDelaySeconds`를 설정하십시오. 그런 다음 준비되는 즉시 `readinessProbe.initialDelaySeconds`에 대한 더 짧은 값을 사용하여 요청을 서비스로 라우트할 수 있습니다.

## Spring Boot 1의 상태 검사
{: #spring-health-boot1}

Spring Boot 1 Actuator는 `/health` 엔드포인트를 정의하고, 모두가 정상이면 `{"status": "UP"}` 페이로드를 리턴합니다. 엔드포인트에는 권한이 필요 없지만 권한이 구성되고 호출자에게 권한이 부여되면 추가 정보가 응답에 포함됩니다.

Spring Boot 1 Actuator를 사용하는 인증되지 않은 성공적 검사의 예:

```
$ curl -i localhost:8080/health
HTTP/1.1 200
X-Application-Context: application
Content-Type: application/vnd.spring-boot.actuator.v1+json;charset=UTF-8
Content-Length: 15

{"status":"UP"}
```
{: screen}

인증된 성공 상태의 검사 예제:

```
$ curl -i localhost:8080/health
HTTP/1.1 200 OK
X-Application-Context: application
Content-Type: application/vnd.spring-boot.actuator.v1+json;charset=UTF-8
Content-Length: 221

{
    "status" : "UP",
  "diskSpace" : {
        "status" : "UP",
    "total" : 63251804160,
    "free" : 31316164608,
    "threshold" : 10485760
    },
  "db" : {
        "status" : "UP",
    "database" : "H2",
    "hello" : 1
    }
}
```
{: screen}

[v1.5.x의 Spring Boot 참조](https://docs.spring.io/spring-boot/docs/1.5.9.RELEASE/reference/htmlsingle/#_auto_configured_healthindicators){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")에 설명된 대로 상태 엔드포인트의 기본 동작을 특성 또는 코드로 대체할 수 있습니다.

### Spring Boot 1의 활동 상태 프로브
{: #spring-liveness-boot1}

Spring Boot 1의 경우, 상태 코드 200과 함께 {"status": "UP"}을 항상 리턴하는 활동 상태에 대한 단순 http 엔드포인트는 다음과 유사합니다.

```java
@RestController
public class Liveness {
    private static Map<String, String> alwaysOk = Collections.singletonMap("status", "OK");

    @GetMapping("/liveness")
    public Map<String,String> testLiveness() {
        return alwaysOk;
    }
}
```
{: codeblock}

```
$ curl -i localhost:8080/liveness
HTTP/1.1 200
Content-Type: application/json;charset=UTF-8
Transfer-Encoding: chunked
Date: Thu, 07 Feb 2019 22:43:32 GMT

{"status":"OK"}
```
{: screen}

### Kubernetes에서 Spring Boot 1 준비 상태 및 활동 상태 프로브 구성
{: #spring-probe-config-1}

준비 상태 및 활동 상태 프로브 구성을 Kubernetes 배치 Manifest의 컨테이너 정의에 추가하십시오(경로 및 포트 값 참고).

```yaml
spec:
  containers:
  - name: ...
    image: ...
    readinessProbe:
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 60
      timeoutSeconds: 5
    livenessProbe:
      httpGet:
        path: /liveness
        port: 8080
      initialDelaySeconds: 130
      timeoutSeconds: 10
      failureThreshold: 10
```
{: codeblock}

주기를 다시 시작하지 않으려면 서비스를 초기화하는 데 소요되는 시간보다 길게 `livenessProbe.initialDelaySeconds`를 설정하십시오. 그런 다음 준비되는 즉시 `readinessProbe.initialDelaySeconds`에 대한 더 짧은 값을 사용하여 요청을 서비스로 라우트할 수 있습니다.
