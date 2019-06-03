---

copyright:
  years: 2019
lastupdated: "2019-05-20"

keywords: health check jax-rs, jax-rs endpoint, jax-rs status, readiness jax-rs, liveness jax-rs, microprofile health

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

# Diagnostics d'intégrité avec JAX-RS
{: #jaxrs-healthcheck}

Comme nous l'avons vu dans la section précédente, Kubernetes propose plusieurs méthodes pour intégrer des sondes d'intégrité, y compris l'exécution d'une commande, ainsi que la vérification du réseau via des noeuds finaux TCP ou HTTP. Bien qu'il soit possible d'implémenter n'importe laquelle de ces sondes dans le langage Java, nous détaillons ici une implémentation JAX-RS des sondes HTTP et MicroProfile Health.

## Définition d'un noeud final JAX-RS de disponibilité
{: #mp-readiness}

Un noeud final de disponibilité simple est similaire à l'exemple suivant :

```java
@Path("ready")
public class ReadinessEndpoint {
  @GET
  @Produces(MediaType.APPLICATION_JSON)
  public Response testReadiness() {
    if ( ! ready ) {
      return Response.status(503).entity("{\"status\":\"DOWN\"}").build();
    }
    return Response.ok("{\"status\":\"UP\"}").build();
  }
}
```
{: codeblock}

Un noeud final pour une application qui est déployée dans WebSphere Liberty atteint un certain niveau de disponibilité sans effort supplémentaire. Liberty rejette les demandes adressées au noeud final d'intégrité avec une réponse 503 appropriée jusqu'à ce que l'application soit en cours d'exécution. Cette implémentation de base doit être étendue pour inclure les fonctions requises. Par exemple, envisagez d'évaluer les ressources CDI `@ApplicationScoped` pour vous assurer que le traitement des injections par dépendance est terminé. Une fois la sonde implémentée, elle peut être activée en configurant le type de sonde HTTP comme indiqué dans la section précédente.

N'oubliez jamais le rôle et l'objectif de la tolérance aux pannes : les microservices sont censés gérer les pannes et les perturbations dans les communications avec les services en aval. Vous devez vérifier les fonctions pour lesquelles il n'y a pas de rétromigration possible dans une vérification de disponibilité.

## Définition d'un noeud final JAX-RS d'activité
{: #jaxrs-liveness}

Une sonde d'activité doit faire l'objet d'une réflexion quant à ce qui vérifié, car toute défaillance entraîne l'arrêt immédiat du processus. Un noeud final d'activité simple se présente comme suit : 

```java
@Path("liveness")
public class LivenessEndpoint {
    @GET
    @Produces(MediaType.APPLICATION_JSON)
    public Response testLiveness() {
      return Response.ok("{\"status\":\"UP\"}").build();
    }
}
```
{: codeblock}

L'implémentation de la sonde de vivacité peut être étendue pour vérifier les conditions locales de traitement qui indiquent un état de traitement irrécupérable. Cependant, la difficulté de ces contrôles réside souvent dans le fait que si ce type de traitement est possible, le serveur est probablement suffisamment sain pour exécuter d'autres requêtes. Pour cette raison, il convient de simplifier au maximum l'implémentation des contrôles d'activité. Une fois que les sources de vivacité sont codées pour activer le noeud final de la sonde, le noeud final de vivacité HTTP peut être activé dans les sources d'orchestration des conteneurs comme expliqué dans le dernier chapitre.

Pour éviter les cycles de redémarrage, l'attribut `initialDelaySeconds` du contrôle d'activité doit être plus long que la plus longue durée de démarrage prévue du serveur. Pour une application Java qui met généralement 30 secondes à démarrer, choisissez une valeur plus élevée, 60 secondes, par exemple.

## Utilisation de l'API MicroProfile Health
{: #mp-health-api}

L'[API MicroProfile Health](https://www.ibm.com/support/knowledgecenter/en/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/twlp_microprofile_healthcheck.html){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe") (mpHealth) définit une infrastructure pour simplifier l'implémentation des diagnostics d'intégrité. La version 1.0 de l'API `mpHealth` ne définit qu'un seul noeud final. La version suivante définira deux noeud finaux, un pour l'activité, et un pour la disponibilité.

Pour utiliser l'API MicroProfile Health avec Liberty, ajoutez la fonction `mpHealth` au fichier `server.xml` :

```xml
<featureManager>
   <feature>mpHealth-1.0</feature>
</featureManager>
```
{: codeblock}

Vous pouvez ensuite vérifier le statut de l'application avec un navigateur en accédant au noeud final `/health`. Le code renvoie un contenu `{"outcome": "UP"}` lorsque tout est correct. Le code suivant montre comment utiliser les API MicroProfile Health pour indiquer si le pod fonctionne correctement. La méthode `call` permet aux développeurs de fournir des algorithmes de contrôle pour indiquer l'état de santé d'un pod. Un manque de mémoire constitue, par exemple, un bon indicateur de la santé d'un pod. La vérification de la connexion à la base de données en aval pourrait être un bon moyen de vérifier si le pod est prêt. S'il n'y a pas de contrôle spécifique, vous pouvez utiliser ce qui suit pour indiquer si un pod est actif ou prêt.

```java
import org.eclipse.microprofile.health.Health;
import org.eclipse.microprofile.health.HealthCheck;
import org.eclipse.microprofile.health.HealthCheckResponse;

import javax.enterprise.context.ApplicationScoped;

@Health
@ApplicationScoped
public class HealthResource implements HealthCheck {

    @Override
    public HealthCheckResponse call() {
        return HealthCheckResponse.named("service-a").withData("service-a", "ok").up().build();
    }
}
```
{: codeblock}

Vous devez ensuite spécifier `livenessProbe` et `readinessProbe` pour Kubernetes, comme indiqué dans l'exemple suivant :
```yaml
livenessProbe:
  exec:
    command: ["sh", "-c", "curl -s http://localhost:9080/health| grep -q service-a"]
  initialDelaySeconds: 60
readinessProbe:
  exec:
    command: ["sh", "-c", "curl -s http://localhost:9080/health | grep -q service-a"]
  initialDelaySeconds: 40
```
{: codeblock}

Comme nous l'avons mentionné précédemment, vous pouvez utiliser différentes logiques pour indiquer la disponibilité et l'activité. Vous pouvez utiliser MicroProfile Health pour indiquer la vérification de disponibilité, puis créer un noeud final JAX-RS pour la vérification de disponibilité ou vice-versa. La prochaine version de MicroProfile Health fournira deux noeuds finaux : un pour l'activité et un pour la disponibilité.

OpenLiberty dispose d'un guide pratique qui explique à l'aide d'un exemple comment ajouter des vérifications personnalisées au noeud final MicroProfile Health : https://openliberty.io/guides/microprofile-health.html

Pour redéfinir le comportement par défaut du noeud final d'intégrité de MicroProfile, consultez la page [Réalisation de bilans de santé dans MicroProfile](https://www.ibm.com/support/knowledgecenter/en/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/twlp_microprofile_healthcheck.html){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe").
