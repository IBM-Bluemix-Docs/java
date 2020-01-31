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
{:external: target="_blank" .external}

# Configuration with MicroProfile
{: #mp-configuration}

The MicroProfile Config API enables application configuration properties to be stored in multiple sources so you can write your code without environment-specific code paths. The API uses [CDI](/docs/java?topic=java-mp-cdi#mp-cdi) to inject value specified properties into your application.

## Prerequisites
{: #mp-config-prereq}

To use the MicroProfile Config API in your application, you need to add the following dependencies to your `pom.xml` file:

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

## Injecting Configuration
{: #mp-config-inject}

Within your CDI bean class, use the following imports:

```java
//CDI 1.2
import javax.inject.Inject;

//mpConfig 1.2
import org.eclipse.microprofile.config.inject.ConfigProperty;
```
{: codeblock}

To inject the value of a property named `WATSON_URL` into the `watsonService` variable, see this example:

```java
@Inject 
@ConfigProperty(name = "WATSON_URL") 
String watsonService;
```
{: codeblock}

You can also specify a default value that is used if the named property can't be found:

```java
@Inject 
@ConfigProperty(name = "WATSON_URL", defaultValue = "https://gateway.watsonplatform.net/tone-analyzer/api/v3/tone?version=2017-09-21") 
String watsonService;
```
{: codeblock}

And that's all there is to it!

For more information about MicroProfile Config, see:

* [Eclipse MicroProfile Config – what is it?](https://www.eclipse.org/community/eclipse_newsletter/2017/september/article3.php){: external}
* OpenLiberty Guide: [Configuring microservices](https://openliberty.io/guides/microprofile-config.html){: external}
