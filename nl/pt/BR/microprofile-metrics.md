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

# Métricas com MicroProfile
{: #mp-metrics}

O MicroProfile fornece um recurso de Métricas para incluir métricas customizadas em seu aplicativo, usando anotações simples. Para ativar esse recurso, inclua o recurso `mpMetrics-1.1` em seu `server.xml`. Opcionalmente, é possível incluir o recurso `monitor-1.0` se desejar ver mais métricas específicas do servidor de aplicativos (como sobre os conjuntos de conexões JDBC, por exemplo).

Importe a anotação `@Counted` para criar um contador simples:

```java
import org.eclipse.microprofile.metrics.annotation.Counted;
```
{: codeblock}

Em seguida, use a anotação `@Counted` para criar um contador simples, conforme mostrado no exemplo a seguir que conta o número de vezes que o método `createPortfoloio` é chamado: 

```java
@POST
@Path("/{owner}")
@Produces("application/json")
@Counted(monotonic=true, name="portfolios", displayName="Stock Trader portfolios", description="Number of portfolios created in the Stock Trader applications")
@RolesAllowed({"StockTrader"})
public JsonObject createPortfolio(@PathParam("owner") String owner) throws SQLException {
```
{: codeblock}

Para construir esse código, inclua a sub-rotina a seguir no arquivo `pom.xml` do Maven:

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

Com isso em vigor, o contador é incrementado toda vez que o método JAX-RS `createPortfolio` é chamado. 

É possível chamar o URI `GET /metrics` para ver as métricas da JVM (estatísticas de carregamento de classe, heap e coleta de lixo) e as definidas pelo aplicativo. O URI `GET /metrics/application` retorna somente as métricas definidas pelo aplicativo. 

É possível acessar a API de REST GET por meio da CLI curl usando a porta designada (32388 neste exemplo):

```
Johns-MacBook-Pro-8:StockTrader jalcorn$ curl http://9.42.2.107:32388/metrics/application

# TYPE application:com_ibm_hybrid_cloud_sample_portfolio_portfolio_portfolios counter

# HELP application:com_ibm_hybrid_cloud_sample_portfolio_portfolio_portfolios Number of portfolios created in the Stock Trader applications

application:com_ibm_hybrid_cloud_sample_portfolio_portfolio_portfolios

Johns-MacBook-Pro-8:StockTrader jalcorn$
```
{: screen}

Como é possível ver, dois portfólios são contados conforme esperado. 

Algumas coisas a notar:
- Um contador na memória será usado: se o pod for reiniciado, o valor será reconfigurado como zero; se houver múltiplas réplicas, cada uma terá o seu próprio valor exclusivo.
- O texto "# HELP" é o que está especificado como a descrição na anotação `@Counted`.

É possível visualizar a saída desse terminal GET de REST em seu navegador da web também:

![Navegador da web do terminal GET de REST](images/microprofile-metrics-image1.png "Navegador da web do terminal GET de REST")

Por padrão, o terminal `/metrics` requer que o https e as credenciais de login sejam passados. O Liberty 18.0.0.3 introduziu a sub-rotina a seguir que você pode colocar em seu `server.xml` para definir esse terminal para permitir http e para ser não autenticado:

```xml
<mpMetrics authentication="false"/>
```

A configuração do extrator Prometheus é simplificada para tornar o acesso ao terminal mais fácil.
