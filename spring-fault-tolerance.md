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

# Fault tolerance with Spring
{: #spring-tolerance}

Creating a resilient system places requirements on all of the services within it. The dynamic nature of cloud environments demands that services are designed to be prepared, and respond gracefully to, the unexpected.

Spring uses the [Netflix Hystrix](https://github.com/Netflix/Hystrix/wiki){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon") fault tolerance library to support application-level fault tolerance concerns. Hystrix provides support for fallbacks, circuit breakers, bulkheads, and associated metrics. 

This information builds on fault tolerance practices described in [Cloud-native development: Fault tolerance](/docs/java?topic=cloud-native-fault-tolerance#fault-tolerance).
{: note}

## Using Hystrix with a Spring Application
{: #spring-hystrix-starter}

To make Hystrix easier to use within a Spring application, there is a Spring Boot Starter that enables an annotated approach to integrating Hystrix within your application.

Adding this single dependency to your application, will bring in Hystrix. 

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```
{: codeblock}

Similar to many other Spring Starters, to indicate to Spring that you want to use the functionality in the application, you add an annotation your main application class. To enable Spring's processing of the Hystrix annotations for circuit breaking, add `@EnableCircuitBreaker`.

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

To add circuit breaking behavior to a method, you annotate it with `@HystrixCommand`. Spring will find these methods within an application annotated with `@EnableCircuitBreaker` and wrap them with Hystrix functionality to monitor for errors/timeouts, and invoke the appropriate alternate behavior when required. 

`@HystrixCommand` is only supported on methods in an `@Component` or `@Service` {: note}

The following `@HystrixCommand` annotation wraps the `service()` invocation to provide the circuit breaking behavior. The proxy will call `fallback()` method if the `service()` method fails or if the circuit is open.

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

Learn more by checking out the Spring-based [Hystrix Circuit Breaker](https://spring.io/guides/gs/circuit-breaker/){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon") example.
{: tip}

### Using Timeouts
{: #spring-timeout}

When invoking a remote service, it may not return immediately, it may take a while, or it may take forever. Hystrix gives you the
ability to define what is acceptable for your application. A simple addition to the `HystrixCommand` annotation can be used to change the timeout value from the default value of 1 second:

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

Hystrix supports both semaphore and queue-based bulkheads. The following snippet shows how to configure a queue-based bulkhead that allocates 4 threads and limits the number of outstanding requests to 10:

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

The Hystrix Spring starter has one extra trick up it's sleeve, it will enhance the default `/health` endpoint for the application (supplied via a Spring Actuator, for more check the the [Metrics Topic](/docs/java?topic=java-spring-metrics#spring-metrics)).

If the health endpoint is [configured to include extra detail](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-endpoints.html#production-ready-health){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon"), then Circuit breaker status will be included with the health check information. (This behavior is disabled by default).

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

For more on Hystrix configuration, see the [Hystrix Configuration Wiki](https://github.com/Netflix/Hystrix/wiki/Configuration){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon").

For more on Hystrix with Spring, see:

* [A Guide to Spring Cloud Netflix](https://www.baeldung.com/spring-cloud-netflix-hystrix){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon")
* [Spring Circuit Breaker Guide](https://spring.io/guides/gs/circuit-breaker/){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon")
