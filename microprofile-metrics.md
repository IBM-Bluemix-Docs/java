---

copyright:
  years: 2019
lastupdated: "2019-02-05"

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

MicroProfile provides `mpMetrics`, which let you annotate your source code saying what you'd like to track in your application. Note that, while `mpMetrics` is a great addition to MicroProfile, it does have some issues with how it integrates with Prometheus, tracked in issue #136: https://github.ibm.com/ibmcloud/CloudNative/issues/135. :FIXME:

For example, in our Portfolio microservice, we are asserting that we would like for it to track a count of how many portfolios have been created. We do this first by importing the Counted annotation:

import org.eclipse.microprofile.metrics.annotation.Counted;

And then we add the @Counted annotation to the createPortfolio method:

```java
@POST
@Path("/{owner}")
@Produces("application/json")
@Counted(monotonic=true, name="portfolios", displayName="Stock Trader portfolios", description="Number of portfolios created in the Stock Trader applications")
@RolesAllowed({"StockTrader"})
public JsonObject createPortfolio(@PathParam("owner") String owner) throws SQLException {
```

To build this code, we need to add the following stanza to our Maven pom.xml:

```xml
<dependency>
  <groupId>org.eclipse.microprofile</groupId>
  <artifactId>microprofile</artifactId>
  <version>2.0.1</version>
  <type>pom</type>
  <scope>provided</scope>
</dependency>
```

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

![](images/microprofile-metrics-image1.png){width="6.655in" height="0.9289271653543307in"}

Note that by default, the `/metrics` endpoint requires https and login credentials be passed. But Liberty 18.0.0.3 introduced the following stanza you can put in your server.xml to say you want this endpoint to allow http and to be unauthenticated:

```xml
<mpMetrics authentication="false"/>
```

This makes configuring the Prometheus scraper to hit this endpoint much easier.

**To Do:** Describe configuring the Prometheus scraper to hit this endpoint, and show a Grafana chart of it.
