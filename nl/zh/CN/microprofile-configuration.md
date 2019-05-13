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

# 使用 MicroProfile 进行配置
{: #mp-configuration}

MicroProfile Config API 支持应用程序配置属性存储在多个源中，从而您可以在不使用特定于环境的代码路径的情况下编写代码。该 API 使用 [CDI](/docs/java?topic=java-mp-cdi#mp-cdi) 向应用程序注入值指定的属性。

## 先决条件
{: #mp-config-prereq}

要在应用程序中使用 MicroProfile Config API，需要将以下依赖项添加到 `pom.xml` 文件中：

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

在 CDI Bean 类中，使用以下 import 语句：

```java
//CDI 1.2
import javax.inject.Inject;

//mpConfig 1.2
import org.eclipse.microprofile.config.inject.ConfigProperty;
```
{: codeblock}

要将名为 `WATSON_URL` 属性的值注入 `watsonService` 变量，请参阅以下示例：

```java
@Inject 
@ConfigProperty(name = "WATSON_URL") 
String watsonService;
```
{: codeblock}

此外，您还可以指定在找不到指定的属性时使用的缺省值：

```java
@Inject 
@ConfigProperty(name = "WATSON_URL", defaultValue = "https://gateway.watsonplatform.net/tone-analyzer/api/v3/tone?version=2017-09-21") 
String watsonService;
```
{: codeblock}

这就是关于使用 MicroProfile 进行配置的所有内容！

有关 MicroProfile Config 的更多信息，请参阅：

* [Eclipse MicroProfile Config – what is it?](https://www.eclipse.org/community/eclipse_newsletter/2017/september/article3.php){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")
* OpenLiberty 指南：[Configuring microservices](https://openliberty.io/guides/microprofile-config.html){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")
