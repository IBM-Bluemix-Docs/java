---

copyright:
  years: 2019
lastupdated: "2019-03-15"

keywords: spring framework, spring reference, spring boot, boot actuator, spring kubernetes

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

# Spring
{: #spring-overview}

「Spring 平台」是一個專案生態系統，旨在讓您更容易建立 Java 應用程式。"Spring" 這個術語通常是指 Spring Framework，但一般也指隸屬於 Spring 或使用 Spring 型技術（包括 Spring Boot）的任何專案。

## Spring Framework
{: #spring-framework}

Spring Framework 於 2002 年推出，之後大約每 3 年發佈一個新的主要版本。架構包含一組大型元件（稱為「Spring 模組」），涵蓋從「REST 端點」到「資料庫摘要」的所有元件。在 2017 年的最新版本，Spring Framework 5 引進更新的 JDK 支援並隨附 WebFlux，此為新模組，根據 [Project Reactor](https://projectreactor.io/){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示") 進行反應程式設計。

Spring Framework 4.3 是最後一個特性分支，支援到 2020 年。使用舊版 Spring 的應用程式應該移轉至 Spring Framework 5。

Spring Framework 的綜合性文件如下：

* [Spring Framework 5.1.x 參考手冊](https://docs.spring.io/spring/docs/5.1.x/spring-framework-reference/){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")
* [Spring Framework 4.3.x 參考手冊](https://docs.spring.io/spring/docs/4.3.x/spring-framework-reference/){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")
* [升級至 Spring Framework 5.x](https://github.com/spring-projects/spring-framework/wiki/Upgrading-to-Spring-Framework-5.x){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")

## Spring Boot
{: #spring-boot}

Spring Boot 已在 2014 年推出，以「改善無容器 Web 應用程式架構」。它的目標是將配置從 XML 移至程式碼。Spring Boot 提供一個機制，可根據個人認為應該使用哪些技術的觀點來建立應用程式。Spring Boot 應用程式是 Spring 應用程式，使用該生態系統獨有的程式設計模型。

Spring Boot 強調使用慣例更勝於配置，並使用註釋和類別路徑探索的組合來啟用其他功能。依預設，Spring Boot 應用程式會建置到自行包含的可執行 JAR 檔，其中包含所有必要的相依關係（包括內嵌 Tomcat 伺服器）。或者，應用程式也可以包裝成部署至應用程式伺服器的 WAR 檔。

Spring Boot 現在為 2.0 版，已於 2018 年發行。1.5.x 分支的維護將於 2019 年 8 月停止。

Spring Boot 的綜合性文件如下：

* [Spring Boot 2.1.x 參考手冊](https://docs.spring.io/spring-boot/docs/2.1.x/reference){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")
* [Spring Boot 1.5.x 參考手冊](https://docs.spring.io/spring-boot/docs/1.5.x/reference/){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")

## 選取 Spring Framework 或 Spring Boot
{: #spring-framework-or-boot}

對於新的雲端原生應用程式，如果您選擇使用 Spring Framework，您也應該使用 Spring Boot。Spring Boot 會簡化 Spring 型應用程式的配置與組合，並提供 Spring Actuator 等特性，以簡化建立[雲端原生應用程式](/docs/cloud-native?topic=cloud-native-overview#overview)的作業。

### Spring Boot Actuator
{: #spring-boot-actuator}

Spring Boot Actuator 定義一組有助於檢查執行中 Spring 應用程式的端點。啟用它們就像將單一個相依關係新增至應用程式一樣簡單。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```
{: codeblock}

Spring Boot Actuator 的結構與行為在 Spring Boot 1 與 Spring Boot 2 之間有顯著變更。Boot 1 Actuator 與應用程式搭配使用，但有個別的配置與安全機制。Boot 2 版本的功能更強大，使用的安全模型與其餘 Spring 應用程式整合。

如果您將 Boot 1 應用程式移轉至 Boot 2，則行為變更會特別適切。如需詳細資料，請閱讀 [Boot1 至 Boot 2 移轉手冊](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.0-Migration-Guide#spring-boot-actuator){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示") 中的「掣動器」小節。
{: tip}

在 Boot 2 中，「掣動器」用來作為迷你 REST 架構。所有端點都在 `/actuator` 環境定義下。提供用於執行性能檢查，或收集度量、應用程式或其他環境資訊的端點。如需完整清單，請查閱[掣動器文件](https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/html/production-ready-features.html#production-ready){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")。

您可以輕易使用 Spring `@Component` 來建立自己的「掣動器」端點：

```java
@Endpoint(id="example")
@Component
public class Example {
    @ReadOperation
    public String example() {
        return "{\"example\":\"value\"}";
    }
}
```
{: codeblock}

Spring Boot 有兩個控制項可配置運行環境中有哪些「掣動器」在作用中，「掣動器」可為 "enabled" 及 "exposed"。「掣動器」必須兩者都是，才能夠呼叫到。依預設，會啟用許多端點，但只會公開性能及資訊。此外，如果應用程式已啟用 Spring Security，則必須明確地容許「掣動器」端點進行存取。

Spring Boot 1 中的「掣動器」有自己的安全配置，通常在 application.properties 中配置。其為 "enabled" 和 "sensitive"，而非 "enabled" 和 "exposed"。如果透過 HTTP 提供機密端點，則需要鑑別。依預設，在 Spring Boot 1 中，會啟用 Spring Boot Actuator 的 /metrics 端點，但被視為機密，因此需要授權，或設為非機密。對於部分環境，配置 Spring Security 可能才是正確的做法，但在 Kubernetes 中，度量端點仍在叢集內部。如需相關資訊，請查閱 [Spring Boot 1 Actuator 文件](https://docs.spring.io/spring-boot/docs/1.5.2.RELEASE/reference/htmlsingle/#production-ready){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")。
{: note}

## Spring Cloud
{: #spring-cloud}

Spring Cloud 是協力廠商雲端技術與 Spring 程式設計模型之間的整合集合。它旨在協助開發人員建置 Spring 應用程式以部署至雲端。Spring Cloud 有可能因為其模組本質而涵蓋範圍廣泛，Spring Cloud 內的每個專案都著重於一個特定技術或一組技術。

Spring Cloud 專案遵循一般的 Spring 方法，相較於配置，更偏好於使用慣例。大部分功能都是透過在建置時期新增正確的相依關係來啟用。

如需 Spring Cloud 專案及支援的技術的相關資訊，請參閱官方的 [Spring Cloud 專案網站](https://spring.io/projects/spring-cloud){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")。

### 具有 Kubernetes 的 Spring Cloud
{: #spring-cloud-kubernetes}

Spring Cloud Kubernetes 是一個 Spring Cloud 專案，用來讓在 Kubernetes 中執行的 Spring 應用程式更容易執行各種作業。Spring Cloud Kubernetes 提供一般 Spring 介面的實作，這些介面對映至 Kubernetes 概念及服務。

Spring 過去使用 Eureka 等 Netflix 程式庫作為「服務登錄」，並使用 Ribbon 作為用戶端「負載平衡器」。在 Kubernetes 環境中，這兩個角色都由 Kubernetes 本身履行，導致應用程式層次功能變得多餘。Spring Cloud Kubernetes 將 `DiscoveryClient` 等 Spring 摘要調整為基礎 Kubernetes 機制。

在已安裝 Istio 並使用 Istio 來影響服務遞送的叢集中，如果透過 Spring Cloud Kubernetes 來使用 Ribbon 探索，則應多加留意。用戶端負載平衡意味著用戶端想要自行選取目的地服務，而 Istio 可能正在嘗試將呼叫遞送至其他位置。其中只有一個獲勝，導致對呼叫的遞送方式產生分岐。如果可能的話，應避免這種組合。
{: note}

此外，Spring Cloud Kubernetes 還提供 Kubernetes ConfigMaps 和 Secrets 至 Spring `@Autowired` 配置 Bean 之間的整合。這包含一些策略，用來處理在執行應用程式時對 ConfigMaps 與 Secrets 所做的動態變更。

最後，Spring Cloud Kubernetes 會擴增預設的 Spring Boot Actuator 性能端點，以包含與部署相關的其他資訊。

如需相關資訊，請查閱 [Spring Cloud Kubernetes 文件](https://cloud.spring.io/spring-cloud-static/spring-cloud-kubernetes/2.1.0.RC1/single/spring-cloud-kubernetes.html){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")


<!--
### Spring Cloud Streams
{: #spring-cloud-streams}


:FIXME:
-->
