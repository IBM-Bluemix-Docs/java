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

从 Spring Framework 5 开始，Spring 中的度量值现在由 Micrometer 处理。Micrometer 是一个将自身描述为“用于度量值的 SLF4J”的框架。就像 SLF4J 充当独立于供应商的 API 以连接到各种不同的日志记录后端进行日志记录一样，Micrometer 提供的也是独立于供应商的 API，可用于检测和度量代码。然后，可以将这些度量值提供给度量值聚集器，例如 Prometheus、DataDog、InfluxDB 和 Telegraf。

通过 Micrometer 框架，Spring 可以集成到各种云本机体系结构中。添加对 Prometheus 或 Statsd 的支持十分简单，只要修改依赖项即可，并且如果度量值收集器是基于推送的，只需在 `application.properties` 中提供目标信息。Spring 和 Micrometer 根据其在应用程序的类路径上找到的依赖项来确定要在运行时执行的操作。

## 启用度量值
{: #spring-metrics-enabling}

基本度量值由 Spring Boot Actuator 提供，这需要 `pom.xml` 文件中有 `spring-boot-starter-actuator` 依赖项：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-actuator</artifactId>
</dependency>
```
{: codeblock}

如果您仍位于 Spring Boot 1.5.x 上，那么度量值（和 Actuator）的工作方式略有不同。Micrometer 已成为 Spring Boot 2.0 中 Spring 度量值的标准，但 Boot 1.5.x 可以使用向后移植来实现创建和收集度量值的一致实践。更重要的是，Micrometer 支持各种监视系统（包括 Prometheus），因此相比 Boot 1.5.x 度量值系统，Micrometer 更适合用于云部署。
{: note}

## Micrometer 度量值
{: #spring-metrics-micrometer}

Micrometer 通过其接口 `MeterRegistry` 抽象出度量值目标的概念。通过 `MeterRegistry`，应用程序可获取定制计数器、标尺和计时器。如果需要多个目标，Micrometer 提供了 `CompositeMeterRegistry` 实现，允许将应用程序与度量值的实际配置目标相隔离。

在任一版本的 Spring Boot 中，启用基于 Micrometer 的度量值端点时，Spring Boot 都会自动创建 `CompositeMeterRegistry`，并使其作为 `Bean` 供应用程序使用。注册表会自动使用在类路径上找到的受支持的 `MeterRegistry` 实现进行填充。支持多个监视系统或在这些系统之间进行切换十分简单，只要在 pom.xml 中更改依赖项即可。

Spring 支持通过 `MeterRegistryCustomizer`（其本身就是 Bean）来定制 `MeterRegistry` Bean。Spring 识别到此类 Bean 存在时，可以在报告任何度量值之前配置全局 `MeterRegistry`。此定制通常用于向 `MeterRegistry`、DataCenter 或 EnvironmentType 添加常用标记。

对度量值命名时，Spring 和 Micrometer 建议遵循命名约定，即采用 `.` 来拆分词，而不是使用驼峰式大小写、`_` 或其他方法。命名非常重要，因为 Micrometer 会将这些度量值中继到一个或多个已配置的目标，并且会根据需要执行任何名称转换以满足目标的需求。

## 使用 Spring Boot 配置度量值
{: #spring-metrics-configuration}

以下部分概述了如何使用 Micrometer 启用 Spring Boot Actuator 度量值。了解如何收集度量值，并以 Spring Boot 2 作为最新版本开始，为 Prometheus（针对每个发行版）提供端点。

### 在 Spring Boot 2 中配置度量值
{: #spring-metrics-boot2}

Spring Boot 2 Actuator 度量值端点必须启用并公开用于 Web 访问。缺省情况下，此端点已启用，但未公开（请参阅 [Spring Endpoint 文档](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-endpoints.html){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标") 以获取更多信息）。  

将以下内容添加到 `application.properties` 以公开度量值端点：

```properties
management.endpoints.web.exposure.include=health,metrics
```
{: codeblock}

通过检查 `/actuator` 端点来验证是否已启用度量值端点。本地运行应用程序并使用 `jq` 采用精美的格式显示 JSON 响应时，输出类似于以下内容：

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

访问 `/actuator/metrics` 端点以显示可用度量值的 JSON 格式列表，类似于以下输出：
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

需要另一个请求才能查看单个度量值的实际值（可以从 Actuator URL 列表推断出）。例如，要检索 JVM 线程状态：

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

有关如何定制度量值 Actuator 行为的更多信息，请参阅[官方文档](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-metrics.html){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")。

### 在 Spring Boot 2 中启用 Prometheus 支持
{: #spring-prometheus-boot2}

Prometheus 需要应用程序托管 Prometheus 服务器从中进行读取以获取度量值的端点。在 Spring 中托管此端点依赖于应用程序来提供数据。这意味着 Prometheus 端点支持使用 Spring MVC、Spring WebFlux 或 Jersey 的 Spring Boot 应用程序。

要启用 Prometheus 端点，请在 `pom.xml` 中添加以下依赖项：
```xml
<dependency>
  <groupId>io.micrometer</groupId>
  <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```
{: codeblock}

Spring 会注意到此依赖项，并自动配置 `PrometheusMeterRegistry`，使其成为全局 `MeterRegistry` Bean 中已配置的某个注册表。但是，与度量值端点一样，Prometheus 端点必须通过向 `application.properties` 添加以下内容来启用并公开用于 Web 访问：

```properties
management.endpoints.web.exposure.include=health,metrics,prometheus
```
{: codeblock}

通过检查 `/actuator` 端点来验证是否已启用该 Prometheus 端点。如果本地运行应用程序，那么输出类似于以下内容：

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

`/actuator/prometheus` 端点将以 Prometheus 抓取格式发出所有配置的度量值：

```
$ curl -s localhost:8080/actuator/prometheus
# HELP system_cpu_usage The "recent cpu usage" for the whole system
# TYPE system_cpu_usage gauge
system_cpu_usage 0.046231032221805926
...
```
{: screen}

将以下注释添加到服务定义，以便为度量值端点启用 Prometheus 抓取：

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

缺省情况下，Spring Boot 1 中启用了 Spring Boot Actuator 的 `/metrics` 端点，但此端点被视为“敏感”端点，因此需要授权。对于某些环境，保护度量值端点可能是正确的应对措施，但是对度量值端点的每个请求都进行授权会使等待时间太长，导致无法通过 Kubernetes 的自动检查。

通过将以下内容添加到 `application.properties`，可禁用度量值端点的敏感属性，以除去授权需求：
```properties
endpoints.metrics.sensitive=false
```
{: codeblock}

现在将启用缺省 Spring Boot 1 度量值支持，生成的输出类似于以下内容：
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

有关如何定制 Spring Boot 1 度量值 Actuator 行为的更多信息，请参阅[官方文档](https://docs.spring.io/spring-boot/docs/1.5.x/reference/html/production-ready-metrics.html){: new_window}![外部链接图标](../icons/launch-glyph.svg "外部链接图标")。

### 在 Spring Boot 1 中启用 Prometheus 支持
{: #spring-prometheus-boot1}

Prometheus 支持未构建到 Spring Boot 1 Actuator 度量值中，但可以通过 Micrometer 添加支持。

如[概述](#spring-metrics)中所述，Micrometer 已向后移植到 Spring Boot 1，因为它具有一组更丰富的计量表原语，并且对监视系统（如 Prometheus 和 StatsD）的支持更好。将 Micrometer 与 Spring Boot 1 配合使用非常简单：Micrometer 需要特定的适配器库，并且至少需要一个注册表。将以下内容添加到 `pom.xml` 以包含适配器库和 Prometheus 注册表的依赖项：

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

这些依赖项将创建并公开 `/prometheus` 度量值端点。与缺省 `/metrics` 端点一样，缺省情况下，`/prometheus` 端点也被视为敏感端点。要除去授权需求，请将以下内容添加到 `application.properties`：

```properties
endpoints.prometheus.sensitive=false
```
{: codeblock}

`/prometheus` 端点生成的输出类似于 Spring Boot 2 的相应内容：

```
$ curl -s localhost:8080/prometheus
# HELP jvm_memory_used_bytes The amount of used memory
# TYPE jvm_memory_used_bytes gauge
jvm_memory_used_bytes{area="heap",id="PS Survivor Space",} 2.2594152E7
...
```
{: screen}

将以下注释添加到服务定义，以便为度量值端点启用 Prometheus 抓取：

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

有关将 Micrometer 与 Spring Boot 1 配合使用的更多信息，请参阅 [https://micrometer.io/docs/ref/spring/1.5 ](https://micrometer.io/docs/ref/spring/1.5){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")。

## 使用 Micrometer 进行计时调用
{: #spring-metrics-timing}

Spring MVC、WebFlux 和 Jersey Server 均为 Spring 提供自动配置，用于在 `management.metrics.web.server.auto-time-requests` 属性设置为 `true` 时，自动收集所有请求的计时信息。缺省设置为 `true`。

```properties
management.metrics.web.server.auto-time-requests=true
```
{: codeblock}

Micrometer 还支持 `@Timed` 注释，此注释可用于收集受支持方法调用的计时信息。值得注意的是，此注释不能用于对任意方法计时，因为它需要容器支持才能收集数据。servlet 容器支持的操作（例如，Spring MVC、Webflux 或 Jersey 服务方法）支持 `@Timed` 注释。

要在收集计时信息时更有选择性，请将 `management.metrics.web.server.auto-time-requests` 属性设置为 `false`，并对各个服务方法或控制器使用 `@Timed` 注释：

```java
@Timed
@GetMapping("/myendpoint")
public String getSomething() { ... }
```
{: codeblock}

如果 `management.metrics.web.server.auto-time-requests` 为 `true`，那么可以使用 `@Timed` 注释向给定的报告度量值添加额外标记，例如：

```java
@Timed(extraTags = { "mytagname", "mytagvalue"})
@GetMapping("/myendpoint")
public String getSomething() { ... }
```
{: codeblock}

## Spring 和 Micrometer 的定制度量值
{: #spring-metrics-custom}

要进行特定于应用程序域的度量，您需要创建自己的定制度量值。Micrometer 定义了多种基本 `Meter` 类型：

* `Counter` 跟踪可能增大的值。
* `Gauge` 在发布（或查询）计量表时度量并返回观察值。
* `Timer` 跟踪某个事件发生的次数以及该事件的累计耗用时间。 
* `DistributionSummary` 类似于 `Timer`，并且它还会跟踪事件的分布，但不会度量时间。

在 `MeterRegistry` 上使用 helper 方法或使用 Meter 的 Fluent 构建器来创建新的 Meter。 

注入了 `MeterRegistry` 后，在构造函数（将 `MeterRegistry` 作为参数传递）或 `@PostConstruct` 方法中创建 `Meter`。

例如，要在 `RestController` 中使用 `Counter`，可以执行类似于下面的操作：

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

然后，在服务方法中，可以更新计量表值：

```java
@GetMapping("/")
public String getSomething() {
    myCounter.increment();
    ...
}
```
{: codeblock}


要了解有关在 Spring 中创建定制度量值的更多信息，请参阅：

* [Spring Boot Metrics](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-metrics.html){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")
* [Micrometer concepts](https://micrometer.io/docs/concepts){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")
