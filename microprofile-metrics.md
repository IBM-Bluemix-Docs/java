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

# Metrics with MicroProfile
{: #mp-metrics}

MicroProfile provides a Metrics capability that lets you use simple annotations to add custom metrics to your application. You just add the `mpMetrics-1.1` feature to your `server.xml` to enable this. Note you can optionally also add the `monitor-1.0` feature, if you'd like additional app server specific metrics (like about JDBC connection pools, for example).

Import the `@Counted` annotation to create a simple counter:

```java
import org.eclipse.microprofile.metrics.annotation.Counted;
```
{: codeblock}

Then use the `@Counted` annotation to create a simple counter. This example counts how many times the `createPortfolio` method has been called:

```java
@POST
@Path("/{owner}")
@Produces("application/json")
@Counted(monotonic=true, name="portfolios", displayName="Stock Trader portfolios", description="Number of portfolios created in the Stock Trader applications")
@RolesAllowed({"StockTrader"})
public JsonObject createPortfolio(@PathParam("owner") String owner) throws SQLException {
```
{: codeblock}

To build this code, we need to add the following stanza to our Maven `pom.xml`:

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

With that in place, every time the createPortfolio JAX-RS method gets called, the counter will be incremented. We can invoke the `GET /metrics` URI to see both jvm (classloading, heap, and garbage collection statistics) and application-defined metrics. The `GET /metrics/application` URI will return only application-defined metrics. We can hit this REST GET API via the curl CLI, using the assigned port (32388 in this example):

```
Johns-MacBook-Pro-8:StockTrader jalcorn$ curl http://9.42.2.107:32388/metrics/application

# TYPE application:com_ibm_hybrid_cloud_sample_portfolio_portfolio_portfolios counter

# HELP application:com_ibm_hybrid_cloud_sample_portfolio_portfolio_portfolios Number of portfolios created in the Stock Trader applications

application:com_ibm_hybrid_cloud_sample_portfolio_portfolio_portfolios

Johns-MacBook-Pro-8:StockTrader jalcorn$
```
{: screen}

As you can see, it has counted that we have created 2 portfolios. A few things of note:

- This is an in-memory counter: if the pod is restarted, the value will be reset to zero; if there are multiple replicas, each will have its own (localized) value.
- The "# HELP" text is what we specified as the description in the `@Counted` annotation.

You can view the output of this REST GET endpoint in your web browser as well:

![REST GET endpoint web browser](images/microprofile-metrics-image1.png "REST GET endpoint web browser"){: caption="Figure 1. REST GET endpoint web browser" caption-side="bottom"}

Note that by default, the `/metrics` endpoint requires https and login credentials be passed. Liberty 18.0.0.3 introduced the following stanza you can put in your server.xml to say you want this endpoint to allow http and to be unauthenticated:

```xml
<mpMetrics authentication="false"/>
```

This makes configuring the Prometheus scraper to hit this endpoint much easier.
