---

copyright:
  years: 2019
lastupdated: "2019-04-22"

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

MicroProfile 提供一種「度量」功能，可讓您使用簡單的註釋將自訂度量新增至應用程式。若要啟用這個功能，請將 `mpMetrics-1.1` 功能新增至 `server.xml`。如果您想要看到其他應用程式伺服器特定度量（例如，JDBC 連線儲存區的相關資訊），您可以選擇性地新增 `monitor-1.0` 功能。

匯入 `@Counted` 註釋以建立簡單的計數器：

```java
import org.eclipse.microprofile.metrics.annotation.Counted;
```
{: codeblock}

然後，使用 `@Counted` 註釋來建立簡單計數器，如下列範例所示，計算呼叫 `createPortfoloio` 方法的次數： 

```java
@POST
@Path("/{owner}")
@Produces("application/json")
@Counted(monotonic=true, name="portfolios", displayName="Stock Trader portfolios", description="Number of portfolios created in the Stock Trader applications")
@RolesAllowed({"StockTrader"})
public JsonObject createPortfolio(@PathParam("owner") String owner) throws SQLException {
```
{: codeblock}

若要建置此程式碼，請將下列段落新增至 Maven 的 `pom.xml` 檔：

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

有了這個，每次呼叫 `createPortfolio` JAX-RS 方法時，計數器就會增加。 

您可以呼叫 `GET /metrics` URI，以同時查看 JVM（類別載入、資料堆和記憶體回收統計資料）和應用程式定義的度量。`GET /metrics/application` URI 只會傳回應用程式定義的度量。 

您可以使用指派的埠（在本範例中是 32388），透過 curl CLI 來存取 REST GET API：

```
Johns-MacBook-Pro-8:StockTrader jalcorn$ curl http://9.42.2.107:32388/metrics/application

# TYPE application:com_ibm_hybrid_cloud_sample_portfolio_portfolio_portfolios counter

# HELP application:com_ibm_hybrid_cloud_sample_portfolio_portfolio_portfolios Number of portfolios created in the Stock Trader applications

application:com_ibm_hybrid_cloud_sample_portfolio_portfolio_portfolios

Johns-MacBook-Pro-8:StockTrader jalcorn$
```
{: screen}

如您所見，按預期將兩個組合列入計數。 

注意事項如下：
- 這是記憶體內的計數器：如果已重新啟動 Pod，此值會重設為零；如果有多個抄本，則每一個抄本都有自己的唯一值。
- "# HELP" 文字是在 `@Counted` 註釋中指定為說明的文字。

您可以在 Web 瀏覽器中檢視這個 REST GET 端點的輸出：

![REST GET 端點 Web 瀏覽器](images/microprofile-metrics-image1.png "REST GET 端點 Web 瀏覽器"){: caption="圖 1. REST GET 端點 Web 瀏覽器" caption-side="bottom"}

依預設，`/metrics` 端點需要通過 https 和登入認證。Liberty 18.0.0.3 引進了下列段落，您可以將其放在 `server.xml` 中，定義此端點以容許 http 且未經鑑別：

```xml
<mpMetrics authentication="false"/>
```

Prometheus 提取器配置已簡化，讓存取端點更加輕鬆。
