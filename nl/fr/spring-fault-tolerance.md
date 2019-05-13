---

copyright:
  years: 2018, 2019
lastupdated: "2019-04-22"

keywords: fault tolerance spring, hystrix spring, netflix spring, hystrixcommand spring, bulkhead spring, circuit breaker spring

subcollection: java

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}

# Tolérance aux pannes avec Spring
{: #spring-tolerance}

La création d'un système résilient place des exigences sur tous les services qu'il inclut. La nature dynamique d'un environnement cloud exige que des services soient conçus pour répondre à des situations inattendues.

Spring utilise la bibliothèque de tolérance aux pannes [Netflix Hystrix](https://github.com/Netflix/Hystrix/wiki){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe") pour prendre en charge les problème de tolérance aux pannes au niveau de l'application. Avec Hystrix, vous pouvez créer des rétromigrations, des disjoncteurs, des cloisons et des métriques associées.

Ces informations reposent sur les pratiques de tolérances aux pannes décrites dans [Développement cloud natif : Tolérance aux pannes](/docs/java?topic=cloud-native-fault-tolerance#fault-tolerance).
{: note}

## Utilisation d'Hystrix avec une application Spring
{: #spring-hystrix-starter}

Pour que Hystrix soit plus facile à utiliser au sein d'une application Spring, le module de démarrage Spring Boot active une approche annotée pour intégrer Hystrix dans votre application.

Pour ajouter Hystrix, ajoutez la dépendance suivante :

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```
{: codeblock}

Pour que Spring puisse utiliser cette nouvelle fonction, ajoutez une annotation à votre classe d'application principale et activez le traitement de Spring des annotations Hystrix pour la disjonction en ajoutant `@EnableCircuitBreaker`.

```java
@SpringBootApplication
@EnableCircuitBreaker
public class MyApplication {

	public static void main(String[] args) {
		SpringApplication.run(MyApplication.class, args);
	}
```
{: codeblock}

### Utilisation de HystrixCommand pour définir une rétromigration
{: #spring-fallback}

Pour ajouter un comportement de disjoncteur à une méthode, annotez-la avec `@HystrixCommand`. Spring détecte les méthodes dans une application annotée avec `@EnableCircuitBreaker` et les encapsule avec le support Hystrix afin de surveiller les erreurs et les dépassements de délai d'attente. Spring appelle ensuite l'alternative appropriée lorsque cela est nécessaire.

L'annotation `@HystrixCommand` est prise en charge uniquement sur les méthodes dans un élément `@Component` ou `@Service`.
{: note}

L'annotation `@HystrixCommand` suivante encapsule l'appel `service()` pour fournir le comportement de disjoncteur. Le proxy appelle la méthode `fallback()` si la méthode `service()` échoue ou si le circuit est ouvert.

```java
@Autowired
private MicroService myService;

@GetMapping("/endpoint")
public String value() throws Exception {
    return "Service returned: " + myService.service();
}

@Service
class MicroService {
    @HystrixCommand(fallbackMethod = "fallback")
    public String service() {
        // Do some processing. If an exception is thrown
        // the fallback method will be called
        return "Success";
    }

    public String fallback(Throwable e) {
        return "Fallback! Reason: " + e.getMessage() + "\n";
    }
}
```
{: codeblock}

Pour en savoir plus, consultez l'exemple de [disjoncteur Hystrix](https://spring.io/guides/gs/circuit-breaker/){: new_window} basé sur Spring ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe").
{: tip}

### Utilisation des délais d'attente
{: #spring-timeout}

Comment votre application répond-elle à un service distant qui ne répond pas ? L'attente peut être longue ou elle peut durer indéfiniment. Hystrix vous offre la possibilité de définir la durée d'attente acceptable pour votre application.

Un simple ajout à l'annotation `HystrixCommand` peut permettre de modifier la valeur par défaut du délai d'attente (de 1 seconde à 30 secondes, par exemple) :

```java
@HystrixCommand(
    fallbackMethod = "fallback",
    commandProperties = {
        @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "30000"),
    }
)
```
{: codeblock}

### Utilisation des cloisons
{: #spring-bulkhead}

Hystrix prend en charge à la fois les cloisons sémaphores et les cloisons basées sur les files d'attente. Le fragment suivant présente comment configurer une cloison basée sur une file d'attente qui alloue quatre unités d'exécution et limite le nombre de demandes en suspens à 10 :

```java
@HystrixCommand(
    fallbackMethod = "fallback",
    threadPoolProperties = {
        @HystrixProperty(name = "coreSize", value = "4"),
        @HystrixProperty(name = "maxQueueSize", value = "10")
    }
)
```
{: codeblock}

### Statut du disjoncteur
{: #spring-breaker-status}

Le module de démarrage Hystrix Spring permet également, par l'intermédiaire d'un élément Spring Actuator, d'améliorer le noeud final `/health` par défaut pour l'application. Pour plus d'informations, voir [Métriques avec Spring](/docs/java?topic=java-spring-metrics#spring-metrics).

Si le noeud final d'intégrité est [configuré pour inclure des détails supplémentaires](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-endpoints.html#production-ready-health){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe"), le statut du disjoncteur est inclus dans les informations de diagnostic d'intégrité (ce comportement est désactivé par défaut).

```
{
    "hystrix": {
        "openCircuitBreakers": [
            "MicroService::service"
        ],
        "status": "CIRCUIT_OPEN"
    },
    "status": "UP"
}
```
{: screen}

## Etapes suivantes
{: #spring-tolerance-next-steps notoc}

Pour en savoir plus sur la configuration d'Hystrix, consultez le [wiki sur la configuration d'Hystrix](https://github.com/Netflix/Hystrix/wiki/Configuration){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe").

Pour plus d'informations sur Hystrix avec Spring, voir :

* [A Guide to Spring Cloud Netflix](https://www.baeldung.com/spring-cloud-netflix-hystrix){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe")
* [Spring Circuit Breaker Guide](https://spring.io/guides/gs/circuit-breaker/){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe")
