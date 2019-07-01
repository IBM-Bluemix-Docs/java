---

copyright:
  years: 2019
lastupdated: "2019-06-10"

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

# Spring によるメトリック
{: #spring-metrics}

Spring Framework 5 以降では、Spring のメトリックが Micrometer によって扱われます。 Micrometer は、それ自体を「SLF4J for Metrics」として示すフレームワークです。 さまざまなロギング・バックエンドに接続されるベンダー非依存のロギング用 API として SLF4J が機能するのと同様に、Micrometer にも、コードを計測して測定できるベンダー非依存の API が用意されています。この API を使用して、それらのメトリックを Prometheus、DataDog、InfluxDB、Telegraf などのメトリック統合サービスに提供できます。

Micrometer フレームワークにより、Spring はさまざまなクラウド・ネイティブ・アーキテクチャーに統合できます。 Prometheus または Statsd のサポートの追加は、依存関係を変更するだけの、そしてメトリックのコレクターがプッシュ・ベースである場合は `application.properties` に宛先情報を指定するだけの、簡単な操作となります。 Spring および Micrometer は、アプリケーションのクラスパスで検出された依存関係に基づいて、実行時に行う内容を決定します。

## メトリックの有効化
{: #spring-metrics-enabling}

基本的なメトリックが Spring Boot Actuator によって提供されています。これには、以下のように、`pom.xml` ファイル内での `spring-boot-starter-actuator` 依存関係が必要となります。  

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-actuator</artifactId>
</dependency>
```
{: codeblock}

まだ Spring Boot 1.5.x を使用している場合、メトリック (およびアクチュエーター) の動作は少し異なるものとなります。 Spring Boot 2.0 で Micrometer が Spring メトリックの標準となりましたが、Boot 1.5.x ではバックポートが使用可能となり、メトリックを作成および収集するための一貫性のある動作が可能となりました。 さらに重要なこととして、Micrometer は Prometheus などのさまざまなモニター・システムをサポートするので、クラウド・デプロイメントでは Boot 1.5.x メトリックよりも優れた選択肢となります。
{: note}

## Micrometer メトリック
{: #spring-metrics-micrometer}

Micrometer は、そのインターフェース `MeterRegistry` で、メトリックの宛先の概念を要約しています。 `MeterRegistry` は、アプリがカスタム・カウンター、ゲージ、およびタイマーを入手する手段を提供します。 複数の宛先が必要な場合、Micrometer は `CompositeMeterRegistry` 実装を提供して、アプリを実際のメトリックの構成済みの宛先から分離できるようにします。

Spring Boot のどちらのバージョンでも、Micrometer ベースのメトリック・エンドポイントが有効な場合、Spring Boot は自動的に `CompositeMeterRegistry` を作成して、アプリがそれを `Bean` として使用できるようにします。 レジストリーには、クラスパスで検出された、サポート対象の `MeterRegistry` 実装が自動的に取り込まれます。 その後、複数のモニター・システムのサポートやそれらの間での切り替えは、pom.xml で依存関係を変更するだけの簡単な操作となります。

Spring では、それ自体が Bean である `MeterRegistryCustomizer` による `MeterRegistry` Bean のカスタマイズが可能です。 Spring がそのような Bean の存在を識別すると、メトリックが報告される前に、グローバル `MeterRegistry` を構成できます。 このカスタマイズは `MeterRegistry`、DataCenter、または EnvironmentType に共通タグを追加するためにしばしば使用されます。

メトリックに名前を付けるとき、キャメル・ケース、`_`、その他の方法を使用するよりも、Spring および Micrometer ではワードを `.` で分割する以下の命名規則を推奨します。 命名は重要です。Micrometer はこれらのメトリックを 1 つ以上の構成済み宛先に中継し、宛先の要件を満たすために必要に応じて名前の変換を行うからです。

## Spring Boot によるメトリックの構成
{: #spring-metrics-configuration}

以下のセクションでは、Micrometer で Spring Boot Actuator メトリックを使用できるようにする方法の概要を説明します。メトリックを収集し、Prometheus のエンドポイントを指定する方法を (リリースごとに) 示します。最新版である Spring Boot 2 から始めます。

### Spring Boot 2 でのメトリックの構成
{: #spring-metrics-boot2}

Spring Boot 2 Actuator のメトリック・エンドポイントは、有効になっていて Web アクセス用に公開されていることが必要です。 デフォルトで、エンドポイントは有効になっていますが、公開されていません (詳しくは、[Spring エンドポイントの資料](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-endpoints.html){: new_window}
![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン")を参照してください)。  

以下を `application.properties` に追加してメトリック・エンドポイントを公開します。

```properties
management.endpoints.web.exposure.include=health,metrics
```
{: codeblock}

`/actuator` エンドポイントを調べて、メトリック・エンドポイントが有効であることを確認します。 アプリをローカルで実行し、`jq` を使用して JSON 応答を整形した場合の出力は以下のようになります。

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

`/actuator/metrics` エンドポイントにアクセスすると、以下の出力のような、使用可能なメトリックの JSON 形式のリストが表示されます。
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

(アクチュエーター URL のリストから推測できる) 個別メトリックの実際の値を表示するには、別の要求が必要です。例えば、jvm スレッドの状態を取得するには、以下のようにします。

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

メトリック・アクチュエーターの動作をカスタマイズする方法について詳しくは、[公式の資料](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-metrics.html){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン") を参照してください。

### Spring Boot 2 での Prometheus サポートの有効化
{: #spring-prometheus-boot2}

Prometheus では、メトリックを取得するために Prometheus Server が読み取るためのエンドポイントをアプリがホストすることが必要となります。 Spring でのこのエンドポイントのホスティングは、データを提供するアプリに依存します。つまり、Prometheus エンドポイントは、Spring MVC、Spring WebFlux、または Jersey を使用する Spring Boot アプリをサポートします。

Prometheus エンドポイントを有効にするには、以下の依存関係を `pom.xml` に追加します。
```xml
<dependency>
  <groupId>io.micrometer</groupId>
  <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```
{: codeblock}

Spring は依存関係を認識し、グローバル `MeterRegistry` Bean 内の構成済みレジストリーの 1 つとして `PrometheusMeterRegistry` を自動的に構成します。ただし、メトリック・エンドポイントの場合と同様に、Prometheus エンドポイントは有効になっていて Web アクセス用に公開されていることが必要です。そのためには、以下を `application.properties` に追加します。

```properties
management.endpoints.web.exposure.include=health,metrics,prometheus
```
{: codeblock}

`/actuator` エンドポイントを調べて、Prometheus エンドポイントが有効であることを確認します。 アプリケーションをローカルで実行した場合、出力は以下の出力のようになります。

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

`/actuator/prometheus` エンドポイントは、構成済みのすべてのメトリックを Prometheus スクレープ・フォーマットで表示します。

```
$ curl -s localhost:8080/actuator/prometheus
# HELP system_cpu_usage The "recent cpu usage" for the whole system
# TYPE system_cpu_usage gauge
system_cpu_usage 0.046231032221805926
...
```
{: screen}

以下の注釈をサービス定義に追加して、メトリック・エンドポイントの Prometheus スクレープを有効にします。

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

### Spring Boot 1 でのメトリックの構成
{: #spring-metrics-boot1}

Spring Boot 1 では、Spring Boot Actuator の `/metrics` エンドポイントはデフォルトで有効になっていますが、「機密」と見なされ、許可が必要です。 一部の環境ではメトリック・エンドポイントを保護することが正解となることもありますが、メトリック・エンドポイントへの各要求を許可する処理は、Kubernetes での自動チェックの待ち時間を非常に長くします。

以下を `application.properties` に追加することにより、メトリック・エンドポイントの機密属性を無効にして、許可の要件を除去してください。
```properties
endpoints.metrics.sensitive=false
```
{: codeblock}

これで、デフォルトの Spring Boot 1 メトリック・サポートが有効になり、以下のような出力が生成されます。
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

Spring Boot 1 メトリック・アクチュエーターの動作をカスタマイズする方法について詳しくは、[公式の資料](https://docs.spring.io/spring-boot/docs/1.5.x/reference/html/production-ready-metrics.html){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン") を参照してください。

### Spring Boot 1 での Prometheus サポートの有効化
{: #spring-prometheus-boot1}

Prometheus サポートは Spring Boot 1 Actuator のメトリックに組み込まれていませんが、Micrometer を使用して追加できます。

[概説](#spring-metrics)に示されているように、Micrometer は、より豊富なメーター・プリミティブのセットを備え、Prometheus や StatsD などのモニター・システムをより良くサポートするので、Spring Boot 1 にバックポートされました。 Micrometer を Spring Boot 1 と共に使用するのは簡単です。そのためには、特定のアダプター・ライブラリーと 1 つ以上のレジストリーが必要です。以下を `pom.xml` に追加して、アダプター・ライブラリーと Prometheus レジストリーの依存関係を含めます。

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

これらの依存関係により、`/prometheus` メトリック・エンドポイントが作成されて公開されます。デフォルトの `/metrics` エンドポイントと同様に、`/prometheus` エンドポイントはデフォルトで機密と見なされます。 この許可要件を除去するには、以下を `application.properties` に追加します。

```properties
endpoints.prometheus.sensitive=false
```
{: codeblock}

`/prometheus` エンドポイントによって生成される出力は、Spring Boot 2 に似ています。

```
$ curl -s localhost:8080/prometheus
# HELP jvm_memory_used_bytes The amount of used memory
# TYPE jvm_memory_used_bytes gauge
jvm_memory_used_bytes{area="heap",id="PS Survivor Space",} 2.2594152E7
...
```
{: screen}

以下の注釈をサービス定義に追加して、メトリック・エンドポイントの Prometheus スクレープを有効にします。

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

Micrometer を Spring Boot 1 と共に使用する方法について詳しくは、[https://micrometer.io/docs/ref/spring/1.5](https://micrometer.io/docs/ref/spring/1.5){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン") を参照してください。

## Micrometer でのタイミング起動
{: #spring-metrics-timing}

Spring MVC、WebFlux、および Jersey Server はそれぞれ、プロパティー `management.metrics.web.server.auto-time-requests` が `true` に設定されているとき、すべての要求のタイミング情報を自動的に収集する Spring の自動構成を提供します。 デフォルト設定は `true` です。

```properties
management.metrics.web.server.auto-time-requests=true
```
{: codeblock}

また Micrometer は、サポート対象のメソッド呼び出しに関するタイミング情報の収集に使用できる、`@Timed` 注釈もサポートしています。 この注釈を使用して任意のメソッドの時間を計測することは、データの収集にコンテナー・サポートが必要となるため、できないことに留意してください。 `@Timed` 注釈は、Spring MVC、Webflux、Jersey サービス・メソッドなど、サーブレット・コンテナーによって支持される操作でサポートされています。

どのタイミング情報を収集するかについて、選択範囲を広げるためには、`management.metrics.web.server.auto-time-requests` プロパティーを `false` に設定し、以下のように `@Timed` 注釈を個別のサービス・メソッドまたはコントローラーに使用します。

```java
@Timed
@GetMapping("/myendpoint")
public String getSomething() { ... }
```
{: codeblock}

`management.metrics.web.server.auto-time-requests` が `true` の場合、`@Timed` 注釈を使用して、追加のタグを所定の報告されたメトリックに追加できます。以下に例を示します。

```java
@Timed(extraTags = { "mytagname", "mytagvalue"})
@GetMapping("/myendpoint")
public String getSomething() { ... }
```
{: codeblock}

## Spring および Micrometer によるカスタム・メトリック
{: #spring-metrics-custom}

アプリケーション・ドメインに特定の計測を行うには、独自のカスタム・メトリックを作成する必要があります。 Micrometer は、以下のように幾つかの基本的な `Meter` タイプを定義しています。

* `Counter` は、増加する値を追跡します。
* `Gauge` は、計量が公開 (または照会) されるときに監視された値を測定して返します。
* `Timer` は、イベントが発生した回数と、そのイベントの累積経過時間を追跡します。 
* `DistributionSummary` は `Timer` と似ていますが、イベントの分散も追跡し、時間は測定しません。

新しい Meter は、`MeterRegistry` のヘルパー・メソッド、または Meter の Fluent Builder を使用して作成します。 

`Meter` をコンストラクターに作成するか (`MeterRegistry` をパラメーターとして渡す)、`MeterRegistry` の注入後に `@PostConstruct` メソッドに作成します。

例えば、`Counter` を `RestController` で使用するには、以下のようにします。

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

その後、サービス・メソッド内で、計量値を更新できます。

```java
@GetMapping("/")
public String getSomething() {
    myCounter.increment();
    ...
}
```
{: codeblock}


Spring でのカスタム・メトリックの作成方法について詳しくは、以下を参照してください。

* [Spring Boot Metrics](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-metrics.html){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン")
* [Micrometer concepts](https://micrometer.io/docs/concepts){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン")
