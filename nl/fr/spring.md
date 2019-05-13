---

copyright:
  years: 2019
lastupdated: "2019-04-22"

keywords: spring framework, spring reference, spring boot, boot actuator, spring kubernetes

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

# Spring
{: #spring-overview}

La plateforme Spring est un écosystème de projets visant à simplifier la création d'applications Java. Habituellement, le terme "Spring" fait référence à Spring Framework, mais il est également utilisé de manière générique pour désigner tout projet qui fait partie de, ou utilise des technologies basées sur Spring, y compris Spring Boot.

## Spring Framework
{: #spring-framework}

Spring Framework est sorti en 2002 et depuis cette date, une version majeure a été publiée environ tous les trois ans. L'infrastructure comprend un large ensemble de composants (appelés modules Spring) qui couvrent tout, des noeuds finaux REST aux abstractions de base de données. Dans sa dernière version datant de 2017, Spring Framework 5 inclut une prise en charge actualisée de JDK, ainsi que WebFlux. WebFlux est un nouveau module pour la programmation réactive s'appuyant sur [Project Reactor](https://projectreactor.io/){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe").

Spring Framework 4.3 est la dernière branche de fonctions pour Spring Framework 4.x, avec une prise en charge jusqu'en 2020. Il est recommandé de faire migrer les applications qui utilisent des versions plus anciennes de Spring vers Spring Framework 5.

Vous trouverez une documentation complète de Spring Framework ici :

* [Guide de référence de Spring Framework 5.1.x](https://docs.spring.io/spring/docs/5.1.x/spring-framework-reference/){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe")
* [Guide de référence de Spring Framework 4.3.x](https://docs.spring.io/spring/docs/4.3.x/spring-framework-reference/){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe")
* [Mise à niveau à Spring Framework 5.x](https://github.com/spring-projects/spring-framework/wiki/Upgrading-to-Spring-Framework-5.x){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe")

## Spring Boot
{: #spring-boot}

Spring Boot a été lancé en 2014, pour "améliorer les architectures d'applications web sans conteneur". Son but était de faire passer la configuration du XML au code. Spring Boot fournit des mécanismes de création d'applications basés sur une analyse raisonnée des technologies à utiliser. Les applications Spring Boot sont des applications Spring et elles utilisent un modèle de programmation unique à cet écosystème.

Spring Boot met l'accent sur les conventions plutôt que sur la configuration, et utilise une combinaison d'annotations et d'opérations de reconnaissance de chemins de classes pour activer des fonctions supplémentaires. Par défaut, les applications Spring Boot sont intégrées dans un fichier JAR exécutable autonome qui inclut toutes les dépendances requises (y compris un serveur Tomcat intégré). Les applications peuvent également être packagées sous forme de fichiers WAR déployés sur un serveur d'application.

Spring Boot en est actuellement à la version 2.0, sortie en 2018. La maintenance de la branche 1.5.x cessera en août 2019.

La documentation complète de Spring Boot est disponible ici :

* [Guide de référence de Spring Boot 2.1.x](https://docs.spring.io/spring-boot/docs/2.1.x/reference/){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe")
* [Guide de référence de Spring Boot 1.5.x](https://docs.spring.io/spring-boot/docs/1.5.x/reference/){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe")

## Sélection de Spring Framework ou Spring Boot
{: #spring-framework-or-boot}

Pour les nouvelles applications natives du cloud, si vous choisissez d'utiliser Spring Framework, vous devez également utiliser Spring Boot. Spring Boot simplifie la configuration et l'assemblage d'applications basées sur Spring, et offre des fonctionnalités, comme Spring Boot Actuator, qui simplifient la création d'[applications natives du cloud](/docs/java?topic=cloud-native-overview#overview).

### Spring Boot Actuator
{: #spring-boot-actuator}

L'actionneur Spring Boot Actuator définit un ensemble de noeuds finaux qui sont utiles pour inspecter une application Spring en cours d'exécution. Les activer est aussi simple que d'ajouter une dépendance à l'application.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```
{: codeblock}

La structure et le comportement de l'actionneur Spring Boot Actuator ont considérablement changé entre Spring Boot 1 et Spring Boot 2. L'actionneur Boot 1 était indépendant de l'application, avec des mécanismes de configuration et de sécurité séparés. La version Boot 2 est plus performante et utilise un modèle de sécurité qui s'intègre avec le reste de l'application Spring.

Les changements de comportement sont particulièrement importants si vous migrez une application Boot 1 vers Boot 2. Lisez la section Actuator du [Guide de migration de Boot 1 à Boot 2](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.0-Migration-Guide#spring-boot-actuator){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe") pour plus de détails.
{: tip}

Dans Boot 2, l'actionneur agit en tant qu'infrastructure mini-REST. Tous les noeuds finaux se trouvent sous un contexte `/actuator`. Des noeuds finaux sont fournis pour effectuer des diagnostics d'intégrité ou recueillir des métriques, des applications ou d'autres informations sur l'environnement. Pour obtenir une liste complète, consultez la [documentation d'Actuator](https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/html/production-ready-features.html#production-ready){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe").

Pour créer votre propre noeud final Actuator, il vous suffit d'utiliser un élément `@Component` Spring :

```java
@Endpoint(id="example")
@Component
public class Example {
    @ReadOperation
    public String example() {
        return "{\"example\":\"value\"}";
    }
}
```
{: codeblock}

Spring Boot dispose de deux commandes pour configurer quels actionneurs sont actifs au moment de l'exécution. Les actionneurs peuvent être "activés" et "exposés". Pour être accessible, un actionneur doit être à la fois activé et exposé. Par défaut, de nombreux noeuds finaux sont activés, mais seuls 'health' et 'info' sont exposés. De plus, si Spring Security est activé pour l'application, l'accès doit être explicitement autorisé sur les noeuds finaux de l'actionneur.

L'actionneur dans Spring Boot 1 a sa propre configuration de sécurité, généralement configurée dans application.properties. Au lieu de "activé" et "exposé", on voit "activé" et "sensible". Les noeuds finaux sensibles nécessitent une authentification s'ils sont servis via HTTP. Par défaut, le noeud final /metrics de l'actionneur Spring Boot est activé dans Spring Boot 1, mais il est considéré comme sensible et nécessite donc une autorisation, ou d'être défini comme non sensible. Pour certains environnements, la configuration de Spring Security peut être la bonne réponse, mais dans Kubernetes, les noeuds finaux /metrics restent internes au cluster. Pour plus d'informations, consultez les [documentations sur l'actionneur Spring Boot 1](https://docs.spring.io/spring-boot/docs/1.5.2.RELEASE/reference/htmlsingle/#production-ready){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe").
{: note}

## Spring Cloud
{: #spring-cloud}

Spring Cloud est une collection d'intégrations entre les technologies de cloud tiers et le modèle de programmation Spring. Son but est d'assister les développeurs qui créent des applications Spring en vue d'un déploiement dans le cloud. L'étendue de la couverture atteinte par Spring Cloud est rendue possible par sa nature modulaire. En effet, chaque projet au sein de Spring Cloud est centré sur une technologie ou un ensemble de technologies particulières.

Les projets Spring Cloud suivent l'approche générale de Spring qui consiste à privilégier les conventions plutôt que la configuration. La plupart des fonctionnalités sont activées en ajoutant la dépendance adéquate au moment de la compilation.

Voir le [site officiel Projets de Spring Cloud](https://spring.io/projects/spring-cloud){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe") pour plus d'informations sur les projets Spring Cloud et les technologies prises en charge.

### Spring Cloud avec Kubernetes
{: #spring-cloud-kubernetes}

Spring Cloud Kubernetes est un projet Spring Cloud destiné à faciliter diverses tâches pour les applications Spring fonctionnant dans Kubernetes. Spring Cloud Kubernetes propose des implémentations d'interfaces Spring communes qui correspondent aux concepts et services de Kubernetes.

Historiquement, Spring utilise les bibliothèques Netflix comme Eureka en tant que registre de services et Ribbon en tant qu'équilibreur de charge côté client. Dans les environnements Kubernetes, ces deux rôles sont assumés par Kubernetes lui-même, ce qui rend superflues les fonctionnalités au niveau applicatif. Spring Cloud Kubernetes adapte les abstractions Spring comme `DiscoveryClient` aux mécanismes Kubernetes sous-jacents.

Soyez prudent si vous utilisez la reconnaissance Ribbon via Spring Cloud Kubernetes dans un cluster avec Istio installé, où Istio est utilisé pour affecter le routage des services. L'équilibrage de charge côté client implique que le client souhaite sélectionner le service de destination lui-même, tandis que Istio peut tenter d'acheminer l'appel ailleurs. Une seule de ces solutions est possible, ce qui entraîne un désaccord sur la façon dont un appel peut être acheminé. Cette combinaison doit être évitée dans la mesure du possible.
{: note}

En outre, Spring Cloud Kubernetes offre l'intégration entre Kubernetes ConfigMaps et Secrets aux beans de configuration Spring `@Autowired`. Cela inclut des stratégies pour gérer les modifications dynamiques apportées à `ConfigMaps` et aux `Secrets` pendant l'exécution de l'application.

Enfin, Spring Cloud Kubernetes augmente le noeud final d'intégrité par défaut de Spring Boot Actuator pour inclure des informations supplémentaires relatives au déploiement.

Pour plus d'informations, consultez la [documentation de Spring Cloud Kubernetes](https://cloud.spring.io/spring-cloud-static/spring-cloud-kubernetes/2.1.0.RC1/single/spring-cloud-kubernetes.html){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe")


<!--
### Spring Cloud Streams
{: #spring-cloud-streams}


:FIXME:
-->
