---

copyright:
  years: 2018, 2019
lastupdated: "2019-06-10"

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

# 配置 Spring 環境
{: #spring-configuration}

服務配置和認證（服務連結）的管理因平台而異。Cloud Foundry 將服務連結詳細資料儲存至字串化 JSON 物件，其會作為 `VCAP_SERVICES` 環境變數傳遞給應用程式。Kubernetes 則是將服務連結儲存成 `ConfigMap` 或 `Secret` 中的字串化 JSON 或平面屬性，然後作為環境變數傳遞給容器化應用程式，或裝載成暫存磁區。本端開發可能有所不同。在沒有環境特定程式碼路徑的情況下，以可攜方式在這些變動因素下作業，相當具有挑戰性。

## 向現有 Spring應用程式新增 {{site.data.keyword.cloud_notm}} 配置
{: #spring-cloud-env}

[{{site.data.keyword.cloud_notm}} Spring Service Bindings](https://github.com/ibm-developer/ibm-cloud-spring-bind){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示") 程式庫透過對不同雲端環境中的資訊進行摘要，為 Spring 應用程式提供了一致的配置存取方式。這會將本端、Cloud Foundry、Cloud Foundry Enterprise Environment、Kubernetes、虛擬機器和 {{site.data.keyword.openwhisk}} 的資訊包含到 `mappings.json` 檔案中的搜尋型樣陣列。Spring 應用程式啟動時，程式庫會讀取該檔案並套用這些型樣，以便探索和指派配置值。

如需 `mappings.json` 檔的相關資訊，請參閱[使用注入的服務認證](/docs/java?topic=cloud-native-configuration#portable-credentials)。

### 向 Spring 應用程式新增程式庫
{: #spring-add-service-library}

若要使用 {{site.data.keyword.cloud_notm}} Spring Service Bindings 程式庫，請在 `pom.xml` 或 `build.gradle` 檔案中包括 `ibm-cloud-spring-boot-service-bind` 相依關係：

**Maven 範例**：

```xml
<dependency>
   <groupId>com.ibm.cloud</groupId>
   <artifactId>ibm-cloud-spring-boot-service-bind</artifactId>
   <version>1.1.1</version>
</dependency>
```
{: codeblock}

**Gradle 範例**：

```json
dependencies {
    compile group: 'com.ibm.cloud', name: 'ibm-cloud-spring-boot-service-bind', version: '1.1.1'
}
```
{: codeblock}

### 存取認證
{: #spring-access-credentials}

若要存取程式碼中的服務配置，您可以使用 `@Value` 註釋，或使用 Spring 架構「環境」類別 `getProperty()` 方法。

範例：存取 IBM Cloudant NoSQL DB 服務的配置資料：

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

## 後續步驟
{: #spring-config-next-steps notoc}

如需 Spring Boot 的相關資訊：

* [Spring Boot 1.5.x](https://docs.spring.io/spring-boot/docs/1.5.x/reference/html/){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")
* [Spring Boot 2.1.x](https://docs.spring.io/spring-boot/docs/2.1.x/reference/html/){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")

若要瞭解有關 {{site.data.keyword.IBM}} 和 Spring 的更多資訊，請參閱：

* [IBM Developer: Spring](https://developer.ibm.com/technologies/spring/){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")
* [IBM Spring Service Bindings](https://github.com/ibm-developer/ibm-cloud-spring-bind){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")
