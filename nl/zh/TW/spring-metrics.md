---

copyright:
  years: 2019
lastupdated: "2019-04-23"

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

# Spring 的度量
{: #spring-metrics}

從 Spring Framework 5 開始，Spring 中的度量現在由 Micrometer 處理。Micrometer 是一個架構，說明其本身為 "SLF4J for Metrics"。就像 SLF4J 作為用於記載的供應商中立 API，可連接至各種不同的記載後端，Micrometer 也提供一個供應商中立 API，用來檢測和測量您的程式碼，然後將這些度量提供給各種度量聚集器，例如，Prometheus、DataDog 或 Influx/Telegraph。

Micrometer 架構可讓 Spring 整合至各種雲端原生架構。只要新增 Prometheus 或 Statsd 支援，即可簡單地修改相依關係，如果度量值收集器是推送型，請在 `application.properties` 中提供目的地資訊。Spring 及 Micrometer 會根據在應用程式的類別路徑上所找到的相依關係，來決定運行環境所要執行的動作。

## 啟用度量
{: #spring-metrics-enabling}

Spring Boot Actuator 會提供基本度量，其需要 `pom.xml` 檔中的 `spring-boot-starter-actuator` 相依關係：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-actuator</artifactId>
</dependency>
```
{: codeblock}

如果您仍在 Spring Boot 1.5.x 上，則度量（和掣動器）的工作方式會有點不同。在 Spring Boot 2.0 中，Micrometer 為 Spring 度量值的標準，但可以對 Boot 1.5.x 使用向後移植，以便在雲端原生 Spring Boot 應用程式中，針對建立與收集度量值的作業，啟用一致的實務。更重要的是，Micrometer 支援各種監視系統，包括 Prometheus，使其比 Boot 1.5.x 度量系統更適合用於雲端部署。
{: note}

## Micrometer 度量
{: #spring-metrics-micrometer}

Micrometer 會透過其介面 `MeterRegistry`，來提供度量目的地的概念摘要。`MeterRegistry` 提供的方法可讓應用程式取得自訂計數器、量規及計時器。如果需要多個目的地，Micrometer 會提供 `CompositeMeterRegistry` 實作，容許從度量的實際配置目的地隔離應用程式。

在 Spring Boot 任一版本中，當啟用 Micrometer 型度量端點時，Spring Boot 會自動建立 `CompositeMeterRegistry`，並使它可供應用程式當作 `Bean` 使用。登錄會自動移入在類別路徑中找到的受支援 `MeterRegistry` 實作。支援多個監視系統（或在它們之間切換），即可簡單地在 pom.xml 中變更相依關係。

Spring 容許透過本身就是 Bean 的 `MeterRegistryCustomizer` 自訂 `MeterRegistry` Bean。當 Spring 識別到這類 Bean 存在時，會在報告任何度量之前，先配置廣域 `MeterRegistry`。這通常用來將一般標籤新增至 `MeterRegistry`、DataCenter 或 EnvironmentType。

命名度量時，Spring 及 Micrometer 建議遵循命名慣例，使用 `.` 來分割單字，而非使用駝峰式大小寫或 `_`，或是其他方法。這是因為 Micrometer 會將那些度量轉遞至已配置的目的地，且會視需要來執行任何名稱轉換，以符合目的地的需求。

## 使用 Spring Boot 配置度量
{: #spring-metrics-configuration}

下列各節概述如何使用 Micrometer 來啟用 Spring Boot Actuator 度量，以收集度量值，並為每個版本的 Prometheus 提供一個端點，從 Spring Boot 2 開始作為最新版本。

### 在 Spring Boot 2 中配置度量
{: #spring-metrics-boot2}

必須同時啟用和公開 Spring Boot 2 Actuator 度量端點，才能進行 Web 存取。依預設，端點會啟用，但不會公開（如需相關資訊，請查閱 [Spring 端點文件](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-endpoints.html){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")）。  

新增下列程式碼至 `application.properties`，以公開度量端點：

```properties
management.endpoints.web.exposure.include=health,metrics
```
{: codeblock}

透過檢查 `/actuator` 端點來驗證是否已啟用度量端點。當您在本端執行應用程式，並使用 `jq` 來細緻列印 Json 回應時，輸出會看起來如下所示：

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

存取 `/actuator/metrics` 端點會顯示 JSON 格式的可用度量清單，其看起來如下所示：

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

需要其他要求才能查看個別度量的實際值（因為可從掣動器 URL 的清單中推斷）。比方說，若要擷取 jvm 執行緒狀態：

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

如需如何自訂度量掣動器行為的相關資訊，請參閱[官方文件](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-metrics.html){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")。

### 在 Spring Boot 2 中啟用 Prometheus 支援
{: #spring-prometheus-boot2}

Prometheus 需要應用程式管理 Prometheus Server 從中讀取以取得度量的端點。在 Spring 中管理此端點須仰賴已具備提供資料能力的應用程式，這表示 Prometheus 端點僅支援使用 Spring MVC、Spring WebFlux 或 Jersey 的 Spring Boot 應用程式。

若要啟用 Prometheus 端點，請在 `pom.xml` 中新增下列相依關係：

```xml
<dependency>
  <groupId>io.micrometer</groupId>
  <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```
{: codeblock}

Spring 會注意相依關係，並自動將 PrometheusMeterRegistry 配置為廣域 MeterRegistry Bean 中已配置的登錄之一。不過，如同度量端點，必須透過將下列程式碼新增至 application.properties 的方式，來啟用並公開 Prometheus 端點，以進行 Web 存取：

```properties
management.endpoints.web.exposure.include=health,metrics,prometheus
```
{: codeblock}

藉由檢查 `/actuator` 端點來驗證是否已啟用 Prometheus 端點。在本端執行應用程式時，輸出看起來如下所示：

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

`/actuator/prometheus` 端點將以 Prometheus 提取格式發出所有已配置的度量：

```
$ curl -s localhost:8080/actuator/prometheus
# HELP system_cpu_usage The "recent cpu usage" for the whole system
# TYPE system_cpu_usage gauge
system_cpu_usage 0.046231032221805926
...
```
{: screen}

將下列註釋新增至服務定義，以針對度量端點啟用 Prometheus 提取：

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

### 在 Spring Boot 1 中配置度量
{: #spring-metrics-boot1}

在 Spring Boot 1 中，依預設啟用 Spring Boot Actuator 的 `/metrics` 端點，但視為「機密」且需要授權。對於部分環境，保護度量端點的安全可能是正確的答案，但是，授權度量端點的每個要求，會對使用 Kubernetes 進行自動檢查的這個作業帶來太多延遲。透過新增下列至 application.properties 的方法，來停用度量端點的機密屬性，以移除授權需求：

```properties
endpoints.metrics.sensitive=false
```
{: codeblock}

這會啟用預設的 Spring Boot 1 度量值支援，它會產生看起來如下所示的輸出：

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

如需如何自訂 Spring Boot 1 度量掣動器行為的相關資訊，請參閱[官方文件](https://docs.spring.io/spring-boot/docs/1.5.x/reference/html/production-ready-metrics.html){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")。

### 在 Spring Boot 1 中啟用 Prometheus 支援
{: #spring-prometheus-boot1}

Prometheus 支援未內建在 Spring Boot 1 Actuator 度量中，但我們可以透過 Micrometer 來新增 Prometheus 支援。

如[概觀](#spring-metrics)中所提及，Micrometer 已向後移植到 Spring Boot 1，因為它有一組更豐富的計量基本元素，並對監視系統（例如，Prometheus 及 StatsD）提供更好的支援。搭配使用 Micrometer 與 Spring Boot 1 是非常直接明確的：它需要特定的配接器程式庫，以及至少一個登錄。將下列程式碼新增至您的 `pom.xml`，以併入配接器程式庫與 Prometheus 登錄的相依關係：

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

這些相依關係會建立並公開 `/prometheus` 度量端點。如同預設的`/metrics` 端點，依預設，`/prometheus` 端點會被視為機密。若要移除授權需求，請將下列程式碼新增至 application.properties：

```properties
endpoints.prometheus.sensitive=false
```
{: codeblock}

`/prometheus` 端點所產生的輸出類似 Spring Boot 2 的輸出：

```
$ curl -s localhost:8080/prometheus
# HELP jvm_memory_used_bytes The amount of used memory
# TYPE jvm_memory_used_bytes gauge
jvm_memory_used_bytes{area="heap",id="PS Survivor Space",} 2.2594152E7
...
```
{: screen}

將下列註釋新增至服務定義，以針對度量端點啟用 Prometheus 提取：

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

Spring MVC、WebFlux 及 Jersey Server 每個都提供自動配置 Spring，可在 `management.metrics.web.server.auto-time-requests` 內容設為 `true` 時，自動收集所有要求的計時資訊。此為預設值。


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

如果 `management.metrics.web.server.auto-time-requests` 為 `true`，則 `@Timed` 註釋可用來將額外的標籤新增至給定的報告度量，例如：

```java
@Timed(extraTags = { "mytagname", "mytagvalue"})
@GetMapping("/myendpoint")
public String getSomething() { ... }
```
{: codeblock}

## Spring 及 Micrometer 的自訂度量
{: #spring-metrics-custom}

若要建立應用程式網域特定的測量，您需要建立自己的自訂度量。Micrometer 定義一些基本的 `Meter` 類型：

* `Counter` 可追蹤只能增加的值。
* `Gauge` 可測量並在發佈（或查詢）計量時，傳回已觀察的值。
* `Timer` 可追蹤事件發生的次數，以及該事件的累計經歷時間。 
* `DistributionSummary` 類似 `Timer`，但也會追蹤事件的配送，且不會測量時間。

新「計量」的建立方式是在 `MeterRegistry` 上使用 helper 方法，或使用「計量」的 fluent 建置器。 

在應用程式碼中工作時，在注入 `MeterRegistry` 之後，於建構子中（將 `MeterRegistry`當作參數傳遞）或於 `@PostConstruct` 方法中，建立您的 `Meter`。

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


若要進一步瞭解在 Spring 中建立自訂度量：

* [Spring Boot Metrics](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-metrics.html){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")
* [Micrometer concepts](https://micrometer.io/docs/concepts){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")
