---

copyright:
  years: 2019
lastupdated: "2019-03-27"

keywords: mpmetrics microprofile, mpmetrics, prometheus java, metrics java, microprofile metrics

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

# MicroProfile의 메트릭
{: #mp-metrics}

MicroProfile은 간단한 어노테이션을 사용하여 애플리케이션에 사용자 정의 메트릭을 추가할 수 있게 하는 메트릭 기능을 제공합니다. 이 기능을 사용으로 설정하려면 `mpMetrics-1.1` 기능을 `server.xml`에 추가하면 됩니다. 추가 앱 서버 특정 메트릭(예를 들어, JDBC 연결 풀에 대한)을 원하는 경우 선택적으로 `monitor-1.0` 기능을 추가할 수도 있습니다.

`@Counted` 어노테이션을 가져와 간단한 카운터를 작성하십시오.

```java
import org.eclipse.microprofile.metrics.annotation.Counted;
```
{: codeblock}

그런 다음 `@Counted` 어노테이션을 사용하여 간단한 카운터를 작성하십시오. 이 예제에서는 `createPortfolio` 메소드가 호출된 횟수를 계산합니다.

```java
@POST
@Path("/{owner}")
@Produces("application/json")
@Counted(monotonic=true, name="portfolios", displayName="Stock Trader portfolios", description="Number of portfolios created in the Stock Trader applications")
@RolesAllowed({"StockTrader"})
public JsonObject createPortfolio(@PathParam("owner") String owner) throws SQLException {
```
{: codeblock}

이 코드를 빌드하려면 다음 스탠자를 Maven `pom.xml`에 추가해야 합니다.

```xml
<dependency>
  <groupId>org.eclipse.microprofile</groupId>
  <artifactId>microprofile</artifactId>
  <version>2.0.1</version>
  <type>pom</type>
  <scope>provided</scope>
</dependency>
```
{: codeblock}

이와 함께 createPortfolio JAX-RS 메소드가 호출될 때마다 카운터가 증분됩니다. `GET /metrics` URI를 호출하여 JVM(클래스 로딩, 힙 및 가비지 콜렉션 통계)과 애플리케이션 정의 메트릭을 둘 다 볼 수 있습니다. `GET /metrics/application` URI는 애플리케이션 정의 메트릭만 리턴합니다. 지정된 포트(예제에서는 32388)를 사용하여 curl CLI를 통해 이 REST GET API를 실행할 수 있습니다.

```
Johns-MacBook-Pro-8:StockTrader jalcorn$ curl http://9.42.2.107:32388/metrics/application

# TYPE application:com_ibm_hybrid_cloud_sample_portfolio_portfolio_portfolios counter

# HELP application:com_ibm_hybrid_cloud_sample_portfolio_portfolio_portfolios Number of portfolios created in the Stock Trader applications

application:com_ibm_hybrid_cloud_sample_portfolio_portfolio_portfolios

Johns-MacBook-Pro-8:StockTrader jalcorn$
```
{: screen}

표시된 대로 2개의 포트폴리오를 작성했습니다. 다음은 몇 가지 참고사항입니다.

- 이는 인메모리 카운터입니다. 팟(Pod)이 다시 시작되면 값이 0으로 재설정됩니다. 복제본이 여러 개인 경우 각각 고유(자국어로 지원된) 값을 갖게 됩니다.
- "# HELP" 텍스트는 `@Counted` 어노테이션에 설명으로 지정한 내용입니다.

웹 브라우저에서도 이 REST GET 엔드포인트의 출력을 볼 수 있습니다.

![REST GET 엔드포인트 웹 브라우저](images/microprofile-metrics-image1.png "REST GET 엔드포인트 웹 브라우저")

기본적으로 `/metrics` 엔드포인트에서는 https 및 로그인 인증 정보를 전달해야 합니다. Liberty 18.0.0.3은 이 엔드포인트가 http를 허용하고 인증되지 않도록 하기 위해 server.xml에 입력할 수 있는 다음과 같은 스탠자를 도입했습니다.

```xml
<mpMetrics authentication="false"/>
```

이렇게 하면 Prometheus 스크레이퍼가 구성되어 이 엔드포인트를 훨씬 쉽게 실행하게 됩니다.
