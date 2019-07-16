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

# Métricas con MicroProfile
{: #mp-metrics}

MicroProfile proporciona una función Metrics para añadir métricas personalizadas a su aplicación utilizando anotaciones simples. Para habilitar esta característica, añada la característica `mpMetrics-1.1` a `server.xml`. De manera opcional, puede añadir la característica `monitor-1.0` si desea ver más métricas específicas del servidor de la app (como agrupaciones de conexiones JDBC, por ejemplo).

Importe la anotación `@Counted` para crear un contador simple:

```java
import org.eclipse.microprofile.metrics.annotation.Counted;
```
{: codeblock}

A continuación, utilice la anotación `@Counted` para crear un contador simple, tal como se muestra en el ejemplo siguiente, que cuenta cuántas veces se llama al método `createPortfoloio`: 

```java
@POST
@Path("/{owner}")
@Produces("application/json")
@Counted(monotonic=true, name="portfolios", displayName="Stock Trader portfolios", description="Number of portfolios created in the Stock Trader applications")
@RolesAllowed({"StockTrader"})
public JsonObject createPortfolio(@PathParam("owner") String owner) throws SQLException {
```
{: codeblock}

Para compilar este código, añada la stanza siguiente al archivo `pom.xml` de Maven:

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

De este modo, el contador se incrementará cada vez que se llame al método `createPortfolio` de JAX-RS. 

Puede invocar el URI `GET /metrics` para ver tanto las métricas de JVM (estadísticas de carga de clase, almacenamiento dinámico y recogida de basura) como las definidas por la aplicación. El URI `GET /metrics/application` devuelve únicamente métricas definidas por la aplicación. 

Puede acceder a la API REST GET a través de la CLI de curl utilizando el puerto asignado (32388 en este ejemplo):

```
Johns-MacBook-Pro-8:StockTrader jalcorn$ curl http://9.42.2.107:32388/metrics/application

# TYPE application:com_ibm_hybrid_cloud_sample_portfolio_portfolio_portfolios counter

# HELP application:com_ibm_hybrid_cloud_sample_portfolio_portfolio_portfolios Number of portfolios created in the Stock Trader applications

application:com_ibm_hybrid_cloud_sample_portfolio_portfolio_portfolios

Johns-MacBook-Pro-8:StockTrader jalcorn$
```
{: screen}

Como puede ver, se cuentan dos portfolios tal como se esperaba. 

Varios puntos destacados:
- Se utiliza un contador en memoria: si se reinicia el pod, el valor se restablece a cero; si hay varias réplicas, cada una tiene su propio valor exclusivo.
- El texto de "# HELP" es lo que se especifica como descripción en la anotación `@Counted`.

También puede ver la salida de este punto final GET de REST en el navegador web:

![Navegador web de punto final GET REST](images/microprofile-metrics-image1.png "Navegador web de punto final GET REST")
{: caption="Figura 1. Navegador web de punto final GET REST" caption-side="bottom"}

De forma predeterminada, el punto final `/metrics` requiere que se pasen las credenciales de inicio de sesión y https. En Liberty 18.0.0.3 se introdujo la stanza siguiente que puede poner en
`server.xml` para definir que este punto final permite http y que se desautentique:

```xml
<mpMetrics authentication="false"/>
```

Se ha simplificado la configuración del scraper Prometheus para hacer más fácil el acceso al punto final.
