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

# MicroProfile による構成
{: #mp-configuration}

MicroProfile Config API を使用すると、アプリケーション構成プロパティーを複数のソースに保管できるため、環境固有のコード・パスなしでコードを書くことができます。 API は [CDI](/docs/java?topic=java-mp-cdi#mp-cdi) を使用して、値が指定されたプロパティーをアプリケーションに注入します。

## 前提条件
{: #mp-config-prereq}

アプリケーションで MicroProfile Config API を使用するには、`pom.xml` ファイルに次の依存関係を追加する必要があります。

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

## 構成の注入
{: #mp-config-inject}

CDI Bean クラス内で、次のインポートを使用します。

```java
//CDI 1.2
import javax.inject.Inject;

//mpConfig 1.2
import org.eclipse.microprofile.config.inject.ConfigProperty;
```
{: codeblock}

`WATSON_URL` という名前のプロパティーの値を `watsonService` 変数に注入するには、以下の例を参照してください。

```java
@Inject 
@ConfigProperty(name = "WATSON_URL") 
String watsonService;
```
{: codeblock}

名前が指定されたプロパティーが見つからない場合に使用されるデフォルト値を指定することもできます。

```java
@Inject 
@ConfigProperty(name = "WATSON_URL", defaultValue = "https://gateway.watsonplatform.net/tone-analyzer/api/v3/tone?version=2017-09-21") 
String watsonService;
```
{: codeblock}

これで完了です。

MicroProfile Config について詳しくは、以下を参照してください。

* [Eclipse MicroProfile Config とは](https://www.eclipse.org/community/eclipse_newsletter/2017/september/article3.php){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン")
* OpenLiberty ガイド: [マイクロサービスの構成](https://openliberty.io/guides/microprofile-config.html){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン")
