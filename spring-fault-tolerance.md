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
{:external: target="_blank" .external}

# Fault tolerance with Spring
{: #spring-tolerance}

Creating a resilient system places requirements on all of the services within it. The dynamic nature of cloud environments demands that services be designed to respond gracefully to the unexpected.

Spring uses the [Netflix Hystrix](https://github.com/Netflix/Hystrix/wiki){: external} fault tolerance library to support application-level fault tolerance concerns. With Hystrix, you can create fallbacks, circuit breakers, bulkheads, and associated metrics.

This information builds on fault tolerance practices that are described in [Cloud-native development: Fault tolerance](/docs/java?topic=cloud-native-fault-tolerance#fault-tolerance).
{: note}

## Using Hystrix with a Spring Application
{: #spring-hystrix-starter}

To make Hystrix easier to use within a Spring application, there is a Spring Boot Starter that enables an annotated approach to integrate Hystrix within your application.

To add Hystrix, add the following dependency:

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```
{: codeblock}

For Spring to use this new function, add an annotation to your main application class, and enable Spring's processing of the Hystrix annotations for circuit breaking by adding `@EnableCircuitBreaker`.

```java
@SpringBootApplication
@EnableCircuitBreaker
public class MyApplication {

	public static void main(String[] args) {
		SpringApplication.run(MyApplication.class, args);
	}
```
{: codeblock}

### Using HystrixCommand to define a fallback
{: #spring-fallback}

To add circuit breaker behavior to a method, annotate it with `@HystrixCommand`. Spring discovers methods within an application that is annotated with `@EnableCircuitBreaker`, and wraps them with Hystrix support to monitor for errors and timeouts. Then, Spring invokes the appropriate alternative behavior when required.

The `@HystrixCommand` annotation is only supported on methods in an `@Component` or `@Service`.
{: note}

The following `@HystrixCommand` annotation wraps the `service()` invocation to provide the circuit breaker behavior. The proxy calls the `fallback()` method if the `service()` method fails or if the circuit is open.

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
        // Do some processing. If an exception is thrown
        // the fallback method will be called
        return "Success";
    }

    public String fallback(Throwable e) {
        return "Fallback! Reason: " + e.getMessage() + "\n";
    }
}
```
{: codeblock}

Learn more by checking out the Spring-based [Hystrix Circuit Breaker](https://spring.io/guides/gs/circuit-breaker/){: external} example.
{: tip}

### Using timeouts
{: #spring-timeout}

How does your application respond to a remote service that is not responding? The wait can take a while, or it can take forever. Hystrix gives you the ability to define what amount of time to wait is acceptable for your application.

A simple addition to the `HystrixCommand` annotation can be used to change the timeout value from the default value of 1 second to 30 seconds:

```java
@HystrixCommand(
    fallbackMethod = "fallback",
    commandProperties = {
        @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "30000"),
    }
)
```
{: codeblock}

### Using Bulkheads
{: #spring-bulkhead}

Hystrix supports both semaphore and queue-based bulkheads. The following snippet shows how to configure a queue-based bulkhead that allocates four threads and limits the number of outstanding requests to 10:

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

### Circuit Breaker Status
{: #spring-breaker-status}

The Hystrix Spring starter has one extra trick up its sleeve, it enhances the default `/health` endpoint for the application, which is supplied through a Spring Actuator. For more information, see [Metrics with Spring](/docs/java?topic=java-spring-metrics#spring-metrics)).

If the health endpoint is [configured to include extra detail](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-endpoints.html#production-ready-health){: external}, then the Circuit breaker status is included with the health check information. (This behavior is disabled by default).

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

## Next Steps
{: #spring-tolerance-next-steps notoc}

For more on Hystrix configuration, see the [Hystrix Configuration Wiki](https://github.com/Netflix/Hystrix/wiki/Configuration){: external}.

For more on Hystrix with Spring, see:

* [A Guide to Spring Cloud Netflix](https://www.baeldung.com/spring-cloud-netflix-hystrix){: external}
* [Spring Circuit Breaker Guide](https://spring.io/guides/gs/circuit-breaker/){: external}
