---

copyright:
  years: 2019
lastupdated: "2019-03-15"

keywords: fault tolerance microprofile, retries microprofile, circuit breakers microprofile, bulkhead microprofile, microprofile limits

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

# Tolérance aux pannes avec MicroProfile
{: #mp-fault-tolerance}

Notre approche recommandée pour ces questions de tolérance aux pannes consiste principalement à tirer parti des fonctionnalités d'Istio. Voir "Istio -- Tolérance aux pannes" dans le chapitre "Création de microservices -- Fonctionnalités Polyglot" de ce livre. Cette section couvre des éléments tels que les nouvelles tentatives, les délais d'attente, les disjoncteurs, les cloisons et les limites de débit.

MicroProfile propose également une approche qui est décrite dans le chapitre "Caractéristiques Java supplémentaires à noter" sous la rubrique "Tolérance aux pannes de MicroProfile". Cette section montre comment la capacité de rétromigration peut être utilisée conjointement avec Istio, et offre des détails pour ceux qui sont intéressés par une approche qui utilise mpFaultTolerance au lieu d'Istio.

Istio n'est pas en mesure d'offrir des fonctions de rétromigration, car la rétromigration exige une connaissance du métier. Une approche plus avancée consiste à utiliser la tolérance aux pannes d'Istio avec les capacités de rétromigration de MicroProfile pour atteindre le maximum de résilience. Par exemple, vous pouvez spécifier une sauvegarde de secours lorsque vous appelez un service de back end. Si Istio ne parvient pas à obtenir un retour réussi, la méthode de rétromigration sera invoquée.

```java
@GET
@Fallback(fallbackMethod="myFallback")
public String callService() {
    //call servicer b
    return  bclient.hello();
}

public String myFallback() {
    return "Greetings\... This is a fallback!";
}
```
{: codeblock}

Si vous utilisez la fonction de tolérance aux pannes d'Istio, vous pouvez désactiver la tolérance aux pannes de MicroProfile en définissant la propriété de configuration MP_Fault_Tolerance_NonFallback_Enabled sur false.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: service-config
data:
  serviceB_host: serviceb-service
  serviceB_http_port: "8080"
  lifetime: "0"
  failFrequency: "0"
  MP_Fault_Tolerance_NonFallback_Enabled: "false"
```
{: codeblock}

Pour plus d'informations sur la comparaison de la tolérance aux pannes d'Istio et de la tolérance aux pannes de MicroProfile, veuillez consulter [ce blogue](https://www.eclipse.org/community/eclipse_newsletter/2018/september/MicroProfile_istio.php){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe") écrit par Emily Jiang.
