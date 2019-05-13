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

# Métriques avec MicroProfile
{: #mp-metrics}

MicroProfile inclut une fonction Metrics permettant d'ajouter des métriques personnalisées à votre application en utilisant des annotations simples. Pour activer cette fonction, ajoutez la fonction `mpMetrics-1.1` à votre fichier `server.xml`. Vous pouvez éventuellement ajouter la fonction `monitor-1.0` si vous souhaitez voir plus de métriques spécifiques au serveur d'applications (par exemple, sur les pools de connexions JDBC).

Importez l'annotation `@Counted` pour créer un compteur simple :

```java
import org.eclipse.microprofile.metrics.annotation.Counted;
```
{: codeblock}

Utilisez ensuite l'annotation `@Counted` pour créer un compteur simple, comme présenté dans l'exemple suivant, qui compte le nombre de fois où la méthode `createPortfoloio` est appelée : 

```java
@POST
@Path("/{owner}")
@Produces("application/json")
@Counted(monotonic=true, name="portfolios", displayName="Stock Trader portfolios", description="Number of portfolios created in the Stock Trader applications")
@RolesAllowed({"StockTrader"})
public JsonObject createPortfolio(@PathParam("owner") String owner) throws SQLException {
```
{: codeblock}

Pour générer ce code, ajoutez la strophe suivante au fichier `pom.xml` de Maven :

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

Une fois que cela est fait, le compteur est incrémenté chaque fois que la méthode JAX-RS `createPortfolio` est appelée. 

Vous pouvez appeler l'URI `GET /metrics` pour voir les métriques définies au niveau de l'application et au niveau de la machine virtuelle Java (statistiques sur le chargement des classes, les segments de mémoire et le nettoyage de la mémoire). L'URI `GET /metrics/application` renvoie uniquement des métriques définies par l'application. 

Vous pouvez accéder à l'API REST GET via l'interface de ligne de commande curl en utilisant le port affecté (32388 dans cet exemple) :

```
Johns-MacBook-Pro-8:StockTrader jalcorn$ curl http://9.42.2.107:32388/metrics/application

# TYPE application:com_ibm_hybrid_cloud_sample_portfolio_portfolio_portfolios counter

# HELP application:com_ibm_hybrid_cloud_sample_portfolio_portfolio_portfolios Number of portfolios created in the Stock Trader applications

application:com_ibm_hybrid_cloud_sample_portfolio_portfolio_portfolios

Johns-MacBook-Pro-8:StockTrader jalcorn$
```
{: screen}

Comme vous le voyez, deux portefeuilles sont comptabilisés, comme prévu. 

Notez les points suivants :
- Il s'agit d'un compteur en mémoire : si le pod est redémarré, la valeur est réinitialisée à zéro ; s'il y a plusieurs répliques, chacune a sa propre valeur unique.
- Le texte "# HELP" est celui spécifié comme description dans l'annotation `@Counted`.

Vous pouvez également afficher la sortie de ce noeud final REST GET dans votre navigateur Web :

![Noeud final REST GET - Navigateur Web](images/microprofile-metrics-image1.png "Noeud final REST GET - Navigateur Web"){: caption="Figure 1. Noeud final REST GET - Navigateur Web" caption-side="bottom"}

Par défaut, le noeud final `/metrics` exige que les données d'identification de connexion et https soient transmises. Liberty 18.0.0.3 a introduit la chaîne suivante que vous pouvez ajouter à votre fichier `server.xml` pour faire en sorte que ce noeud final autorise http et soit non authentifié :

```xml
<mpMetrics authentication="false"/>
```

La configuration du programme d'extraction Prometheus est simplifiée afin de faciliter l'accès au noeud final.
