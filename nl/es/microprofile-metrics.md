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

# Métricas con MicroProfile
{: #mp-metrics}

MicroProfile proporciona una prestación de métricas que le permite utilizar anotaciones simples para añadir métricas personalizadas a la aplicación. Basta con añadir la característica `mpMetrics-1.1` a `server.xml` para habilitar esta opción. Tenga en cuenta que, opcionalmente, también puede añadir la característica `monitor-1.0` si desea más métricas específicas del servidor de apps (como, por ejemplo, las agrupaciones de conexiones JDBC).

Importe la anotación `@Counted` para crear un contador simple:

```java
import org.eclipse.microprofile.metrics.annotation.Counted;
```
{: codeblock}

A continuación, utilice la anotación `@Counted` para crear un contador simple. Este ejemplo cuenta el número de veces que se ha llamado al método `createPortfolio`:

```java
@POST
@Path("/{owner}")
@Produces("application/json")
@Counted(monotonic=true, name="portfolios", displayName="Stock Trader portfolios", description="Number of portfolios created in the Stock Trader applications")
@RolesAllowed({"StockTrader"})
public JsonObject createPortfolio(@PathParam("owner") String owner) throws SQLException {
```
{: codeblock}

Para crear este código, tenemos que añadir la stanza siguiente a nuestro `pom.xml` de Maven:

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

De este modo, cada vez que se llame al método createPortfolio JAX-RS, el contador se incrementará. Podemos invocar el URI `GET /metrics` para ver las métricas de jvm (estadísticas de carga de clases, almacenamiento dinámico y recogida de basura) y las definidas por la aplicación. El URI `GET /metrics/application` sólo devolverá métricas definidas por la aplicación. Se puede alcanzar esta API GET de REST mediante la CLI de curl, utilizando el puerto asignado (32388 en este ejemplo):

```
Johns-MacBook-Pro-8:StockTrader jalcorn$ curl http://9.42.2.107:32388/metrics/application

# TYPE application:com_ibm_hybrid_cloud_sample_portfolio_portfolio_portfolios counter

# HELP application:com_ibm_hybrid_cloud_sample_portfolio_portfolio_portfolios Number of portfolios created in the Stock Trader applications

application:com_ibm_hybrid_cloud_sample_portfolio_portfolio_portfolios

Johns-MacBook-Pro-8:StockTrader jalcorn$
```
{: screen}

Como puede ver, ha contado que hemos creado dos portafolios. Varios puntos destacados:

- Se trata de un contador en memoria: si se reinicia el pod, el valor se restablecerá a cero; si hay varias réplicas, cada una tendrá su propio valor (localizado).
- El texto de "# HELP" es lo que hemos especificado como descripción en la anotación `@Counted`.

También puede ver la salida de este punto final GET de REST en el navegador web:

![Navegador web de punto final GET de REST](images/microprofile-metrics-image1.png "Navegador web de punto final GET de REST")

Tenga en cuenta que, de forma predeterminada, el punto final `/metrics` requiere que se pasen las credenciales de inicio de sesión y https. En Liberty 18.0.0.3 se introdujo la stanza siguiente, que puede poner en server.xml para indicar que desea que este punto final permita http y no tenga autenticación:

```xml
<mpMetrics authentication="false"/>
```

Esto hace que la configuración del recopilador de Prometheus alcance este punto final con mucha más facilidad.
