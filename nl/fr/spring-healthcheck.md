---

copyright:
  years: 2018, 2019
lastupdated: "2019-06-10"

keywords: health check spring, spring health endpoint, spring-boot-actuator, liveness probe spring, readiness probe spring, spring kubernetes probe

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

# Diagnostics d'intégrité avec Spring
{: #spring-healthcheck}

Le noeud final d'intégrité fourni par l'actionneur Spring Boot Actuator est lié au cycle de vie de l'application et n'est pas accessible tant que l'application n'a pas été démarrée. De la même façon, il cesse d'être disponible dès que l'application est arrêtée (ce qui se produit avant que le processus ne s'arrête). Spring Boot Actuator ajoute automatiquement des indicateurs de santé pour les technologies détectées dans le chemin d'accès aux classes. Par exemple, si votre application utilise JDBC, le noeud final d'intégrité va inclure des tests de base pour s'assurer que le magasin de données de secours est accessible. Le noeud final d'intégrité défini par l'actionneur est mieux adapté aux sondes de disponibilité pour cette raison.

Activez Spring Boot Actuator en ajoutant la dépendance `spring-boot-actuator` à votre fichier `pom.xml` :

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-actuator</artifactId>
</dependency>
```
{: codeblock}

Il existe des différences de comportement avec Spring Boot Actuator entre les différentes versions. Les deux prochaines sections explorent la façon de créer des vérifications d'activité et de disponibilité dans les deux versions de Spring Boot, en commençant par la version 2 qui est la plus récente.

## Diagnostics d'intégrité de Spring Boot 2
{: #spring-health-boot2}

L'actionneur Spring Boot Actuator 2 définit un noeud final `/actuator/health`, qui renvoie un contenu `{"status": "UP"}` lorsque tout est en ordre. Ce noeud final est activé par défaut et ne nécessite aucun code d'application.

Consultez l'exemple suivant d'un diagnostic d'intégrité non authentifié réussi qui utilise l'actionneur Spring Boot 2 :
<!-- Spring Boot 2 test project: https://github.com/IBM/spring-alarm-application -->

```
$ curl -i localhost:8080/actuator/health
HTTP/1.1 200
Content-Type: application/vnd.spring-boot.actuator.v2+json;charset=UTF-8
Transfer-Encoding: chunked
Date: Fri, 07 Dec 2018 23:09:09 GMT

{"status":"UP"}
```
{: screen}

Comme indiqué, le noeud final d'intégrité renvoie un simple statut ACTIF ou INACTIF par défaut. Pour les diagnostics d'intégrité automatiques, ce simple contenu est généralement suffisant. Vous pouvez activer la génération d'informations d'intégrité détaillées en définissant la propriété suivante dans `application.properties` :

```properties
management.endpoint.health.show-details=always
```
{: codeblock}

```
$ curl -i localhost:8080/actuator/health
HTTP/1.1 200
Content-Type: application/vnd.spring-boot.actuator.v2+json;charset=UTF-8
Transfer-Encoding: chunked
Date: Fri, 07 Dec 2018 23:09:09 GMT

{"status":"UP","details":{"diskSpace":{"status":"UP","details":{"total":500068036608,"free":407611420672,"threshold":10485760}},"db":{"status":"UP","details":{"database":"H2","hello":1}}}}
```
{: screen}

Notez l'inclusion de certaines informations de base de données H2. Cet exemple d'actionneur Spring ajoute des diagnostics automatiquement pour les services de secours. Dans ce cas, l'application utilise JDBC et le pilote H2 a été reconnu sur le chemin d'accès aux classes.

Vous pouvez remplacer le comportement par défaut du noeud final d'intégrité avec des propriétés ou du code, comme décrit dans le [Guide de référence de Spring Boot 2.1.x](https://docs.spring.io/spring-boot/docs/2.1.x/reference/html/production-ready-endpoints.html#production-ready-health){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe").

### Sonde d'activité dans Spring Boot 2
{: #spring-liveness-boot2}

L'infrastructure d'Actuator dans Spring Boot 2 est sa propre mini infrastructure REST qui peut être étendue avec des noeuds finaux personnalisés. En d'autres termes, vous pouvez ajouter un simple noeud final d'activité qui est géré de la même façon que l'indicateur d'intégrité intégré. Un noeud final d'activité personnalisé peut être défini trivialement comme dans l'exemple suivant :

```java
@Endpoint(id="liveness")
@Component
public class Liveness {
    @ReadOperation
    public String testLiveness() {
            return "{\"status\":\"UP\"}";
    }
}
```
{: codeblock}

Ce noeud final personnalisé doit être exposé en tant que noeud final Web :

```properties
management.endpoints.web.exposure.include=health,liveness,...
```
{: codeblock}

```
$ curl -i localhost:8080/actuator/liveness
HTTP/1.1 200
Content-Type: application/vnd.spring-boot.actuator.v2+json;charset=UTF-8
Content-Length: 15
Date: Thu, 07 Feb 2019 22:38:27 GMT

{"status":"UP"}
```
{: screen}

### Configuration des sondes d'activité et de disponibilité de Spring Boot 2 dans Kubernetes
{: #spring-probe-config-2}

Ajoutez une configuration des sondes d'activité et de disponibilité à la définition de conteneur dans un manifeste de déploiement Kubernetes (notez les valeurs du chemin d'accès et du port) :

```yaml
spec:
  containers:
  - name: ...
    image: ...
    readinessProbe:
      httpGet:
        path: /actuator/health
        port: 8080
      initialDelaySeconds: 60
      timeoutSeconds: 5
    livenessProbe:
      httpGet:
        path: /actuator/liveness
        port: 8080
      initialDelaySeconds: 130
      timeoutSeconds: 10
      failureThreshold: 10
```
{: codeblock}

Pour éviter les cycles de redémarrage, réglez `livenessProbe.initialDelaySeconds` sur une durée supérieure à la durée d'initialisation de votre service. Vous pouvez ensuite utiliser une valeur plus courte pour `readinessProbe.initialDelaySeconds` pour acheminer les demandes vers le service dès qu'il est prêt.

## Diagnostics d'intégrité dans Spring Boot 1
{: #spring-health-boot1}

L'actionneur Spring Boot Actuator 1 définit un noeud final `/health` qui renvoie un contenu `{"status": "UP"}` lorsque tout est en ordre. Le noeud final ne nécessite pas d'autorisation mais si une autorisation est configurée et l'appelant est autorisé, des informations additionnelles sont incluses dans la réponse.

Consultez l'exemple suivant d'un diagnostic d'intégrité non authentifié réussi qui utilise l'actionneur Spring Boot 1 :
```
$ curl -i localhost:8080/health
HTTP/1.1 200
X-Application-Context: application
Content-Type: application/vnd.spring-boot.actuator.v1+json;charset=UTF-8
Content-Length: 15

{"status":"UP"}
```
{: screen}

Exemple de diagnostic d'intégrité réussi avec authentification :

```
$ curl -i localhost:8080/health
HTTP/1.1 200 OK
X-Application-Context: application
Content-Type: application/vnd.spring-boot.actuator.v1+json;charset=UTF-8
Content-Length: 221

{
    "status" : "UP",
  "diskSpace" : {
        "status" : "UP",
    "total" : 63251804160,
    "free" : 31316164608,
    "threshold" : 10485760
    },
  "db" : {
        "status" : "UP",
    "database" : "H2",
    "hello" : 1
    }
}
```
{: screen}

Vous pouvez redéfinir le comportement par défaut du noeud final d'intégrité avec des propriétés ou du code, comme décrit dans le [Guide de référence de Spring Boot v1.5.x](https://docs.spring.io/spring-boot/docs/1.5.9.RELEASE/reference/htmlsingle/#_auto_configured_healthindicators){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe").

### Sonde d'activité dans Spring Boot 1
{: #spring-liveness-boot1}

Pour Spring Boot 1, un simple noeud final HTTP pour l'activité, qui renvoie toujours {"status": "UP"} avec le code de statut 200, ressemblerait au fragment suivant :

```java
@RestController
public class Liveness {
    private static Map<String, String> alwaysOk = Collections.singletonMap("status", "OK");

    @GetMapping("/liveness")
    public Map<String,String> testLiveness() {
        return alwaysOk;
    }
}
```
{: codeblock}

```
$ curl -i localhost:8080/liveness
HTTP/1.1 200
Content-Type: application/json;charset=UTF-8
Transfer-Encoding: chunked
Date: Thu, 07 Feb 2019 22:43:32 GMT

{"status":"OK"}
```
{: screen}

### Configuration des sondes d'activité et de disponibilité de Spring Boot 1 dans Kubernetes
{: #spring-probe-config-1}

Ajoutez une configuration des sondes d'activité et de disponibilité à la définition de conteneur dans un manifeste de déploiement Kubernetes (notez les valeurs du chemin d'accès et du port) :

```yaml
spec:
  containers:
  - name: ...
    image: ...
    readinessProbe:
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 60
      timeoutSeconds: 5
    livenessProbe:
      httpGet:
        path: /liveness
        port: 8080
      initialDelaySeconds: 130
      timeoutSeconds: 10
      failureThreshold: 10
```
{: codeblock}

Pour éviter les cycles de redémarrage, réglez `livenessProbe.initialDelaySeconds` sur une durée supérieure à la durée d'initialisation de votre service. Vous pouvez ensuite utiliser une valeur plus courte pour `readinessProbe.initialDelaySeconds` pour acheminer les demandes vers le service dès qu'il est prêt.
