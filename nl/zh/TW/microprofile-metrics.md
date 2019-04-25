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

# MicroProfile 的度量
{: #mp-metrics}

MicroProfile 提供「度量」功能，可讓您使用簡單的註釋將自訂度量值新增至應用程式。您只需將 `mpMetrics-1.1` 特性新增至 `server.xml`，就能啟用這個功能。請注意，如果您想要其他應用程式伺服器特定度量（例如，關於 JDBC 連線儲存區的詳細資料），您也可以選擇性地新增 `monitor-1.0` 特性。

匯入 `@Counted` 註釋以建立簡單的計數器：

```java
import org.eclipse.microprofile.metrics.annotation.Counted;
```
{: codeblock}

然後，使用 `@Counted` 註釋來建立簡單的計數器。這個範例會計算 `createPortfolio` 方法的呼叫次數：

```java
@POST
@Path("/{owner}")
@Produces("application/json")
@Counted(monotonic=true, name="portfolios", displayName="Stock Trader portfolios", description="Number of portfolios created in the Stock Trader applications")
@RolesAllowed({"StockTrader"})
public JsonObject createPortfolio(@PathParam("owner") String owner) throws SQLException {
```
{: codeblock}

為了建置此程式碼，我們需要將下列段落新增至我們的 Maven `pom.xml`：

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

有了這個，每次呼叫 createPortfolio JAX-RS 方法時，計數器就會增加。我們可以呼叫 `GET /metrics` URI，以查看 jvm（類別載入、資料堆和記憶體回收統計資料）和應用程式定義的度量。`GET /metrics/application` URI 只會傳回應用程式定義的度量。我們可以使用指派的埠（在本範例中是 32388），透過 curl CLI 來命中此 REST GET API：

```
Johns-MacBook-Pro-8:StockTrader jalcorn$ curl http://9.42.2.107:32388/metrics/application

# TYPE application:com_ibm_hybrid_cloud_sample_portfolio_portfolio_portfolios counter

# HELP application:com_ibm_hybrid_cloud_sample_portfolio_portfolio_portfolios Number of portfolios created in the Stock Trader applications

application:com_ibm_hybrid_cloud_sample_portfolio_portfolio_portfolios

Johns-MacBook-Pro-8:StockTrader jalcorn$
```
{: screen}

如您所見，它已算出我們已建立 2 個產品組合。注意事項如下：

- 這是記憶體內的計數器：如果已重新啟動 Pod，此值會重設為零；如果有多個抄本，則每一個抄本都有自己的（本地化）值。
- "# HELP" 文字是我們在 `@Counted` 註釋中指定為說明的文字。

您可以在 Web 瀏覽器中檢視這個 REST GET 端點的輸出：

![REST GET 端點 Web 瀏覽器](images/microprofile-metrics-image1.png "REST GET 端點 Web 瀏覽器"){: caption="圖 1. REST GET 端點 Web 瀏覽器" caption-side="bottom"}

請注意，依預設，`/metrics` 端點需要通過 https 和登入認證。Liberty 18.0.0.3 建立了下列段落，您可以放在 server.xml 中，指出您想要此端點容許 http 且未經鑑別：

```xml
<mpMetrics authentication="false"/>
```

這會使配置 Prometheus 提取器以找到此端點的作業更為輕鬆。
