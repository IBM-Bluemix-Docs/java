---

copyright:
  years: 2019
lastupdated: "2019-03-27"

keywords: health check jax-rs, jax-rs endpoint, jax-rs status, readiness jax-rs, liveness jax-rs, microprofile health

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

# JAX-RS로 상태 검사
{: #jaxrs-healthcheck}

이전 절에서 설명된 대로 Kubernetes는 TCP 또는 HTTP 엔드포인트를 통한 네트워크 검사뿐만 아니라 명령 실행을 포함해 상태 프로브 통합을 위한 여러 방법을 제공합니다. Java 언어로 이러한 프로브를 구현할 수 있지만 MicroProfile Health 및 HTTP 프로브의 JAX-RS 구현이 여기에 자세히 설명됩니다.

## 준비 상태(readiness) JAX-RS 엔드포인트 정의
{: #mp-readiness}

최소 준비 상태 엔드포인트는 다음과 유사합니다.

```java
@Path("ready")
public class ReadinessEndpoint {
  @GET
  @Produces(MediaType.APPLICATION_JSON)
  public Response testReadiness() {
    if ( ! ready ) {
      return Response.status(503).entity("{\"status\":\"DOWN\"}").build();
    }
    return Response.ok("{\"status\":\"UP\"}").build();
  }
}
```
{: codeblock}

WebSphere Liberty에 배치된 애플리케이션의 이와 같은 엔드포인트는 추가 노력 없이 특정 레벨의 준비 상태 동작을 수행합니다. Liberty는 애플리케이션이 시작될 때까지 상태 엔드포인트에 대한 요청에 실패하며 해당 503 응답이 나타납니다. 이 기본 구현은 필수 기능을 포함하도록 확장되어야 합니다. 예를 들어, `@ApplicationScoped` CDI 리소스를 평가하여 종속성 인젝션 처리가 완료되었는지 확인하십시오. 프로브를 구현하면 이전 절에 언급된 대로 HTTP 프로브 유형을 구성하여 사용할 수 있습니다.

항상 결함 허용의 역할과 목적을 기억하십시오. 마이크로서비스는 다운스트림 서비스와의 통신에서 장애 및 중단을 처리합니다. 준비 상태 검사 내에서 실현 가능한 폴백이 없는 기능에 대한 검사만 포함하십시오.

## 활동 상태(liveness) JAX-RS 엔드포인트 정의
{: #jaxrs-liveness}

활동 상태 프로브는 실패 시 프로세스가 즉각적으로 종료되므로 검사 항목에 대해 신중해야 합니다. 간단한 활동 상태 엔드포인트는 다음과 유사합니다.

```java
@Path("liveness")
public class LivenessEndpoint {
    @GET
    @Produces(MediaType.APPLICATION_JSON)
    public Response testLiveness() {
      return Response.ok("{\"status\":\"UP\"}").build();
    }
}
```
{: codeblock}

활동 상태 프로브 구현은 복구 불가능한 프로세스 상태를 표시하는 프로세스 로컬 조건을 참조하도록 확장될 수 있습니다. 하지만 이러한 검사에 대한 문제는 이와 같은 처리가 수행될 수 있는 경우 서버가 다른 요청을 수행할 수 있을 정도로 충분히 양호하다는 사실에 있습니다. 이러한 이유로 "단순하게 유지"는 활동 상태 검사 구현에 쉽게 적용되어야 합니다. 일단 활동 상태 소스가 프로브 엔드포인트를 사용하도록 코딩되면, 마지막 장에 설명된 대로 컨테이너 오케스트레이션 소스에서 http 활동 상태 엔드포인트를 사용할 수 있습니다.

다시 시작 사이클을 방지하려면 활동 상태 검사에 대한 initialDelaySeconds 속성이 가장 긴 예상 서버 시작 시간보다 길어야 합니다. 일반적으로 시작하는 데 30초가 걸리는 자바 애플리케이션 서버의 경우 60초와 같은 더 큰 값을 선택하십시오.

## MicroProfile Health API 사용
{: #mp-health-api}

[MicroProfile Health API](https://www.ibm.com/support/knowledgecenter/en/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/twlp_microprofile_healthcheck.html){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")(mpHealth)는 상태 검사 구현을 단순화하기 위한 프레임워크를 정의합니다. mpHealth API의 버전 1.0은 단일 엔드포인트만 정의합니다. 다음 버전은 활동 상태를 위한 엔드포인트와 준비 상태를 위한 엔드포인트를 모두 제공합니다.

Liberty에서 MicroProfile Health API를 사용하려면 `mpHealth` 기능을 `server.xml`에 추가하십시오.

```xml
<featureManager>
   <feature>mpHealth-1.0</feature>
</featureManager>
```
{: codeblock}

`/health` 엔드포인트에 액세스하여 브라우저를 통해 애플리케이션의 상태를 검사할 수 있습니다. 코드가 모두가 정상이면 페이로드 `{"outcome": "UP"}`을 리턴합니다. 다음에서는 MicroProfile Health API를 사용하여 팟(Pod)이 양호한지 아니면 준비되었는지를 표시하는 방법을 보여줍니다. `call` 메소드를 사용하면 개발자가 팟(Pod)의 상태를 표시하는 검사 알고리즘을 제공할 수 있습니다. 메모리가 부족한 경우 등은 팟(Pod)의 상태에 대한 좋은 지표가 됩니다. 다운스트림 데이터베이스 연결 검사는 팟(Pod)의 준비 상태 상태에 대한 필요한 검사입니다. 특정 검사가 없는 경우 다음을 사용하여 팟(Pod)이 활성인지 아니면 준비 상태인지를 표시할 수 있습니다.

```java
import org.eclipse.microprofile.health.Health;
import org.eclipse.microprofile.health.HealthCheck;
import org.eclipse.microprofile.health.HealthCheckResponse;

import javax.enterprise.context.ApplicationScoped;

@Health
@ApplicationScoped
public class HealthResource implements HealthCheck {

    @Override
    public HealthCheckResponse call() {
        return HealthCheckResponse.named("service-a").withData("service-a", "ok").up().build();
    }
}
```
{: codeblock}

그런 다음 아래에 표시된 대로 Kubernetes에 `livenessProbe` 및 `readinessProbe`를 지정해야 합니다.
```yaml
livenessProbe:
  exec:
    command: ["sh", "-c", "curl -s http://localhost:9080/health| grep -q service-a"]
  initialDelaySeconds: 60
readinessProbe:
  exec:
    command: ["sh", "-c", "curl -s http://localhost:9080/health | grep -q service-a"]
  initialDelaySeconds: 40
```
{: codeblock}

앞에서 언급한 대로 여러 로직을 사용하여 준비 상태 및 활동 상태를 표시할 수 있습니다. MicroProfile Health를 사용하여 준비 상태 검사를 표시한 다음 준비 상태 검사를 위한 JAX-RS 엔드포인트를 작성하거나 그 반대로 수행할 수 있습니다. 다음 버전의 MicroProfile Health는 두 개의 엔드포인트를 제공합니다(하나는 활동 상태용, 다른 하나는 준비 상태용).

OpenLiberty에는 MicroProfile Health 엔드포인트에 사용자 정의 검사를 추가하는 방법에 대한 예를 제공하는 실무용 안내서가 있습니다(https://openliberty.io/guides/microprofile-health.html).

[MicroProfile 상태 검사 수행](https://www.ibm.com/support/knowledgecenter/en/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/twlp_microprofile_healthcheck.html){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")에 설명된 대로 MicroProfile 상태 엔드포인트의 기본 동작을 대체할 수 있습니다.
