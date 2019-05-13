---

copyright:
  years: 2018, 2019
lastupdated: "2019-04-22"

keywords: fault tolerance spring, hystrix spring, netflix spring, hystrixcommand spring, bulkhead spring, circuit breaker spring

subcollection: java

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}

# 使用 Spring 实现容错
{: #spring-tolerance}

创建弹性系统对置于其中的所有服务提出了要求。云环境的动态性质要求将服务设计为能够对意外事件做出正常响应。

Spring 使用 [Netflix Hystrix](https://github.com/Netflix/Hystrix/wiki){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标") 容错库来支持应用程序级别的容错问题。使用 Hystrix，您可以创建回退、断路器、隔板和关联度量值。

此信息基于[云本机开发：容错](/docs/java?topic=cloud-native-fault-tolerance#fault-tolerance)中描述的容错实践而构建。
{: note}

## 将 Hystrix 用于 Spring 应用程序
{: #spring-hystrix-starter}

为了使 Hystrix 更容易在 Spring 应用程序中使用，为您提供了一个 Spring Boot 入门模板，以支持使用带注释的方法在应用程序中集成 Hystrix。

要添加 Hystrix，请添加以下依赖项：

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```
{: codeblock}

要使 Spring 能够使用此新功能，请向主应用程序类添加注释，并通过添加 `@EnableCircuitBreaker` 来支持 Spring 处理 Hystrix 注释以执行断路操作。

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

要向方法添加断路行为，请使用 `@HystrixCommand` 对其进行注释。Spring 将在使用 `@EnableCircuitBreaker` 注释的应用程序中查找这些方法，并使用 Hystrix 支持对其打包以监视错误和超时。然后，Spring 会根据需要调用相应的替代行为。

仅 `@Component` 或 `@Service` 中的方法支持 `@HystrixCommand` 注释。
{: note}

以下 `@HystrixCommand` 注释会对 `service()` 调用打包，以提供断路器行为。如果 `service()` 方法失败或电路为开路，那么代理将调用 `fallback()` 方法。

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

您的应用程序如何响应未响应的远程服务？可能需要等一段时间，或者永远无响应。通过 Hystrix，您能够定义应用程序可以接受的等待时间。

可以在 `HystrixCommand` 注释中添加简单内容，以将超时值从缺省值 1 秒更改为 30 秒：

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

Hystrix Spring 入门模板还有一个额外的秘密武器，可增强应用程序的缺省 `/health` 端点（通过 Spring Actuator 提供，有关更多信息，请参阅 [Spring 的度量值](/docs/java?topic=java-spring-metrics#spring-metrics)）。

如果运行状况端点[配置为包含额外详细信息 ](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-endpoints.html#production-ready-health){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")，那么运行状况检查信息将包含断路器状态。（缺省情况下，此行为处于禁用状态）。

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
