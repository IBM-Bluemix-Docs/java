---

copyright:
  years: 2018, 2019
lastupdated: "2019-04-09"

keywords: spring environment, spring credentials, ibm-cloud-spring-boot-service-bind, service bindings spring, vcap_services spring, access credential spring

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

# Configuration de l'environnement Spring
{: #spring-configuration}

La gestion de la configuration de service et des données d'identification (liaisons de service) varie entre les plateformes. Cloud Foundry stocke les détails de liaison du service dans un objet JSON converti en chaîne qui est transmis à l'application en tant que variable d'environnement `VCAP_SERVICES`. Kubernetes stocke les liaisons de service sous forme d'attributs JSON convertis en chaîne ou à plat dans `ConfigMaps` ou `Secrets`, lesquels peuvent être soit transmis à l'application conteneurisée en tant que variables d'environnement, soit montés en tant que volumes temporaires. Le développement local sera variable. Il peut être difficile d'alterner entre ces différentes variations de manière portable sans disposer de chemins de code spécifiques à un environnement.

## Ajout d'une configuration {{site.data.keyword.cloud_notm}} à des applications Spring existantes
{: #spring-cloud-env}

La bibliothèque [Spring Service Bindings d'{{site.data.keyword.cloud_notm}}](https://github.com/ibm-developer/ibm-cloud-spring-bind){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe") offre à votre application Spring un moyen cohérent d'accéder à la configuration en extrayant des informations de différents environnements cloud, y compris les environnements locaux, Cloud Foundry, Cloud Foundry Enterprise Environment, Kubernetes, les machines virtuelles et {{site.data.keyword.openwhisk}} vers un tableau de modèles de recherche dans un fichier `mappings.json`. Lorsque l'application Spring démarre, la bibliothèque lit le fichier et applique les modèles afin de reconnaître et d'affecter les valeurs de configuration.

Pour en savoir plus sur le fichier `mappings.json`, voir [Working with injected service credentials](/docs/java?topic=cloud-native-configuration#portable-credentials).

### Ajout de la bibliothèque à votre application Spring
{: #spring-add-service-library}

Pour utiliser la bibliothèque {{site.data.keyword.cloud_notm}} Spring Service Bindings, incluez la dépendance `ibm-cloud-spring-boot-service-bind` dans votre fichier `pom.xml` ou `build.gradle` :

**Exemple Maven **:

```xml
<dependency>
   <groupId>com.ibm.cloud</groupId>
   <artifactId>ibm-cloud-spring-boot-service-bind</artifactId>
   <version>1.1.1</version>
</dependency>
```
{: codeblock}

**Exemple Gradle **:

```json
dependencies {
    compile group: 'com.ibm.cloud', name: 'ibm-cloud-spring-boot-service-bind', version: '1.1.1'
}
```
{: codeblock}

### Accès aux données d'identification
{: #spring-access-credentials}

Pour accéder à la configuration de service dans votre code, vous pouvez utiliser l'annotation `@Value`, ou utiliser la méthode `getProperty()` de la classe d'environnement d'infrastructure Spring.

Exemple : Accès aux données de configuration pour un service de base de données NoSQL IBM Cloudant :

```java
   // Use the @Value annotation 
   @Value("cloudant.url")
   String cloudantUrl;

   // Use the Spring Environment object 
   @Autowired
   Environment env;

   String cloudantUsername = 
        env.getProperty("cloudant.username");
```
{: codeblock}

## Etapes suivantes
{: #spring-config-next-steps notoc}

Pour en savoir plus sur Spring Boot :

* [Spring Boot 1.5.x](https://docs.spring.io/spring-boot/docs/1.5.x/reference/html/){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe")
* [Spring Boot 2.1.x](https://docs.spring.io/spring-boot/docs/2.1.x/reference/html/){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe")

Pour en savoir plus sur IBM et Spring :

* [IBM Developer : Spring](https://developer.ibm.com/technologies/spring/){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe")
* [Liaisons de service IBM Spring](https://github.com/ibm-developer/ibm-cloud-spring-bind){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe")
