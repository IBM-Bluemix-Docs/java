---

copyright:
  years: 2019
lastupdated: "2019-06-10"

keywords: spring metrics, configure metrics spring, micrometer spring, micrometer, spring boot 2, spring actuator, prometheus spring

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

# Spring 的度量值
{: #spring-metrics}

從 Spring Framework 5 開始，Spring 中的度量值現在由 Micrometer 處理。Micrometer 是一個架構，說明其本身為 "SLF4J for Metrics"。就像 SLF4J 作為用於記載的供應商中立 API，可連接至各種記載後端，Micrometer 也提供一個供應商中立 API，用來檢測和測量您的程式碼。然後，可以將這些度量值提供給度量值聚集器，例如 Prometheus、DataDog、InfluxDB 和 Telegraf。

Micrometer 架構可讓 Spring 整合至各種雲端原生架構。只要新增 Prometheus 或 Statsd 支援，即可簡單地修改相依關係，如果度量值收集器是推送型，請在 `application.properties` 中提供目的地資訊。Spring 及 Micrometer 會根據在應用程式的類別路徑上所找到的相依關係，來決定運行環境所要執行的動作。

## 啟用度量值
{: #spring-metrics-enabling}

Spring Boot Actuator 會提供基本度量值，其需要 `pom.xml` 檔中的 `spring-boot-starter-actuator` 相依關係：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-actuator</artifactId>
</dependency>
```
{: codeblock}

如果您仍在 Spring Boot 1.5.x 上，則度量值（和掣動器）的工作方式會有點不同。在 Spring Boot 2.0 中，Micrometer 為 Spring 度量值的標準，但可以對 Boot 1.5.x 使用向後移植，以便針對建立與收集度量值的作業，啟用一致的實務。更重要的是，Micrometer 支援各種監視系統，包括 Prometheus，使其比 Boot 1.5.x 度量值系統更適合用於雲端部署。
{: note}

## Micrometer 度量值
{: #spring-metrics-micrometer}

Micrometer 會透過其介面 `MeterRegistry`，來提供度量值目的地的概念摘要。透過 `MeterRegistry`，應用程式可取得自訂計數器、量規和計時器。如果需要多個目的地，Micrometer 提供了 `CompositeMeterRegistry` 實作，容許將應用程式與度量值的實際配置目的地相隔離。

在任一版本的 Spring Boot 中，已啟用 Micrometer 型的度量值端點時，Spring Boot 都會自動建立 `CompositeMeterRegistry`，並使其作為 `Bean` 供應用程式使用。登錄會自動移入在類別路徑中找到的受支援 `MeterRegistry` 實作。支援多個監視系統（或在它們之間切換），即可簡單地在 pom.xml 中變更相依關係。

Spring 容許透過本身就是 Bean 的 `MeterRegistryCustomizer` 自訂 `MeterRegistry` Bean。當 Spring 識別到這類 Bean 存在時，會在報告任何度量值之前，先配置廣域 `MeterRegistry`。此自訂作業通常用來將一般標籤新增至 `MeterRegistry`、DataCenter 或 EnvironmentType。

對度量值命名時，Spring 和 Micrometer 建議遵循命名慣例，即採用 `.` 而非使用駝峰式大小寫或 `_`，或是其他方法。命名非常重要，因為 Micrometer 會將這些度量值轉遞至一個以上已配置的目的地，並且會根據需要執行任何名稱轉換以符合目的地的需求。

## 使用 Spring Boot 配置度量值
{: #spring-metrics-configuration}

下列區段概述了如何使用 Micrometer 啟用 Spring Boot 掣動器度量值。瞭解如何收集度量值，並以 Spring Boot 2 作為最新版本開始，為 Prometheus（針對每個版本）提供端點。

### 在 Spring Boot 2 中配置度量值
{: #spring-metrics-boot2}

必須同時啟用和公開 Spring Boot 2 掣動器度量值端點，才能進行 Web 存取。依預設，端點會啟用，但不會公開（如需相關資訊，請查閱 [Spring 端點文件](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-endpoints.html){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")）。  

新增下列程式碼至 `application.properties`，以公開度量值端點：

```properties
management.endpoints.web.exposure.include=health,metrics
```
{: codeblock}

透過檢查 `/actuator` 端點來驗證是否已啟用度量值端點。當您在本端執行應用程式，並使用 `jq` 來細緻列印 JSON 回應時，輸出會看起來如下所示：

```
$ curl -s localhost:8080/actuator | jq .
{
  "_links": {
    "self": {
      "href": "http://localhost:8080/actuator",
      "templated": false
    },
    ...
    "metrics": {
      "href": "http://localhost:8080/actuator/metrics",
      "templated": false
    },
    "metrics-requiredMetricName": {
      "href": "http://localhost:8080/actuator/metrics/{requiredMetricName}",
      "templated": true
    },
    ...
  }
}
```
{: screen}

存取 `/actuator/metrics` 端點以顯示可用度量值的 JSON 格式清單，類似於下列輸出：
```
$ curl -s localhost:8080/actuator/metrics | jq .
{
  "names": [
    "jvm.memory.max",
    "jvm.threads.states",
    "process.files.max",
    "jvm.gc.memory.promoted",
    ...
  ]
}
```
{: screen}

需要另一個要求才能查看個別度量值的實際值（可以從掣動器 URL 清單推斷出）。比方說，若要擷取 jvm 執行緒狀態：

```
$ curl -s localhost:8080/actuator/metrics/jvm.threads.states | jq .
{
  "name": "jvm.threads.states",
  "description": "The current number of threads having TERMINATED state",
  "baseUnit": "threads",
  "measurements": [
    {
    "statistic": "VALUE",
    "value": 24
    }
  ],
  "availableTags": [
    {
      "tag": "state",
      "values": [
        "timed-waiting",
        "new",
        "runnable",
        "blocked",
        "waiting",
        "terminated"
      ]
    }
  ]
}
```
{: screen}

如需如何自訂度量值掣動器行為的相關資訊，請參閱[官方文件](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-metrics.html){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")。

### 在 Spring Boot 2 中啟用 Prometheus 支援
{: #spring-prometheus-boot2}

Prometheus 需要應用程式管理 Prometheus 伺服器從中進行讀取以取得度量值的端點。在 Spring 中管理此端點仰賴於應用程式來提供資料。這表示 Prometheus 端點支援使用 Spring MVC、Spring WebFlux 或 Jersey 的 Spring Boot 應用程式。

若要啟用 Prometheus 端點，請在 `pom.xml` 中新增下列相依關係：
```xml
<dependency>
  <groupId>io.micrometer</groupId>
  <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```
{: codeblock}

Spring 會注意到此相依關係，並自動配置 `PrometheusMeterRegistry`，使其成為廣域 `MeterRegistry` Bean 中已配置的某個登錄。但是，與度量值端點一樣，Prometheus 端點必須透過向 `application.properties` 新增下列內容來已啟用並公開用於 Web 存取：

```properties
management.endpoints.web.exposure.include=health,metrics,prometheus
```
{: codeblock}

藉由檢查 `/actuator` 端點來驗證是否已啟用 Prometheus 端點。如果本端執行應用程式，則輸出類似於下列內容：

```
$ curl -s localhost:8080/actuator | jq .
{
  "_links": {
    "self": {
      "href": "http://localhost:8080/actuator",
      "templated": false
    },
    ...
    "prometheus": {
      "href": "http://localhost:8080/actuator/prometheus",
      "templated": false
    },
    ...
  }
}
```
{: screen}

`/actuator/prometheus` 端點將以 Prometheus 抓取格式發出所有配置的度量值：

```
$ curl -s localhost:8080/actuator/prometheus
# HELP system_cpu_usage The "recent cpu usage" for the whole system
# TYPE system_cpu_usage gauge
system_cpu_usage 0.046231032221805926
...
```
{: screen}

將下列註釋新增至服務定義，以針對度量值端點啟用 Prometheus 提取：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: ...
  annotations:
    prometheus.io/scrape: 'true'
    prometheus.io/scheme: http
    prometheus.io/path: /actuator/prometheus
    prometheus.io/port: "8080"
  ...
```
{: codeblock}

### 在 Spring Boot 1 中配置度量值
{: #spring-metrics-boot1}

在 Spring Boot 1 中，依預設啟用 Spring Boot 掣動器的 `/metrics` 端點，但視為「機密」且需要授權。對於部分環境，保護度量值端點的安全可能是正確的答案，但是，授權度量值端點的每個要求，會對使用 Kubernetes 進行自動檢查的這個作業帶來太多延遲。

透過將下列內容新增到 `application.properties`，可停用度量值端點的敏感屬性，以移除授權需求：
```properties
endpoints.metrics.sensitive=false
```
{: codeblock}

現在將已啟用預設 Spring Boot 1 度量值支援，產生的輸出類似於下列內容：
```
$ curl -s localhost:8080/metrics | jq .
{
  "names": [
    "jvm.memory.max",
    "jvm.threads.states",
    "process.files.max",
    "jvm.gc.memory.promoted",
    ...
  ]
}
```
{: screen}

如需如何自訂 Spring Boot 1 度量值掣動器行為的相關資訊，請參閱[官方文件](https://docs.spring.io/spring-boot/docs/1.5.x/reference/html/production-ready-metrics.html){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")。

### 在 Spring Boot 1 中啟用 Prometheus 支援
{: #spring-prometheus-boot1}

Prometheus 支援未建置到 Spring Boot 1 掣動器度量值中，但可以透過 Micrometer 新增支援。

如[概觀](#spring-metrics)中所提及，Micrometer 已向後移植到 Spring Boot 1，因為它有一組更豐富的計量基本元素，並對監視系統（例如，Prometheus 及 StatsD）提供更好的支援。將 Micrometer 與 Spring Boot 1 配合使用非常簡單：Micrometer 需要特定的配接器程式庫，並且至少需要一個登錄。將下列程式碼新增至您的 `pom.xml`，以併入配接器程式庫與 Prometheus 登錄的相依關係：

```xml
<properties>
  ...
  <micrometer.version>1.1.1</micrometer.version>
</properties>
<dependencies>
  <dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-spring-legacy</artifactId>
    <version>${micrometer.version}</version>
  </dependency>
  <dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
    <version>${micrometer.version}</version>
  </dependency>
</dependencies>
```
{: codeblock}

這些相依關係將建立並公開 `/prometheus` 度量值端點。如同預設的`/metrics` 端點，依預設，`/prometheus` 端點會被視為機密。若要移除授權需求，請將下列內容新增到 `application.properties`：

```properties
endpoints.prometheus.sensitive=false
```
{: codeblock}

`/prometheus` 端點產生的輸出類似於 Spring Boot 2 的相應內容：

```
$ curl -s localhost:8080/prometheus
# HELP jvm_memory_used_bytes The amount of used memory
# TYPE jvm_memory_used_bytes gauge
jvm_memory_used_bytes{area="heap",id="PS Survivor Space",} 2.2594152E7
...
```
{: screen}

將下列註釋新增至服務定義，以針對度量值端點啟用 Prometheus 提取：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: …
  annotations:
    prometheus.io/scrape: 'true'
    prometheus.io/scheme: http
    prometheus.io/path: /prometheus
    prometheus.io/port: "8080"
  ...
```
{: codeblock}

如需搭配使用 Micrometer 與 Spring Boot 1 的相關資訊，請參閱 [https://micrometer.io/docs/ref/spring/1.5](https://micrometer.io/docs/ref/spring/1.5){: new_window}![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")。

## 使用 Micrometer 的計時呼叫
{: #spring-metrics-timing}

Spring MVC、WebFlux 和 Jersey Server 均為 Spring 提供自動配置，用於在 `management.metrics.web.server.auto-time-requests` 內容設定為 `true` 時，自動收集所有要求的計時資訊。預設設定為 `true`。

```properties
management.metrics.web.server.auto-time-requests=true
```
{: codeblock}

Micrometer 也支援 `@Timed` 註釋，可用來收集所支援方法呼叫的計時資訊。值得注意的是，此註釋不能用來計時任意方法，因為它需要容器支援才能收集資料。Servlet 儲存器所支援的作業，例如，Spring MVC、Webflux 或 Jersey 服務方法，支援 `@Timed` 註釋。

若要更有選擇性地收集計時資訊，請將 `management.metrics.web.server.auto-time-requests` 內容設為 `false`，然後在個別服務方法或控制器上使用 `@Timed` 註釋：

```java
@Timed
@GetMapping("/myendpoint")
public String getSomething() { ... }
```
{: codeblock}

如果 `management.metrics.web.server.auto-time-requests` 為 `true`，則 `@Timed` 註釋可用來將額外的標籤新增至給定的報告度量值，例如：

```java
@Timed(extraTags = { "mytagname", "mytagvalue"})
@GetMapping("/myendpoint")
public String getSomething() { ... }
```
{: codeblock}

## Spring 及 Micrometer 的自訂度量值
{: #spring-metrics-custom}

若要建立應用程式網域特定的測量，您需要建立自己的自訂度量值。Micrometer 定義一些基本的 `Meter` 類型：

* `Counter` 追蹤可能增加的值。
* `Gauge` 可測量並在發佈（或查詢）計量時，傳回已觀察的值。
* `Timer` 追蹤某個事件發生的次數以及該事件的累計經歷時間。 
* `DistributionSummary` 類似 `Timer`，但也會追蹤事件的分佈，且不會測量時間。

新「計量」的建立方式是在 `MeterRegistry` 上使用 helper 方法，或使用「計量」的 fluent 建置器。 

注入了 `MeterRegistry` 後，在建構函數（將 `MeterRegistry` 作為參數傳遞）或 `@PostConstruct` 方法中建立 `Meter`。

例如，若要使用 `RestController` 內的 `Counter`，您可以執行如下動作：

```java
@RestController
public class MyController {
    
    private final Counter myCounter;
    
    public MyController(MeterRegistry meterRegistry) {
    
        // Create the counter using the helper method on the builder
        myCounter = meterRegistry.counter("my.counter", "mytagname", "mytagvalue");
        
    }
```
{: codeblock}

然後，在服務方法內，您可以更新計量值：

```java
@GetMapping("/")
public String getSomething() {
    myCounter.increment();
    ...
}
```
{: codeblock}


若要進一步瞭解在 Spring 中建立自訂度量值：

* [Spring Boot Metrics](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-metrics.html){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")
* [Micrometer concepts](https://micrometer.io/docs/concepts){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")
