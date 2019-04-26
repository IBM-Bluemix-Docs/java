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

# MicroProfile によるメトリック
{: #mp-metrics}

MicroProfile は、アプリケーションにカスタム・メトリックを追加するためのシンプルなアノテーションを使用できるメトリック機能を提供しています。ユーザーは `mpMetrics-1.1` フィーチャーを `server.xml` に追加するだけでこれを使用できます。アプリ・サーバー固有の追加のメトリック (JDBC 接続プールに関するものなど) を使用する場合は、オプションで `monitor-1.0` フィーチャーも追加できます。

単純なカウンターを作成するために、`@Counted` アノテーションをインポートします。

```java
import org.eclipse.microprofile.metrics.annotation.Counted;
```
{: codeblock}

次に、`@Counted` アノテーションを使用して、単純なカウンターを作成します。この例では、`createPortfolio` メソッドが何回呼び出されたかをカウントします。

```java
@POST
@Path("/{owner}")
@Produces("application/json")
@Counted(monotonic=true, name="portfolios", displayName="Stock Trader portfolios", description="Number of portfolios created in the Stock Trader applications")
@RolesAllowed({"StockTrader"})
public JsonObject createPortfolio(@PathParam("owner") String owner) throws SQLException {
```
{: codeblock}

このコードをビルドするには、Maven `pom.xml` に以下のスタンザを追加する必要があります。

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

こうすると、createPortfolio JAX-RS メソッドが呼び出されるたびにカウンターが増分します。`GET /metrics` URI を呼び出して、jvm (クラスロード、ヒープ、ガーベッジ・コレクションの統計) とアプリケーションで定義されたメトリックの両方を調べることができます。`GET /metrics/application` URI は、アプリケーションで定義されたメトリックのみを返します。この REST GET API は、割り当てられたポート (この例では 32388) を使用して curl CLI を介して利用できます。

```
Johns-MacBook-Pro-8:StockTrader jalcorn$ curl http://9.42.2.107:32388/metrics/application

# TYPE application:com_ibm_hybrid_cloud_sample_portfolio_portfolio_portfolios counter

# HELP application:com_ibm_hybrid_cloud_sample_portfolio_portfolio_portfolios Number of portfolios created in the Stock Trader applications

application:com_ibm_hybrid_cloud_sample_portfolio_portfolio_portfolios

Johns-MacBook-Pro-8:StockTrader jalcorn$
```
{: screen}

出力に示されているように、2 つのポートフォリオを作成したことがカウントされています。以下の点に注目してください。

- これはメモリー内カウンターです。ポッドが再起動されると値はゼロにリセットされます。また、複数のレプリカがある場合、それぞれが独自の (ローカライズされた) 値を持つことになります。
- "# HELP" テキストは、`@Counted` アノテーション内の記述として指定するものです。

同様に、この REST GET エンドポイントの出力を Web ブラウザーで表示できます。

![RESTGET エンドポイントを表示する Web ブラウザー](images/microprofile-metrics-image1.png "REST GET エンドポイントを表示する Web ブラウザー")

デフォルトで、`/metrics` エンドポイントでは https およびログイン資格情報が渡される必要があるので、注意してください。Liberty 18.0.0.3 では、このエンドポイントに http を許可し、認証しないよう指示するための、server.xml に投入できる以下のスタンザが導入されています。

```xml
<mpMetrics authentication="false"/>
```

これにより、このエンドポイントを検出するための Prometheus scraper の構成が格段に容易になっています。
