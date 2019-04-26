---

copyright:
  years: 2018, 2019
lastupdated: "2019-03-15"

keywords: fault tolerance spring, hystrix spring, netflix spring, hystrixcommand spring, bulkhead spring, circuit breaker spring

subcollection: java

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}

# Tolérance aux pannes avec Spring
{: #spring-tolerance}

La création d'un système résilient place des exigences sur tous les services qu'il inclut. La nature dynamique des environnements de cloud exige que ces services soient conçus pour être préparés et répondre aux imprévus.

Spring utilise la bibliothèque de tolérance aux pannes [Netflix Hystrix](https://github.com/Netflix/Hystrix/wiki){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe") pour prendre en charge les problème de tolérance aux pannes au niveau de l'application. Hystrix offre un support en terme de rétromigrations, de disjoncteur, de cloison et de mesures associées. 

Ces informations reposent sur les pratiques de tolérances aux pannes décrites dans [Développement cloud natif : Tolérance aux pannes](/docs/java?topic=cloud-native-fault-tolerance#fault-tolerance).
{: note}

## Utilisation d'Hystrix avec une application Spring
{: #spring-hystrix-starter}

Pour que Hystrix soit plus facile à utiliser au sein d'une application Spring, le module de démarrage Spring Boot active une approche annotée pour intégrer Hystrix dans votre application.

L'ajout de cette simple dépendance à votre application initiera Hystrix. 

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```
{: codeblock}

Comme beaucoup d'autres modules de démarrage Spring, pour indiquer à Spring que vous souhaitez utiliser la fonctionnalité dans l'application, vous devez ajouter une annotation à votre classe d'application principale. Pour activer le traitement par Spring des annotations Hystrix pour la coupure de circuit, ajoutez `@EnableCircuitBreaker`.

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

Pour ajouter un comportement de coupure de circuit à une méthode, vous devez l'annoter avec `@HystrixCommand`. Spring trouvera ces méthodes au sein d'une application annotées avec l'annotation `@EnableCircuitBreaker` et les encapsulera avec la fonctionnalité Hystrix pour surveiller les erreurs ou dépassements de délai d'attente, et appellera un comportement alternatif au besoin. 

`@HystrixCommand` est uniquement pris en charge sur les méthodes dans un `@Component` ou un `@Service` {: note}

L'annotation `@HystrixCommand` suivante encapsule l'invocation `service()` pour fournir le comportement de coupure de circuit. Le proxy appellera la méthode `fallback()` si la méthode `service()` échoue ou si le circuit est ouvert.

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

Lorsqu'un service distant est appelé, il est possible qu'il ne soit pas retourné immédiatement, que l'opération prenne un certain temps, ou qu'elle n'aboutisse jamais. Hystrix vous offre la possibilité de définir ce qui est acceptable pour votre application. Une simple addition à l'annotation `HystrixCommand` peut être utilisée pour remplacer la valeur par défaut du délai d'attente par 1 seconde :

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

Hystrix prend en charge à la fois les cloisons sémaphores et les cloisons basées sur les files d'attente. Le fragment suivant montre comment configurer une cloison basée sur une file d'attente qui alloue 4 unités d'exécution et limite le nombre de demandes en suspens à 10 :

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

Le module de démarrage Hystrix Spring possède une astuce supplémentaire : il améliore le noeud final par défaut de l'application, `/health` (fourni via un actionneur Spring). Pour plus d'informations, voir la rubrique [Métriques](/docs/java?topic=java-spring-metrics#spring-metrics).

Si le noeud final d'intégrité est [configuré pour inclure des détails supplémentaires ](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-endpoints.html#production-ready-health){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe"), le statut du disjoncteur sera inclus dans les informations de diagnostic d'intégrité (ce comportement est désactivé par défaut).

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
