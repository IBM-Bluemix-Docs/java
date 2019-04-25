---

copyright:
  years: 2019
lastupdated: "2019-03-15"

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

# Configuração com MicroProfile
{: #mp-configuration}

A API de configuração do MicroProfile permite que as propriedades de configuração do aplicativo sejam armazenadas em múltiplas origens, permitindo que você grave seu código sem caminhos de código específicos do ambiente. A API usa [CDI](/docs/java?topic=java-mp-cdi#mp-cdi) para injetar as propriedades especificadas do valor.

## Pré-requisito
{: #mp-config-prereq}

Para usar a API de configuração do MicroProfile em seu aplicativo, é necessário incluir as dependências a seguir em seu arquivo `pom.xml`:

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

## Injetando configuração
{: #mp-config-inject}

Dentro da classe de bean de CDI, use as importações a seguir:

```java
//CDI 1.2
import javax.inject.Inject;

//mpConfig 1.2
import org.eclipse.microprofile.config.inject.ConfigProperty;
```
{: codeblock}

Para injetar o valor de uma propriedade denominada `WATSON_URL` na variável `watsonService`, use algo semelhante a isto:

```java
@Inject 
@ConfigProperty(name = "WATSON_URL") 
String watsonService;
```
{: codeblock}

Também é possível especificar um valor padrão que será usado se não for possível localizar a propriedade nomeada:

```java
@Inject 
@ConfigProperty(name = "WATSON_URL", defaultValue = "https://gateway.watsonplatform.net/tone-analyzer/api/v3/tone?version=2017-09-21") 
String watsonService;
```
{: codeblock}

E isso é tudo o que há para ele!

Para obter mais informações sobre a configuração do MicroProfile, consulte:

* [Configuração do MicroProfile Eclipse - O que é?](https://www.eclipse.org/community/eclipse_newsletter/2017/september/article3.php){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo")
* Guia do OpenLiberty: [Configurando microsserviços](https://openliberty.io/guides/microprofile-config.html){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo")
