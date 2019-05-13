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

# Spring によるヘルス・チェック
{: #spring-healthcheck}

Spring Boot Actuator によって提供されるヘルス・エンドポイントは、アプリケーションのライフサイクルと結び付いています。アプリケーションが開始するまで、エンドポイントに到達することはできません。また、アプリケーションが停止すると (これはプロセスがシャットダウンする前に生じます)、エンドポイントは使用不可になります。 Spring Boot Actuator は、クラスパスで検出されたテクノロジーのために、追加のヘルス・インディケーターを自動的に追加します。 例えば、アプリケーションで JDBC を使用する場合、ヘルス・エンドポイントには、バッキング・データ・ストアにアクセスできることを確認するためのいくつかの基本的なテストが含まれています。 この理由により、Actuator が定義するヘルス・エンドポイントは、readiness プローブとして使用するためにより適しています。

Spring Boot Actuator を使用可能にするには、以下のように `spring-boot-actuator` 依存関係を `pom.xml` ファイルに追加します。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-actuator</artifactId>
</dependency>
```
{: codeblock}

Spring Boot Actuator のバージョン間には、いくつかの動作の相違があります。 以下の 2 つのセクションを使用して、Spring Boot の両方のバージョンで、liveness と readiness のチェックを作成する方法を検討します。まず最新版である 2 から始めます。

## Spring Boot 2 でのヘルス・チェック
{: #spring-health-boot2}

Spring Boot 2 Actuator は `/actuator/health endpoint` を定義します。これは、すべてが正常であれば `{"status": "UP"}` ペイロードを返します。 このエンドポイントはデフォルトで有効になっていて、アプリケーション・コードを必要としません。

以下は、Spring Boot 2 Actuator を使用した非認証の成功したヘルス・チェックの例です。
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

示されているように、ヘルス・エンドポイントはデフォルトで単純な UP または DOWN の状況を返します。 自動化されたヘルス・チェックのためには、この単純なペイロードで通常は十分です。 詳細なヘルス情報は、以下のプロパティーを `application.properties` に設定することにより使用可能になります。

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

いくつかの H2 データベース情報が含まれていることに注目してください。 この例では、Spring Actuator がバッキング・サービスのためのチェックを自動的に追加しています。 このケースでは、アプリケーションは JDBC を使用し、クラスパス上に H2 ドライバーが検出されました。

[Spring Boot Reference Guide for 2.1.x](https://docs.spring.io/spring-boot/docs/2.1.x/reference/html/production-ready-endpoints.html#production-ready-health){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン") で説明されているように、プロパティーまたはコードを使用して、ヘルス・エンドポイントのデフォルトの動作をオーバーライドできます。

### Spring Boot 2 での Liveness プローブ
{: #spring-liveness-boot2}

Spring Boot 2 の Actuator フレームワークは、カスタム・エンドポイントによって拡張可能な独自の mini-REST-framework です。 つまり、標準装備のヘルス・インディケーターと同じ方法で管理される単純な liveness エンドポイントを追加できます。 カスタムの liveness エンドポイントは、以下のように簡単に宣言できます。

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

このカスタム・エンドポイントは、以下のように Web エンドポイントとして公開する必要があります。

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

### Kubernetes での Spring Boot 2 の readiness プローブと liveness プローブの構成
{: #spring-probe-config-2}

Kubernetes デプロイメントのマニフェストで、readiness および liveness プローブ構成をコンテナー定義に追加します (path および port の値に注目してください)。

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

再始動サイクルを避けるために、安全を見てサービスの初期化時間よりも十分に長い時間を `livenessProbe.initialDelaySeconds` の値として設定してください。 `readinessProbe.initialDelaySeconds` にはそれよりも短い値を使用することで、レディー状態になったサービスに直ちに要求をルーティングできます。

## Spring Boot 1 でのヘルス・チェック
{: #spring-health-boot1}

Spring Boot 1 アクチュエーターは、`/health` エンドポイントを定義します。これは、すべてが正常であれば、`{"status": "UP"}` ペイロードを返します。 エンドポイントは権限を必要としませんが、権限が構成されていて呼び出し元に権限が付与されている場合は、追加情報が応答に含められます。

以下は、Spring Boot 1 Actuator を使用した非認証の成功したヘルス・チェックの例です。

```
$ curl -i localhost:8080/health
HTTP/1.1 200
X-Application-Context: application
Content-Type: application/vnd.spring-boot.actuator.v1+json;charset=UTF-8
Content-Length: 15

{"status":"UP"}
```
{: screen}

成功した認証ヘルス・チェックの例:

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

[Spring Boot Reference v1.5.x](https://docs.spring.io/spring-boot/docs/1.5.9.RELEASE/reference/htmlsingle/#_auto_configured_healthindicators){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン") で説明されているように、プロパティーまたはコードを使用して、ヘルス・エンドポイントのデフォルトの動作をオーバーライドできます。

### Spring Boot 1 での Liveness プローブ
{: #spring-liveness-boot1}

Spring Boot 1 の場合、常に {"status": "UP"} を状況コード 200 で返す liveness のための単純な http エンドポイントは、以下のようになります。

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

### Kubernetes での Spring Boot 1 の readiness プローブと liveness プローブの構成
{: #spring-probe-config-1}

Kubernetes デプロイメントのマニフェストで、readiness および liveness プローブ構成をコンテナー定義に追加します (path および port の値に注目してください)。

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

再始動サイクルを避けるために、安全を見てサービスの初期化時間よりも十分に長い時間を `livenessProbe.initialDelaySeconds` の値として設定してください。 `readinessProbe.initialDelaySeconds` にはそれよりも短い値を使用することで、レディー状態になったサービスに直ちに要求をルーティングできます。
