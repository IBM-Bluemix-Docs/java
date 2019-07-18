---

copyright:
  years: 2019
lastupdated: "2019-04-22"

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

# 環境定義及相依關係注入（CDI、JSR-365）
{: #mp-cdi}

CDI 在 Jakarta-EE 及 MicroProfile 中都是一個中心元素，因為它用來將各種元件連接在一起。CDI 定義**環境定義**來指定及管理 Bean 生命週期，並使用**相依關係注入**，以動態及安全類型方式，滿足已宣告的相依關係。CDI 也使用鬆散耦合事件和攔截程式模型，並提供功能強大且靈活的可攜式延伸機制，供其他架構定義自己的 CDI Bean 或更新現有的元件。

MicroProfile 規格（例如，MicroProfile Config、MicroProfile JWT）採用 CDI 優先方法，根據 CDI 延伸機制，以新功能來加強現有的標準（例如，JAX-RS）。CDI 1.2 是 MicroProfile 1.0 版及之後版本的一部分（MicroProfile 2.0 會移至 CDI 2.0）。

## CDI Bean
{: #cdi-bean}

CDI 在 _Bean_ 上操作。CDI Bean 是使用 [Bean 定義註釋](https://docs.jboss.org/cdi/spec/2.0/cdi-spec.html){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示") 來建立的。幾乎所有的一般舊 Java&trade; 物件 (POJO) 都可以是 Bean，該 POJO 具有不含引數的建構子，或者具有含 `@Inject` 註釋的建構子。

必須探索定義 CDI Bean 的註釋。可以藉由使用 `META-INF` 資料夾（適用於 .jar 保存檔）或 `WEB-INF` 資料夾（適用於 .war 保存檔）中的 `beans.xml` 檔，來啟用 CDI 註釋掃描。此檔案可以是空的，也可以包含類似於下列的內容：

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

存在空的 `beans.xml` 檔，或者 `beans.xml` 具有 `bean-discovery-mode="all"`，都會使所有潛在的物件變成 CDI Bean。否則，只有 CDI Bean 定義註釋的物件才是 CDI Bean。

## CDI 範圍
{: #cdi-scopes}

CDI 範圍類型註釋會控制 Bean 的生命週期：

* 如果只要應用程式作用中，實例就會存活，請使用 `giedApplicationScoped` 類別層次註釋。
* 如果要為每個要求（例如，Servlet 要求）建立新的 Bean 實例，請使用 `@RequestScoped` 類別層次註釋。
* 如果新實例屬於另一個物件，請使用 `@Dependent` 類別層次註釋。相依 Bean 的實例永遠不會在不同的用戶端或不同的注入點之間共用。它會在其所屬的物件建立時進行實例化，然後在其所屬的物件遭到破壞時毀損。
* 如果未定義類別層次註釋，則 `@Dependent` 為預設值。

CDI 容器會根據已定義的範圍來建立和毀損 Bean 實例，使它們與適當的環境定義產生關聯，然後將它們當作相依關係注入到其他物件中。

## CDI Bean 注入
{: #cdi-inject}

CDI 會透過 `@Inject`，將已定義的 Bean 注入至其他元件。例如，下列 POJO `MyBean` 為 CDI Bean：

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

因為它是 `@ApplicationScoped`，所以只會建立一個實例。在下列程式碼中，`MyRestEndPoint` 是 `@RequestScoped`，這表示會為每個要求建立一個實例。CDI 會將相同的 `MyBean` 實例注入至每一個當中。

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

CDI 正在管理這些 Bean 的生命週期：

* 使用標註了 `@PostConstruct` 的方法，而非使用建構子來起始設定 Bean。在 CDI 容器將相依關係和相關聯的 Bean 注入至適當的環境定義之後，便會呼叫它。
* 使用標註了 `@PreDestroy` 的方法來清除您的 Bean。如此便會在移除相依關係之前呼叫它。
