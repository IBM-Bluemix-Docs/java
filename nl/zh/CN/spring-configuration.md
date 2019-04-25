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

# 配置 Spring 环境
{: #spring-configuration}

服务配置和凭证（服务绑定）的管理因不同的平台而有所不同。Cloud Foundry 将服务绑定详细信息存储在字符串化的 JSON 对象中，该对象会作为 `VCAP_SERVICES` 环境变量传递到应用程序。Kubernetes 将服务绑定作为字符串化 JSON 或平面属性存储在 `ConfigMap` 或 `Secret` 中，可以将其作为环境变量传递到容器化应用程序，或作为临时卷安装。本地开发将有所不同。在不具备特定于环境的代码路径的情况下，以可移植方式应对这些变化因素可能很困难。

## 向现有 Spring 应用程序添加 {{site.data.keyword.cloud_notm}} 配置
{: #spring-cloud-env}

[{{site.data.keyword.cloud_notm}} Spring Service Bindings](https://github.com/ibm-developer/ibm-cloud-spring-bind){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标") 库通过将不同云环境（包括本地、Cloud Foundry、Cloud Foundry Enterprise Environment、Kubernetes、虚拟机和 {{site.data.keyword.openwhisk}}）中的信息抽象化为 `mappings.json` 文件中的搜索模式数组，为 Spring 应用程序提供了访问配置的一致方式。Spring 应用程序启动时，库会读取该文件并应用这些模式，以便发现和分配配置值。

有关 `mappings.json` 文件的更多信息，请参阅[使用注入的服务凭证](/docs/java?topic=cloud-native-configuration#portable-credentials)。

### 向 Spring 应用程序添加库
{: #spring-add-service-library}

要使用 {{site.data.keyword.cloud_notm}}Spring Service Bindings 库，请在 `pom.xml` 或 `build.gradle` 文件中包含 `ibm-cloud-spring-boot-service-bind` 依赖项：

**Maven 示例**：

```xml
<dependency>
   <groupId>com.ibm.cloud</groupId>
   <artifactId>ibm-cloud-spring-boot-service-bind</artifactId>
   <version>1.1.1</version>
</dependency>
```
{: codeblock}

**Gradle 示例**：

```json
dependencies {
    compile group: 'com.ibm.cloud', name: 'ibm-cloud-spring-boot-service-bind', version: '1.1.1'
}
```
{: codeblock}

### 访问凭证
{: #spring-access-credentials}

要在代码中访问服务配置，可以使用 `@Value` 注释，或使用 Spring 框架 Environment 类的 `getProperty()` 方法。

示例：访问 IBM Cloudant NoSQL DB 服务的配置数据：

```java
   // 使用 @Value 注释
   @Value("cloudant.url")
   String cloudantUrl;

   // 使用 Spring Environment 对象
   @Autowired
   Environment env;

   String cloudantUsername = 
        env.getProperty("cloudant.username");
```
{: codeblock}

## 后续步骤
{: #spring-config-next-steps notoc}

有关 Spring Boot 的更多信息，请参阅：

* [Spring Boot 1.5.x](https://docs.spring.io/spring-boot/docs/1.5.x/reference/html/){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")
* [Spring Boot 2.1.x](https://docs.spring.io/spring-boot/docs/2.1.x/reference/html/){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")

要了解有关 IBM 和 Spring 的更多信息，请参阅：

* [IBM Developer：Spring](https://developer.ibm.com/technologies/spring/){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")
* [IBM Spring Service Bindings](https://github.com/ibm-developer/ibm-cloud-spring-bind){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")
