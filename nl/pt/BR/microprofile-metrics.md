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

# Métricas com MicroProfile
{: #mp-metrics}

O MicroProfile fornece um recurso de Métricas que permite que você use anotações simples para incluir métricas customizadas em seu aplicativo. Você apenas inclui o recurso `mpMetrics-1.1` em seu `server.xml` para ativar isso. Observe que é possível, opcionalmente, incluir também o recurso `monitor-1.0`, se desejar que métricas adicionais específicas do servidor de app (sobre os conjuntos de conexões JDBC, por exemplo).

Importe a anotação `@Counted` para criar um contador simples:

```java
import org.eclipse.microprofile.metrics.annotation.Counted;
```
{: codeblock}

Em seguida, use a anotação `@Counted` para criar um contador simples. Esse exemplo conta quantas vezes o método `createPortfolio` foi chamado:

```java
@POST
@Path("/{owner}")
@Produces("application/json")
@Counted(monotonic=true, name="portfolios", displayName="Stock Trader portfolios", description="Number of portfolios created in the Stock Trader applications")
@RolesAllowed({"StockTrader"})
public JsonObject createPortfolio(@PathParam("owner") String owner) throws SQLException {
```
{: codeblock}

Para construir esse código, é necessário incluir a sub-rotina a seguir em nosso Maven `pom.xml`:

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

Com isso no lugar, toda vez que o método createPortfolio JAX-RS for chamado, o contador será incrementado. Podemos chamar o URI `GET /metrics` para ver as métricas jvm (classloading, heap e estatísticas de coleta de lixo) e as métricas definidas pelo aplicativo. O URI `GET /metrics/application` retornará somente métricas definidas pelo aplicativo. Podemos atingir essa API GET de REST por meio da CLI curl, usando a porta designada (32388 nesse exemplo):

```
Johns-MacBook-Pro-8:StockTrader jalcorn$ curl http://9.42.2.107:32388/metrics/application

# TYPE application:com_ibm_hybrid_cloud_sample_portfolio_portfolio_portfolios counter

# HELP application:com_ibm_hybrid_cloud_sample_portfolio_portfolio_portfolios Number of portfolios created in the Stock Trader applications

application:com_ibm_hybrid_cloud_sample_portfolio_portfolio_portfolios

Johns-MacBook-Pro-8:StockTrader jalcorn$
```
{: screen}

Como você pode ver, ele contou que nós criamos 2 portfólios. Algumas coisas a notar:

- Este é um contador na memória: se o pod for reiniciado, o valor será reconfigurado para zero; se houver várias réplicas, cada uma terá seu próprio valor (localizado).
- O texto "# HELP" é o que especificamos como a descrição na anotação `@Counted`.

É possível visualizar a saída desse terminal GET de REST em seu navegador da web também:

![Navegador da web do terminal GET de REST](images/microprofile-metrics-image1.png "Navegador da web do terminal GET de REST")

Observe que, por padrão, o terminal `/metrics` requer que https e credenciais de login sejam passados. O Liberty 18.0.0.3 introduziu a sub-rotina a seguir que é possível colocar em seu server.xml para dizer que você deseja que esse terminal permita http e para não ser autenticado:

```xml
<mpMetrics authentication="false"/>
```

Isso torna a configuração do escrêiper da Prometheus para atingir esse terminal muito mais fácil.
