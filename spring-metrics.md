---

copyright:
  years: 2019, 2020
lastupdated: "2020-03-19"

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
{:external: target="_blank" .external}

# Metrics with Spring
{: #spring-metrics}

As of Spring Framework 5, metrics in Spring are now handled by Micrometer. Micrometer is a framework that describes itself as "SLF4J for Metrics". Just like SLF4J acts as a vendor-neutral API for logging that connects to various logging backends, Micrometer provides a vendor-neutral API that can instrument and measure your code. Then, it can supply those metrics onwards to metrics aggregators, such as Prometheus, DataDog, InfluxDB, and Telegraf. 

The Micrometer framework allows Spring to integrate into a wide variety of cloud-native architectures. Adding support for Prometheus or Statsd is a simple matter of modifying dependencies, and if the metrics collector is push-based, providing destination information in `application.properties`. Spring and Micrometer determine what to do at runtime based on which dependencies they find on the application's class path.

## Enabling Metrics
{: #spring-metrics-enabling}

Basic metrics are supplied by the Spring Boot Actuator, which requires the `spring-boot-starter-actuator` dependency in your `pom.xml` file:  

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-actuator</artifactId>
</dependency>
```
{: codeblock}

If you are still on Spring Boot 1.5.x, then metrics (and actuators) work a little differently. Micrometer was made the standard for Spring metrics in Spring Boot 2.0, but backports were made available for Boot 1.5.x that enable consistent practices for creating and gathering metrics. More importantly, Micrometer supports various monitoring systems, including Prometheus, that make it a better choice than the Boot 1.5.x metrics system for cloud deployments.
{: note}

## Micrometer metrics
{: #spring-metrics-micrometer}

Micrometer abstracts the concept of a destination for metrics through its interface `MeterRegistry`. The `MeterRegistry` provides the way for the app to obtain custom counters, gauges, and timers. If multiple destinations are required, Micrometer supplies a `CompositeMeterRegistry` implementation, allowing the app to be isolated from the actual configured destinations for metrics.

In either version of Spring Boot, when a Micrometer-based metrics endpoint is enabled, Spring Boot automatically creates a `CompositeMeterRegistry`, and makes it available to the app as a `Bean`. The registry is automatically populated with the supported `MeterRegistry` implementations that are found on the class path. Supporting multiple monitoring systems, or switching between them, is then a simple matter of altering dependencies in the pom.xml. 

Spring allows for customization of the `MeterRegistry` bean through a `MeterRegistryCustomizer`, which is itself, a bean. When Spring identifies the presence of such a bean, it can configure the global `MeterRegistry` before any metrics are reported. This customization is often used to add common tags to the `MeterRegistry`, DataCenter, or EnvironmentType.

When you name metrics, Spring and Micrometer recommend following a naming convention that splits words with a `.` rather than using camel case or `_`, or other approaches. Naming is important because Micrometer is relaying those metrics to one or more configured destinations, and performs any name conversions as needed to meet the requirements of the destinations.

## Configuring metrics with Spring Boot
{: #spring-metrics-configuration}

The following sections outline how to enable Spring Boot Actuator metrics with Micrometer. Learn to collect metrics, and provide an endpoint for Prometheus (for each release) starting with Spring Boot 2 as the most current.

### Configuring metrics in Spring Boot 2
{: #spring-metrics-boot2}

The Spring Boot 2.0 Actuator metrics endpoint must be both enabled and exposed for web access. By default the endpoint is enabled, but not exposed (check [the Spring Endpoint Documentation](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-features.html#production-ready-endpoints){: external} for more information).

Add the following to `application.properties` to expose the metrics endpoint:

```properties
management.endpoints.web.exposure.include=health,metrics
```
{: codeblock}

Verify that the metrics endpoint is enabled by checking the `/actuator` endpoint. The output looks something like the following when you run the app locally and use `jq` to pretty print the JSON response:

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

Access the `/actuator/metrics` endpoint to display a JSON formatted list of available metrics similar to the following output:
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

Another request is required to see the actual value of an individual metric (as can be inferred from the list of actuator URLs). To retrieve the jvm thread states, for example:

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

For more information about how to customize the behavior of the metrics actuator, see the [official documentation](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-features.html#production-ready-metrics){: external}.

### Enabling Prometheus support in Spring Boot 2
{: #spring-prometheus-boot2}

Prometheus requires the app to host an endpoint that the Prometheus Server reads from to obtain the metrics. The hosting of this endpoint in Spring relies on the app to serve the data. Which means that the Prometheus endpoint supports Spring Boot apps that use Spring MVC, Spring WebFlux, or Jersey.

To enable the Prometheus endpoint, add the following dependency in `pom.xml`:
```xml
<dependency>
  <groupId>io.micrometer</groupId>
  <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```
{: codeblock}

Spring sees the dependency, and automatically configures `PrometheusMeterRegistry` as one of the configured registries in the global `MeterRegistry` bean. However, as with the metrics endpoint, the Prometheus endpoint must be enabled and exposed for web access by adding the following to `application.properties`:

```properties
management.endpoints.web.exposure.include=health,metrics,prometheus
```
{: codeblock}

Verify that the Prometheus endpoint is enabled by checking the `/actuator` endpoint. The output looks something like the following output if you run the app locally:

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

The `/actuator/prometheus` endpoint emits all configured metrics in Prometheus scrape format:

```
$ curl -s localhost:8080/actuator/prometheus
# HELP system_cpu_usage The "recent cpu usage" for the whole system
# TYPE system_cpu_usage gauge
system_cpu_usage 0.046231032221805926
...
```
{: screen}

Add the following annotations to your service definition to enable Prometheus scraping for the metrics endpoint:

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

### Configuring metrics in Spring Boot 1.0
{: #spring-metrics-boot1}

The `/metrics` endpoint of the Spring Boot Actuator is enabled by default in Spring Boot 1, but is considered “sensitive” and requires authorization. For some environments, securing the metrics endpoint might be the right answer, but authorizing each request to the metrics endpoint introduces too much latency for automated checking with Kubernetes.

Disable the sensitive attribute of the metrics endpoint to remove the authorization requirement by adding the following to `application.properties`:
```properties
endpoints.metrics.sensitive=false
```
{: codeblock}

Now the default Spring Boot 1.0 metrics support is enabled, which produces the following output:
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

For more information about how to customize the behavior of the Spring Boot 1.0 metrics actuator, see the [official documentation](https://docs.spring.io/spring-boot/docs/1.5.x/reference/html/production-ready-metrics.html){: external}.

### Enabling Prometheus support in Spring Boot 1.0
{: #spring-prometheus-boot1}

Prometheus support is not built into the Spring Boot 1.0 Actuator metrics, but can be added with Micrometer.

As mentioned in the [overview](#spring-metrics), Micrometer was backported to Spring Boot 1 because it has a richer set of meter primitives and better support for monitoring systems like Prometheus and StatsD. Using Micrometer with Spring Boot 1 is straight-forward: it requires a specific adapter library, and at least one registry. Add the following to your `pom.xml` to include dependencies for the adapter library and the Prometheus registry:

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

These dependencies create and expose a `/prometheus` metrics endpoint. As with the default `/metrics` endpoint, the `/prometheus` endpoint is considered sensitive by default. To remove the authorization requirement, add the following to `application.properties`:

```properties
endpoints.prometheus.sensitive=false
```
{: codeblock}

The output that is produced by the `/prometheus` endpoint is similar to Spring Boot 2:

```
$ curl -s localhost:8080/prometheus
# HELP jvm_memory_used_bytes The amount of used memory
# TYPE jvm_memory_used_bytes gauge
jvm_memory_used_bytes{area="heap",id="PS Survivor Space",} 2.2594152E7
...
```
{: screen}

Add the following annotations to your service definition to enable Prometheus scraping for the metrics endpoint:

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

For more information about using Micrometer with Spring Boot 1, see [https://micrometer.io/docs/ref/spring/1.5](https://micrometer.io/docs/ref/spring/1.5){: external}.

## Timing invocations with Micrometer
{: #spring-metrics-timing}

Spring MVC, WebFlux, and Jersey Server each provide auto-configuration for Spring that automatically collects timing information for all requests when the property `management.metrics.web.server.auto-time-requests` is set to `true`. The default setting is `true`.

```properties
management.metrics.web.server.auto-time-requests=true
```
{: codeblock}

Micrometer also supports the `@Timed` annotation, which can be used to collect timing information for supported method calls. It's worth noting that this annotation cannot be used to time arbitrary methods, as it requires container support to collect the data. `@Timed` annotations are supported for operations that are backed by a servlet container, for example, Spring MVC, Webflux, or Jersey service methods.

To be more selective about what timing information is collected, set the `management.metrics.web.server.auto-time-requests` property to `false`, and use the `@Timed` annotation on individual service methods, or controllers:

```java
@Timed
@GetMapping("/myendpoint")
public String getSomething() { ... }
```
{: codeblock}

If `management.metrics.web.server.auto-time-requests` is `true`, the `@Timed` annotation can be used to add extra tags to a given reported metric, for example:

```java
@Timed(extraTags = { "mytagname", "mytagvalue"})
@GetMapping("/myendpoint")
public String getSomething() { ... }
```
{: codeblock}

## Custom Metrics with Spring and Micrometer
{: #spring-metrics-custom}

To make application-domain-specific measurements, you need to create your own custom metrics. Micrometer defines a few basic `Meter` types:

* A `Counter` tracks a value that can increase.
* A `Gauge` measures and returns the observed value when the meter is published (or queried).
* A `Timer` tracks how many times an event occurs, and the cumulative elapsed time for that event. 
* A `DistributionSummary` is similar to a `Timer`, but it also tracks a distribution of events, and does not measure time.

New Meters are created by using helper methods on the `MeterRegistry`, or by using the Meter's fluent builder. 

Create your `Meter` in the constructor (passing the `MeterRegistry` as a parameter) or in a `@PostConstruct` method after the `MeterRegistry` is injected.

For example, to use a `Counter` within a `RestController` you can do something like the following;

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

Then, within your service methods, you can update the meter value:

```java
@GetMapping("/")
public String getSomething() {
    myCounter.increment();
    ...
}
```
{: codeblock}


To learn more about creating custom metrics in Spring:

* [Spring Boot Metrics](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-features.html#production-ready-metrics){: external}
* [Micrometer concepts](https://micrometer.io/docs/concepts){: external}
