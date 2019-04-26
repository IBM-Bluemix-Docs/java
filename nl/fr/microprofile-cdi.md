---

copyright:
  years: 2019
lastupdated: "2019-03-15"

keywords: dependency injection, cdi jakarta, cdi beans, cdi scopes, bean lifecycle, context injection microprofile, microprofile cdi

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

# Injections de contextes et de dépendances (CDI, JSR-365)
{: #mp-cdi}

CDI est un élément central de Jakarta-EE et de MicroProfile, car il est utilisé pour relier entre eux différents composants. CDI définit des **contextes** pour spécifier et gérer les cycles de vie des beans et utilise l'**injection de dépendances** pour satisfaire les dépendances déclarées de manière dynamique et sûre. CDI utilise également un modèle d'événement et d'intercepteur à couplage lâche, et fournit un mécanisme d'extension portable puissant et flexible pour que d'autres infrastructures puissent définir leurs propres beans CDI ou mettre à jour les composants existants.

Les spécifications MicroProfile (p. ex. MicroProfile Config, MicroProfile JWT, etc.) adoptent l'approche 'CDI-first', en s'appuyant sur le mécanisme d'extension CDI pour améliorer les normes existantes (comme JAX-RS) avec de nouvelles fonctions. CDI 1.2 fait partie de la version 1.0 de MicroProfile et des versions ultérieures (MicroProfile 2.0 passe à CDI 2.0).

## Beans CDI
{: #cdi-bean}

L'injection de contextes et de dépendances (ou CDI) utilise des _beans_. Les beans CDI sont créés à l'aide d'[annotation de définition des beans](https://docs.jboss.org/cdi/spec/2.0/cdi-spec.html){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe"). Presque tous les objets Java simples (POJO) qui ont soit un constructeur sans argument, soit un constructeur avec l'annotation `@Inject` peuvent être un bean.

Les annotations qui définissent les beans dans CDI doivent être reconnues. L'analyse des annotations du CDI peut être activée à l'aide d'un fichier `beans.xml` dans le dossier `META-INF` (pour une archive jar) ou dans le dossier `WEB-INF` (pour une archive war). Ce fichier peut être vide ou peut contenir un contenu tel que :

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://xmlns.jcp.org/xml/ns/javaee"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
            http://xmlns.jcp.org/xml/ns/javaee/beans_1_1.xsd"
      version="1.1" bean-discovery-mode="all">
  // enable interceptors; decorators; alternatives
</beans>
```
{: codeblock}

La présence d'un fichier `beans.xml` vide ou d'un fichier `beans.xml` avec `bean-discovery-mode="all"` fait de tous les objets potentiels des beans CDI. Sinon, seuls les objets comportant des annotations définissant les beans CDI sont des beans CDI.

## Portée du CDI
{: #cdi-scopes}

Les annotations de type portée du CDI contrôlent le cycle de vie des beans :

* Utilisez l'annotation de niveau classe `@ApplicationScoped` si une instance doit vivre aussi longtemps que l'application est exécutée.
* Utilisez l'annotation de niveau classe `@RequestScoped` si une nouvelle instance du bean doit être créée pour chaque demande (une demande Servlet, par exemple).
* Utilisez l'annotation de niveau classe `@Dependent` si la nouvelle instance doit appartenir à un autre objet. Une instance d'un bean dépendant n'est jamais partagée entre différents clients ou différents points d'injection. Il est instancié lorsque l'objet auquel il appartient est créé puis détruit lorsque l'objet auquel il appartient est détruit.
* Si aucune annotation de niveau classe n'est définie, la valeur par défaut est `@Dependent`.

Le conteneur de CDI crée et détruit les instances de bean en fonction de la portée définies. Il les associe au contexte approprié puis les injecte en tant que dépendances dans d'autres objets.

## Injection de beans CDI
{: #cdi-inject}

CDI injectera les beans définis dans d'autres composants via `@Inject`. Par exemple, le bean `MyBean` de l'objet POJO suivant est un bean CDI :

```java
@ApplicationScoped
public class MyBean {
    int i=0;
    public String sayHello() {
        return "MyBean hello " + i++;
    }
}
```
{: codeblock}

Comme `@ApplicationScoped` est défini, une seule instance sera créée. Dans l'exemple suivant, la portée de `MyRestEndPoint` est `@RequestScoped`, ce qui signifie qu'une instance sera créée pour chaque demande. CDI injectera la même instance `MyBean` dans chacune.

```java
@RequestScoped
@Path("/hello")
public class MyRestEndPoint {
    @Inject MyBean bean;

    @GET
    @Produces (MediaType.TEXT_PLAIN)
    public String sayHello() {
        return bean.sayHello();
    }
}
```
{: codeblock}

CDI gère le cycle de vie de ces beans :

* Utilisez une méthode annotée avec `@PostConstruct` au lieu d'un constructeur pour initialiser votre bean. Il sera invoqué après que le conteneur CDI aura injecté les dépendances et les beans associés dans le contexte approprié.
* Utilisez une méthode annotée avec `@PreDestroy` pour nettoyer votre bean. Il sera invoqué avant la suppression des dépendances.
