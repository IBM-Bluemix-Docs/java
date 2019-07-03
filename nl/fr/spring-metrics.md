---

copyright:
  years: 2019
lastupdated: "2019-06-10"

keywords: spring metrics, configure metrics spring, micrometer spring, micrometer, spring boot 2, spring actuator, prometheus spring

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

# Métriques avec Spring
{: #spring-metrics}

A partir de Spring Framework 5, les métriques dans Spring sont traitées par Micrometer. Micrometer est une infrastructure qui se décrit elle-même comme "SLF4J for Metrics". De la même manière que SLF4J agit en tant qu'API non liée à un fournisseur pour la journalisation et se connecte à différents systèmes de back-end de journalisation, Micrometer inclut une API non liée à un fournisseur qui permet d'instrumenter et de mesurer votre code. Il peut ensuite fournir ces mesures à différents agrégateurs de métriques, tels que Prometheus, DataDog ou Influx/Telegraph. 

L'infrastructure Micrometer permet à Spring de s'intégrer à une grande variété d'architectures natives du cloud. Pour activer la prise en charge de Prometheus ou Statsd, il vous suffit de modifier les dépendances, et si le collecteur de métriques est de type Push, de fournir des informations de destination dans `application.properties`. Spring et Micrometer déterminent ce qu'il faut faire au moment de l'exécution en fonction des dépendances qu'ils trouvent sur le chemin de classe de l'application.

## Activation de Métriques
{: #spring-metrics-enabling}

Des métriques de base sont fournies par Spring Boot Actuator, qui demande la dépendance `spring-boot-starter-actuator` dans votre fichier `pom.xml` :  

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-actuator</artifactId>
</dependency>
```
{: codeblock}

Si vous utilisez Spring Boot 1.5.x, les métriques (et les actionneurs) fonctionnent un peu différemment. Micrometer est devenu la norme pour les métriques Spring dans Spring Boot 2.0, mais des rétroportages ont été mis à disposition pour Boot 1.5.x, ce qui permet des pratiques cohérentes pour créer et collecter des métriques. Plus important encore, Micrometer prend en charge plusieurs systèmes de surveillance, dont Prometheus, qui font de lui un meilleur choix que le système de métriques Boot 1.5.x pour les déploiements en cloud.
{: note}

## Métriques Micrometer
{: #spring-metrics-micrometer}

Micromètre résout le problème de destination pour les métriques via son interface `MeterRegistry`. L'interface `MeterRegistry` permet entre autres à l'application d'obtenir des jauges, des minuteurs et des compteurs personnalisés. Si plusieurs destinations sont requises, Micrometer fournit une implémentation `CompositeMeterRegistry`, permettant d'isoler l'application des destinations réellement configurées pour les métriques.

Dans l'une ou l'autre des versions de Spring Boot, lorsqu'un noeud final de métrique basé sur Micrometer est activé, Spring Boot crée automatiquement un `CompositeMeterRegistry`, et le met à disposition de l'application en tant que `Bean`. Le registre est automatiquement rempli avec les implémentations `MeterRegistry` prises en charge trouvées dans le chemin d'accès aux classes. La prise en charge de plusieurs systèmes de surveillance, ou le passage d'un système à l'autre, est alors une simple question de modification des dépendances dans le fichier pom.xml. 

Spring permet de personnaliser le bean `MeterRegistry` via un élément `MeterRegistryCustomizer`, qui lui-même est un bean. Lorsque Spring identifie la présence d'un tel bean, il peut configurer l'élément `MeterRegistry` global avant que les mesures ne soient rapportées. Cette personnalisation est souvent utilisée pour ajouter des balises communes à `MeterRegistry`, DataCenter ou EnvironmentType.

Lorsque vous nommez des métriques, Spring et Micrometer recommandent de suivre une convention de nommage qui divise les mots avec un `.` plutôt que d'utiliser une casse mixte ou `_`, ou d'autres approches. Ces conventions sont importantes car Micrometer relaie ces métriques vers une ou plusieurs destinations configurées et effectue toutes les conversions de noms nécessaires pour répondre aux exigences des destinations.

## Configuration des métriques avec Spring Boot
{: #spring-metrics-configuration}

Les sections suivantes mettent en avant le mode d'activation des métriques Spring Boot Actuator avec Micrometer. Apprenez à collecter des métriques et fournissez un noeud final pour Prometheus (pour chaque édition) en commençant par Spring Boot 2 comme version la plus récente.

### Configuration des métriques dans Spring Boot 2
{: #spring-metrics-boot2}

Le noeud final metrics de l'actionneur Spring Boot Actuator 2 doit être à la fois activé et exposé pour l'accès Web. Par défaut, le noeud final est activé mais il n'est pas exposé (consultez la [documentation sur les noeuds finaux de Spring ](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-endpoints.html){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe") pour plus d'informations).  

Ajoutez ce qui suit à `application.properties` pour exposer le noeud final metrics :

```properties
management.endpoints.web.exposure.include=health,metrics
```
{: codeblock}

Vérifiez que le noeud final metrics est activé en activant le noeud final `/actuator`. La sortie est similaire à l'exemple suivant lorsque vous exécutez l'application localement et utilisez `jq` pour formater automatiquement la réponse JSON :

```
$ curl -s localhost:8080/actuator | jq .
{
  "_links": {
    "self": {
      "href": "http://localhost:8080/actuator",
      "templated": false
    },
    ...
    "metrics": {
      "href": "http://localhost:8080/actuator/metrics",
      "templated": false
    },
    "metrics-requiredMetricName": {
      "href": "http://localhost:8080/actuator/metrics/{requiredMetricName}",
      "templated": true
    },
    ...
  }
}
```
{: screen}

Accédez au noeud final `/actuator/metrics` pour afficher une liste formatée en JSON des métriques disponibles, similaire à la sortie suivante :
```
$ curl -s localhost:8080/actuator/metrics | jq .
{
  "names": [
    "jvm.memory.max",
    "jvm.threads.states",
    "process.files.max",
    "jvm.gc.memory.promoted",
    ...
  ]
}
```
{: screen}

Une autre requête est nécessaire pour voir la valeur réelle d'une métrique individuelle (comme on peut le déduire de la liste des URL d'actionneurs). Pour récupérer les états de l'unité d'exécution jvm, par exemple :

```
$ curl -s localhost:8080/actuator/metrics/jvm.threads.states | jq .
{
  "name": "jvm.threads.states",
  "description": "The current number of threads having TERMINATED state",
  "baseUnit": "threads",
  "measurements": [
    {
    "statistic": "VALUE",
    "value": 24
    }
  ],
  "availableTags": [
    {
      "tag": "state",
      "values": [
        "timed-waiting",
        "new",
        "runnable",
        "blocked",
        "waiting",
        "terminated"
      ]
    }
  ]
}
```
{: screen}

Pour plus d'informations sur la personnalisation du comportement de l'actionneur de métriques, voir la [documentation officielle](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-metrics.html){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe").

### Activation de la prise en charge de Prometheus dans Spring Boot 2
{: #spring-prometheus-boot2}

Prometheus nécessite que l'application héberge un noeud final que le serveur Prometheus lit pour obtenir les métriques. L'hébergement de ce noeud final dans Spring repose sur le fait que l'application a déjà la capacité de servir des données. En d'autre termes, le noeud final Prometheus prend uniquement en charge les applications Spring Boot qui utilisent Spring MVC, Spring WebFlux ou Jersey. 

Pour activer le noeud final Prometheus, ajoutez la dépendance suivante dans `pom.xml` :
```xml
<dependency>
  <groupId>io.micrometer</groupId>
  <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```
{: codeblock}

Spring voit la dépendance et configure automatiquement `PrometheusMeterRegistry` comme l'un des registres configurés dans le bean `MeterRegistry` global. Cependant, comme pour le noeud final metrics, le noeud final Prometheus doit être activé et exposé pour l'accès Web en ajoutant ceci à `application.properties` :

```properties
management.endpoints.web.exposure.include=health,metrics,prometheus
```
{: codeblock}

Vérifiez que le noeud final Prometheus est activé en activant le noeud final `/actuator`. La sortie est similaire à l'exemple suivant lorsque vous exécutez l'application localement : 

```
$ curl -s localhost:8080/actuator | jq .
{
  "_links": {
    "self": {
      "href": "http://localhost:8080/actuator",
      "templated": false
    },
    ...
    "prometheus": {
      "href": "http://localhost:8080/actuator/prometheus",
      "templated": false
    },
    ...
  }
}
```
{: screen}

Le noeud final `/actuator/prometheus` émet toutes les métriques configurées dans le format d'extraction de Prometheus :

```
$ curl -s localhost:8080/actuator/prometheus
# HELP system_cpu_usage The "recent cpu usage" for the whole system
# TYPE system_cpu_usage gauge
system_cpu_usage 0.046231032221805926
...
```
{: screen}

Ajoutez les annotations suivantes à votre définition de service pour activer l'extraction de Prometheus pour le noeud final metrics :

```yaml
apiVersion: v1
kind: Service
metadata:
  name: ...
  annotations:
    prometheus.io/scrape: 'true'
    prometheus.io/scheme: http
    prometheus.io/path: /actuator/prometheus
    prometheus.io/port: "8080"
  ...
```
{: codeblock}

### Configuration des métriques dans Spring Boot 1
{: #spring-metrics-boot1}

Le noeud final `/metrics` de l'actionneur Spring Boot Actuator est activé par défaut dans Spring Boot 1, mais il est considéré “sensible” et demande une autorisation. Pour certains environnements, la sécurisation du noeud final metrics peut être la réponse correcte mais l'autorisation de chaque requête envoyée au noeud final metrics introduit un temps d'attente trop important pour la vérification automatisée avec Kubernetes.

Désactivez l'attribut sensible du noeud final metrics pour supprimer le besoin d'autorisation en ajoutant ce qui suit à `application.properties` :
```properties
endpoints.metrics.sensitive=false
```
{: codeblock}

A présent que la prise en charge des métriques par défaut de Spring Boot 1 est activée, la sortie suivante est générée : 
```
$ curl -s localhost:8080/metrics | jq .
{
  "names": [
    "jvm.memory.max",
    "jvm.threads.states",
    "process.files.max",
    "jvm.gc.memory.promoted",
    ...
  ]
}
```
{: screen}

Pour en savoir plus sur la manière dont vous pouvez personnaliser le comportement de l'actionneur de métriques de Spring Boot 1, veuillez consulter la [documentation officielle](https://docs.spring.io/spring-boot/docs/1.5.x/reference/html/production-ready-metrics.html){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe").

### Activation de la prise en charge Prometheus dans Spring Boot 1
{: #spring-prometheus-boot1}

La prise en charge de Prometheus n'est pas intégrée dans les paramètres de l'actionneur Spring Boot 1, mais elle peut être ajoutée via Micrometer.

Comme mentionné dans la section [Présentation](#spring-metrics), Micrometer a été rétroporté vers Spring Boot 1 parce qu'il possède un ensemble plus riche de primitives de mesure et une meilleure prise en charge des systèmes de surveillance comme Prometheus et StatsD. L'utilisation de Micrometer avec Spring Boot 1 est simple : elle nécessite une bibliothèque d'adaptateurs spécifique, et au moins un registre. Ajoutez ce qui suit à votre fichier `pom.xml` pour inclure des dépendances pour la bibliothèque d'adaptateurs et le registre Prometheus :

```xml
<properties>
  ...
  <micrometer.version>1.1.1</micrometer.version>
</properties>
<dependencies>
  <dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-spring-legacy</artifactId>
    <version>${micrometer.version}</version>
  </dependency>
  <dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
    <version>${micrometer.version}</version>
  </dependency>
</dependencies>
```
{: codeblock}

Ces dépendances créent et exposent un noeud final `/prometheus`. Comme pour le noeud final `/metrics` par défaut, le noeud final `/prometheus` est considéré sensible par défaut. Pour supprimer l'exigence d'autorisation, ajoutez ce qui suit à `application.properties` :

```properties
endpoints.prometheus.sensitive=false
```
{: codeblock}

Le résultat généré par le noeud final `/prometheus` est similaire à celui de Spring Boot 2 :

```
$ curl -s localhost:8080/prometheus
# HELP jvm_memory_used_bytes The amount of used memory
# TYPE jvm_memory_used_bytes gauge
jvm_memory_used_bytes{area="heap",id="PS Survivor Space",} 2.2594152E7
...
```
{: screen}

Ajoutez les annotations suivantes à votre définition de service pour activer l'extraction de Prometheus pour le noeud final metrics :

```yaml
apiVersion: v1
kind: Service
metadata:
  name: …
  annotations:
    prometheus.io/scrape: 'true'
    prometheus.io/scheme: http
    prometheus.io/path: /prometheus
    prometheus.io/port: "8080"
  ...
```
{: codeblock}

Pour plus d'informations sur l'utilisation de Micrometer avec Spring Boot 1, voir [https://micrometer.io/docs/ref/spring/1.5](https://micrometer.io/docs/ref/spring/1.5){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe").

## Invocations de temporisation avec Micrometer
{: #spring-metrics-timing}

Spring MVC, WebFlux et Jersey Server fournissent chacun une auto-configuration pour Spring qui collecte automatiquement des informations de temporisation pour toutes les requêtes lorsque la propriété `management.metrics.web.server.auto-time-requests` est réglée sur `true`. Le paramètre par défaut est `true`.

```properties
management.metrics.web.server.auto-time-requests=true
```
{: codeblock}

Micrometer prend également en charge l'annotation `@Timed`, qui peut être utilisée pour collecter des informations de temporisation pour les appels de méthode pris en charge. Il est important de noter que cette annotation ne peut pas être utilisée pour chronométrer des méthodes arbitraires, car elle nécessite la prise en charge des conteneurs pour recueillir les données. Les annotations `@Timed` sont prises en charge pour les opérations sauvegardées par un conteneur de servlet, par exemple les méthodes de service Spring MVC, Webflux ou Jersey.

Pour être plus sélectif quant aux informations de temporisation collectées, définissez la propriété `management.metrics.web.server.auto-time-requests` sur `false` et utilisez l'annotation `@Timed` sur chaque méthode de service, ou contrôleur :

```java
@Timed
@GetMapping("/myendpoint")
public String getSomething() { ... }
```
{: codeblock}

Si `management.metrics.web.server.auto-time-requests` est défini sur `true`, l'annotation `@Timed` peut être utilisée pour ajouter des balises additionnelles à une métrique signalée spécifique, par exemple :

```java
@Timed(extraTags = { "mytagname", "mytagvalue"})
@GetMapping("/myendpoint")
public String getSomething() { ... }
```
{: codeblock}

## Métriques personnalisées avec Spring et Micrometer
{: #spring-metrics-custom}

Pour effectuer des mesures spécifiques au domaine d'application, vous devez créer vos propres métriques personnalisées. Micrometer définit plusieurs types de compteurs de base`` :

* `Counter` suit une valeur qui augmente.
* `Gauge` mesure et renvoie la valeur observée lorsque le compteur est publié (ou interrogé).
* `Timer` enregistre le nombre d'occurrences d'un événement et le temps cumulé écoulé pour cet événement. 
* `DistributionSummary` est similaire à un élément `Timer` mais il effectue le suivi d'une distribution d'événements sans mesurer la durée.

Les nouveaux compteurs sont créés soit à l'aide des méthodes d'auxiliaire sur `MeterRegistry`, soit à l'aide de l'élément Fluent Builder du compteur. 

Créez votre compteur `Meter` dans le constructeur (en transmettant `MeterRegistry` comme paramètre) ou dans une méthode `@PostConstruct` après l'injection de `MeterRegistry`.

Par exemple, pour utiliser un élément de type `Counter` dans un élément `RestController`, vous pouvez procéder comme suit.

```java
@RestController
public class MyController {
    
    private final Counter myCounter;
    
    public MyController(MeterRegistry meterRegistry) {
    
        // Create the counter using the helper method on the builder
        myCounter = meterRegistry.counter("my.counter", "mytagname", "mytagvalue");
        
    }
```
{: codeblock}

Puis, dans vos méthodes de service, vous pouvez mettre à jour la valeur de compteur :

```java
@GetMapping("/")
public String getSomething() {
    myCounter.increment();
    ...
}
```
{: codeblock}


Pour en savoir plus sur la création de métriques personnalisées dans Spring, voir :

* [Métriques Spring Boot](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-metrics.html){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe")
* [Concepts de Micrometer](https://micrometer.io/docs/concepts){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe")
