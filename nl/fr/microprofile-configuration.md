---

copyright:
  years: 2019
lastupdated: "2019-04-22"

keywords: microprofile api, microprofile cdi, microprofile config api, config api, store properties multiple sources

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

# Configuration avec MicroProfile
{: #mp-configuration}

L'API MicroProfile Config permet de stocker les propriétés de configuration des applications dans plusieurs sources, ce qui vous permet d'écrire votre code sans chemin de code spécifique à l'environnement. L'API utilise [CDI](/docs/java?topic=java-mp-cdi#mp-cdi) pour injecter des propriétés spécifiées par valeur dans votre application.

## Conditions requises
{: #mp-config-prereq}

Pour utiliser l'API MicroProfile Config dans votre application, vous devez ajouter les dépendances suivantes à votre fichier `pom.xml` :

```xml
<dependency>
  <groupId>org.eclipse.microprofile</groupId>
  <artifactId>microprofile</artifactId>
  <version>2.1</version>
  <type>pom</type>
  <scope>provided</scope>
</dependency>
```
{: codeblock}

## Injection de configuration
{: #mp-config-inject}

Dans votre classe de bean CDI, utilisez les importations suivantes :

```java
//CDI 1.2
import javax.inject.Inject;

//mpConfig 1.2
import org.eclipse.microprofile.config.inject.ConfigProperty;
```
{: codeblock}

Pour injecter la valeur d'une propriété nommée `WATSON_URL` dans la variable `watsonService`, voir cet exemple :

```java
@Inject 
@ConfigProperty(name = "WATSON_URL") 
String watsonService;
```
{: codeblock}

Vous pouvez également spécifier une valeur par défaut qui est utilisée si la propriété nommée est introuvable :

```java
@Inject 
@ConfigProperty(name = "WATSON_URL", defaultValue = "https://gateway.watsonplatform.net/tone-analyzer/api/v3/tone?version=2017-09-21") 
String watsonService;
```
{: codeblock}

Et voilà !

Pour plus d'informations sur MicroProfile Config, voir :

* [Eclipse MicroProfile Config – what is it?](https://www.eclipse.org/community/eclipse_newsletter/2017/september/article3.php){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe")
* Guide OpenLiberty : [Configuring microservices](https://openliberty.io/guides/microprofile-config.html){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe")
