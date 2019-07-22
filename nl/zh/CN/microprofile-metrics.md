---

copyright:
  years: 2019
lastupdated: "2019-06-10"

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

# MicroProfile 的度量值
{: #mp-metrics}

MicroProfile 提供了“度量值”功能，该功能支持您使用简单注释将定制度量值添加到应用程序。要启用此功能，您只需将 `mpMetrics-1.1` 功能添加到 `server.xml` 中即可。如果要查看更多特定于应用程序服务器的度量值（例如，有关 JDBC 连接池的度量值），您可以选择添加 `monitor-1.0` 功能。

导入 `@Counted` 注释以创建简单计数器：

```java
import org.eclipse.microprofile.metrics.annotation.Counted;
```
{: codeblock}

然后，使用 `@Counted` 注释来创建简单计数器，如以下示例所示，此示例将统计已调用 `createPortfoloio` 方法的次数： 

```java
@POST
@Path("/{owner}")
@Produces("application/json")
@Counted(monotonic=true, name="portfolios", displayName="Stock Trader portfolios", description="Number of portfolios created in the Stock Trader applications")
@RolesAllowed({"StockTrader"})
public JsonObject createPortfolio(@PathParam("owner") String owner) throws SQLException {
```
{: codeblock}

要构建此代码，请将以下节添加到 Maven 的 `pom.xml` 文件：

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

添加后，每次调用 `createPortfolio` JAX-RS 方法时，计数器都会递增。 

可以调用 `GET /metrics` URI 来查看 JVM（类装入、堆和垃圾回收统计信息）和应用程序定义的度量值。`GET /metrics/application` URI 会仅返回应用程序定义的度量值。 

可以使用分配的端口（在此示例中为 32388），通过 curl CLI 来访问 REST GET API：

```
Johns-MacBook-Pro-8:StockTrader jalcorn$ curl http://9.42.2.107:32388/metrics/application

# TYPE application:com_ibm_hybrid_cloud_sample_portfolio_portfolio_portfolios counter

# HELP application:com_ibm_hybrid_cloud_sample_portfolio_portfolio_portfolios Number of portfolios created in the Stock Trader applications

application:com_ibm_hybrid_cloud_sample_portfolio_portfolio_portfolios

Johns-MacBook-Pro-8:StockTrader jalcorn$
```
{: screen}

如您所见，已按预期对两个产品服务组合进行计数。 

需注意以下几点：
- 使用内存中的计数器：如果重新启动 pod，值将会重置为零；如果有多个副本，那么每个副本都有其自己的唯一值。
- “#HELP”文本是我们在 `@Counted` 注释中指定为描述的内容。

还可以在 Web 浏览器中查看此 REST GET 端点的输出：

![REST GET 端点 Web 浏览器](images/microprofile-metrics-image1.png "REST GET 端点 Web 浏览器")

缺省情况下，`/metrics` 端点需要传递 HTTPS 和登录凭证。Liberty 18.0.0.3 引入了以下节，您可以将其放入 `server.xml` 中，以将此端点定义为允许使用 HTTP，并且不对此端点进行认证：

```xml
<mpMetrics authentication="false"/>
```

简化了 Prometheus 提取器配置，以使端点更容易访问。
