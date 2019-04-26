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

# Métriques avec MicroProfile
{: #mp-metrics}

MicroProfile offre la fonction Métriques qui vous permet d'utiliser des annotations simples pour ajouter des métriques personnalisées à votre application. Il vous suffit d'ajouter la fonction `mpMetrics-1.1` à votre fichier `server.xml` pour l'activer. Notez que vous pouvez également ajouter la fonction `monitor-1.0`, si vous souhaitez des métriques supplémentaires spécifiques au serveur d'application (comme par exemple, des pools de connexion JDBC).

Importez l'annotation `@Counted` pour créer un compteur simple :

```java
import org.eclipse.microprofile.metrics.annotation.Counted;
```
{: codeblock}

Utilisez ensuite l'annotation `@Counted` pour créer un compteur simple. Cet exemple compte combien de fois la méthode `createPortfolio` a été appelée :

```java
@POST
@Path("/{owner}")
@Produces("application/json")
@Counted(monotonic=true, name="portfolios", displayName="Stock Trader portfolios", description="Number of portfolios created in the Stock Trader applications")
@RolesAllowed({"StockTrader"})
public JsonObject createPortfolio(@PathParam("owner") String owner) throws SQLException {
```
{: codeblock}

Pour générer ce code, nous devons ajouter la strophe suivante à notre fichier maven `pom.xml` :

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

Une fois que cela est fait, le compteur est incrémenté chaque fois que la méthode createPortfolio JAX-RS est appelée. Nous pouvons invoquer l'URI `GET /metrics` pour voir les métriques définies au niveau de l'application et au niveau de la machine virtuelle Java (statistiques sur le chargement des classes, les tas et le nettoyage de la mémoire). L'URI `GET /metrics/application` reverra uniquement les métriques définies au niveau de l'application. Nous pouvons accéder à cette API REST GET via le CLI de curl, en utilisant le port assigné (32388 dans cet exemple) :

```
Johns-MacBook-Pro-8:StockTrader jalcorn$ curl http://9.42.2.107:32388/metrics/application

# TYPE application:com_ibm_hybrid_cloud_sample_portfolio_portfolio_portfolios counter

# HELP application:com_ibm_hybrid_cloud_sample_portfolio_portfolio_portfolios Number of portfolios created in the Stock Trader applications

application:com_ibm_hybrid_cloud_sample_portfolio_portfolio_portfolios

Johns-MacBook-Pro-8:StockTrader jalcorn$
```
{: screen}

Comme vous pouvez le voir, le comptage fait apparaître la création de 2 portefeuilles. Notez les points suivants :

- Il s'agit d'un compteur en mémoire : si le pod est redémarré, la valeur sera remise à zéro ; s'il y a plusieurs répliques, chacune aura sa propre valeur (localisée).
- Le texte "# HELP" est ce que nous avons spécifié comme description dans l'annotation `@Counted`.

Vous pouvez également afficher la sortie de ce noeud final REST GET dans votre navigateur Web :

![Noeud final REST GET - Navigateur Web](images/microprofile-metrics-image1.png "Noeud final REST GET - Navigateur Web"){: caption="Figure 1. Noeud final REST GET - Navigateur Web" caption-side="bottom"}

Notez que par défaut, le noeud final `/metrics` nécessite que le https et les informations de connexion soient transmis. Liberty 18.0.0.3 a introduit la strophe suivante que vous pouvez ajouter à votre fichier server.xml pour dire que vous voulez que ce noeud final autorise http et soit non authentifié :

```xml
<mpMetrics authentication="false"/>
```

La configuration du scraper de Prometheus est simplifiée pour accéder plus facilement au noeud final.
