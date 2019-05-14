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

# 使用 MicroProfile 進行配置
{: #mp-configuration}

MicroProfile Config API 可讓應用程式配置內容儲存在多個原始檔，讓您可以撰寫程式碼，而不需環境特定的程式碼路徑。API 使用 [CDI](/docs/java?topic=java-mp-cdi#mp-cdi)，將值指定內容注入至您的應用程式。

## 必要條件
{: #mp-config-prereq}

若要在應用程式中使用 MicroProfile Config API，您需要將下列相依關係新增至您的 `pom.xml` 檔：

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

## 注入配置
{: #mp-config-inject}

在 CDI Bean 類別內，使用下列匯入項目：

```java
//CDI 1.2
import javax.inject.Inject;

//mpConfig 1.2
import org.eclipse.microprofile.config.inject.ConfigProperty;
```
{: codeblock}

若要將名為 `WATSON_URL` 的內容值注入至 `watsonService` 變數，請使用下列範例：

```java
@Inject 
@ConfigProperty(name = "WATSON_URL") 
String watsonService;
```
{: codeblock}

您也可以指定在找不到具名內容時所使用的預設值：

```java
@Inject 
@ConfigProperty(name = "WATSON_URL", defaultValue = "https://gateway.watsonplatform.net/tone-analyzer/api/v3/tone?version=2017-09-21") 
String watsonService;
```
{: codeblock}

這就是它的所有內容！

如需 MicroProfile Config 的相關資訊，請參閱：

* [Eclipse MicroProfile Config – what is it?](https://www.eclipse.org/community/eclipse_newsletter/2017/september/article3.php){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")
* OpenLiberty Guide: [Configuring microservices](https://openliberty.io/guides/microprofile-config.html){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")
