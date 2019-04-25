---

copyright:
  years: 2018, 2019
lastupdated: "2019-03-15"

keywords: fault tolerance spring, hystrix spring, netflix spring, hystrixcommand spring, bulkhead spring, circuit breaker spring

subcollection: java

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}

# 使用 Spring 实现容错
{: #spring-tolerance}

创建弹性系统对置于其中的所有服务提出了要求。云环境的动态性质要求服务设计为准备好并正常应对意外事件。

Spring 使用 [Netflix Hystrix](https://github.com/Netflix/Hystrix/wiki){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标") 容错库来支持应用程序级别的容错问题。Hystrix 提供对回退、断路器、隔板和关联度量值的支持。 

此信息基于[云本机开发：容错](/docs/java?topic=cloud-native-fault-tolerance#fault-tolerance)中描述的容错实践而构建。
{: note}

## 将 Hystrix 用于 Spring 应用程序
{: #spring-hystrix-starter}

为了使 Hystrix 更容易在 Spring 应用程序中使用，提供了一个 Spring Boot 入门模板，支持使用带注释的方法在应用程序中集成 Hystrix。

将以下单个依赖项添加到应用程序会引入 Hystrix。 

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```
{: codeblock}

与其他许多 Spring 入门模板类似，要向 Spring 指示您希望在应用程序中使用该功能，请对主应用程序类添加相应注释。要支持 Spring 处理 Hystrix 注释以执行断路操作，请添加 `@EnableCircuitBreaker`。

```java
@SpringBootApplication
@EnableCircuitBreaker
public class MyApplication {

	public static void main(String[] args) {
		SpringApplication.run(MyApplication.class, args);
	}
```
{: codeblock}

### 使用 HystrixCommand 定义回退
{: #spring-fallback}

要向方法添加断路行为，请使用 `@HystrixCommand` 对其进行注释。Spring 将在使用 `@EnableCircuitBreaker` 注释的应用程序中查找这些方法，并使用 Hystrix 功能对其打包以监视错误/超时，并在需要时调用相应的替代行为。 

仅 `@Component` 或 `@Service` 中的方法上支持 `@HystrixCommand`。{: note}

以下 `@HystrixCommand` 注释会对 `service()` 调用打包，以提供断路行为。如果 `service()` 方法失败或电路为开路，那么代理将调用 `fallback()` 方法。

```java
@Autowired
private MicroService myService;

@GetMapping("/endpoint")
public String value() throws Exception {
    return "Service returned: " + myService.service();
}

@Service
class MicroService {
    @HystrixCommand(fallbackMethod = "fallback")
    public String service() {
        // 执行某些处理。如果抛出异常，
        // 将调用 fallback 方法
        return "Success";
    }

    public String fallback(Throwable e) {
        return "Fallback! Reason: " + e.getMessage() + "\n";
    }
}
```
{: codeblock}

通过查看基于 Spring 的 [Hystrix 断路器](https://spring.io/guides/gs/circuit-breaker/){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标") 示例来了解更多信息。
{: tip}

### 使用超时
{: #spring-timeout}

调用远程服务时，可能不会立即返回，可能需要一段时间才能返回，或者可能永远不会返回。通过 Hystrix，您能够定义应用程序可接受的情况。可以在 `HystrixCommand` 注释中添加简单内容，以将超时值更改为缺省 1 秒之外的值：

```java
@HystrixCommand(
    fallbackMethod = "fallback",
    commandProperties = {
        @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "30000"),
    }
)
```
{: codeblock}

### 使用隔板
{: #spring-bulkhead}

Hystrix 支持基于信标和基于队列的隔板。以下片段说明了如何配置基于队列的隔板，此隔板分配 4 个线程，并将待处理请求数限制为 10：

```java
@HystrixCommand(
    fallbackMethod = "fallback",
    threadPoolProperties = {
        @HystrixProperty(name = "coreSize", value = "4"),
        @HystrixProperty(name = "maxQueueSize", value = "10")
    }
)
```
{: codeblock}

### 断路器状态
{: #spring-breaker-status}

Hystrix Spring 入门模板还有一个额外的秘密武器，可增强应用程序的缺省 `/health` 端点（通过 Spring Actuator 提供；有关更多信息，请查看[度量值主题](/docs/java?topic=java-spring-metrics#spring-metrics)）。

如果运行状况端点[配置为包含额外详细信息](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-endpoints.html#production-ready-health){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")，那么运行状况检查信息将包含断路器状态。（缺省情况下，此行为处于禁用状态）。

```
{
    "hystrix": {
        "openCircuitBreakers": [
            "MicroService::service"
        ],
        "status": "CIRCUIT_OPEN"
    },
    "status": "UP"
}
```
{: screen}

## 后续步骤
{: #spring-tolerance-next-steps notoc}

有关 Hystrix 配置的更多信息，请参阅 [Hystrix 配置 Wiki](https://github.com/Netflix/Hystrix/wiki/Configuration){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")。

有关将 Hystrix 用于 Spring 的更多信息，请参阅：

* [A Guide to Spring Cloud Netflix](https://www.baeldung.com/spring-cloud-netflix-hystrix){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")
* [Spring Circuit Breaker Guide](https://spring.io/guides/gs/circuit-breaker/){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")
