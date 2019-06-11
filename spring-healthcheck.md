---

copyright:
  years: 2018, 2019
lastupdated: "2019-06-10"

keywords: health check spring, spring health endpoint, spring-boot-actuator, liveness probe spring, readiness probe spring, spring kubernetes probe

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

# Health checks with Spring
{: #spring-healthcheck}

The health endpoint that is provided by Spring Boot actuator is tied to the application lifecycle and is not reachable until the application is started. Likewise, it is disabled as soon as the application is stopped (which happens before the process shuts down). Spring Boot Actuator automatically adds health indicators for technologies that are detected on the class path. For example, if your application uses JDBC, the health endpoint includes some basic tests to ensure that the backing data store can be reached. The Actuator-defined health endpoint is better suited for use as a readiness probe for this reason.

Enable Spring Boot Actuator by adding the `spring-boot-actuator` dependency to your `pom.xml` file:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-actuator</artifactId>
</dependency>
```
{: codeblock}

There are some behavior differences with Spring Boot actuator between versions. The next two sections explore how to create liveness and readiness checks in both versions of Spring Boot, starting with 2 as the most current.

## Health checks Spring Boot 2
{: #spring-health-boot2}

The Spring Boot 2 actuator defines an `/actuator/health endpoint`, which returns a `{"status": "UP"}` payload when all is well. This endpoint is enabled by default, and requires no application code.

See the following example of an unauthenticated successful health check that uses the Spring Boot 2 Actuator:
<!-- Spring Boot 2 test project: https://github.com/IBM/spring-alarm-application -->

```
$ curl -i localhost:8080/actuator/health
HTTP/1.1 200
Content-Type: application/vnd.spring-boot.actuator.v2+json;charset=UTF-8
Transfer-Encoding: chunked
Date: Fri, 07 Dec 2018 23:09:09 GMT

{"status":"UP"}
```
{: screen}

As shown, the health endpoint returns a simple UP or DOWN status by default. For automated health checking, this simple payload is usually sufficient. Detailed health information can be enabled by setting the following property in `application.properties`:

```properties
management.endpoint.health.show-details=always
```
{: codeblock}

```
$ curl -i localhost:8080/actuator/health
HTTP/1.1 200
Content-Type: application/vnd.spring-boot.actuator.v2+json;charset=UTF-8
Transfer-Encoding: chunked
Date: Fri, 07 Dec 2018 23:09:09 GMT

{"status":"UP","details":{"diskSpace":{"status":"UP","details":{"total":500068036608,"free":407611420672,"threshold":10485760}},"db":{"status":"UP","details":{"database":"H2","hello":1}}}}
```
{: screen}

Note the inclusion of some H2 database information. This example of Spring Actuator automatically adds checks for backing services. In this case, the application is using JDBC, and the H2 driver was discovered on the class path.

You can override the default behavior of the health endpoint with properties or code, as described in the [Spring Boot Reference Guide for 2.1.x](https://docs.spring.io/spring-boot/docs/2.1.x/reference/html/production-ready-endpoints.html#production-ready-health){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon").

### Liveness probe in Spring Boot 2
{: #spring-liveness-boot2}

The Actuator framework in Spring Boot 2 is its own mini-REST-framework that can be extended with custom endpoints. Which means that you can add a simple liveness endpoint that is managed in the same way the built-in health indicator is. A custom liveness endpoint can be trivially declared like in the following example:

```java
@Endpoint(id="liveness")
@Component
public class Liveness {
    @ReadOperation
    public String testLiveness() {
            return "{\"status\":\"UP\"}";
    }
}
```
{: codeblock}

This custom endpoint must be exposed as a web endpoint:

```properties
management.endpoints.web.exposure.include=health,liveness,...
```
{: codeblock}

```
$ curl -i localhost:8080/actuator/liveness
HTTP/1.1 200
Content-Type: application/vnd.spring-boot.actuator.v2+json;charset=UTF-8
Content-Length: 15
Date: Thu, 07 Feb 2019 22:38:27 GMT

{"status":"UP"}
```
{: screen}

### Configuring Spring Boot 2 readiness and liveness probes in Kubernetes
{: #spring-probe-config-2}

Add readiness and liveness probe configuration to the container definition in a Kubernetes deployment manifest (note the path and port values):

```yaml
spec:
  containers:
  - name: ...
    image: ...
    readinessProbe:
      httpGet:
        path: /actuator/health
        port: 8080
      initialDelaySeconds: 60
      timeoutSeconds: 5
    livenessProbe:
      httpGet:
        path: /actuator/liveness
        port: 8080
      initialDelaySeconds: 130
      timeoutSeconds: 10
      failureThreshold: 10
```
{: codeblock}

To avoid restart cycles, set `livenessProbe.initialDelaySeconds` to be safely longer than it takes your service to initialize. You can then use a shorter value for `readinessProbe.initialDelaySeconds` to route requests to the service as soon as it's ready.

## Health checks in Spring Boot 1
{: #spring-health-boot1}

The Spring Boot 1 actuator defines a `/health` endpoint, which returns a `{"status": "UP"}` payload when all is well. The endpoint does not require authorization, but if authorization is configured and the caller is authorized, additional information is included in the response.

See the following example of an unauthenticated successful health check that uses the Spring Boot 1 Actuator:
```
$ curl -i localhost:8080/health
HTTP/1.1 200
X-Application-Context: application
Content-Type: application/vnd.spring-boot.actuator.v1+json;charset=UTF-8
Content-Length: 15

{"status":"UP"}
```
{: screen}

An example of an authenticated successful health check:

```
$ curl -i localhost:8080/health
HTTP/1.1 200 OK
X-Application-Context: application
Content-Type: application/vnd.spring-boot.actuator.v1+json;charset=UTF-8
Content-Length: 221

{
    "status" : "UP",
    "diskSpace" : {
        "status" : "UP",
        "total" : 63251804160,
        "free" : 31316164608,
        "threshold" : 10485760
    },
    "db" : {
        "status" : "UP",
        "database" : "H2",
        "hello" : 1
    }
}
```
{: screen}

You can override the default behavior of the health endpoint with properties or code, as described in the [Spring Boot Reference v1.5.x](https://docs.spring.io/spring-boot/docs/1.5.9.RELEASE/reference/htmlsingle/#_auto_configured_healthindicators){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon").

### Liveness probe in Spring Boot 1
{: #spring-liveness-boot1}

For Spring Boot 1, a simple HTTP endpoint for liveness that always returns {"status": "UP"} with status code 200 would look something like the following snippet:

```java
@RestController
public class Liveness {
    private static Map<String, String> alwaysOk = Collections.singletonMap("status", "OK");

    @GetMapping("/liveness")
    public Map<String,String> testLiveness() {
        return alwaysOk;
    }
}
```
{: codeblock}

```
$ curl -i localhost:8080/liveness
HTTP/1.1 200
Content-Type: application/json;charset=UTF-8
Transfer-Encoding: chunked
Date: Thu, 07 Feb 2019 22:43:32 GMT

{"status":"OK"}
```
{: screen}

### Configuring Spring Boot 1 readiness and liveness probes in Kubernetes
{: #spring-probe-config-1}

Add readiness and liveness probe configuration to the container definition in a Kubernetes deployment manifest (note the path and port values):

```yaml
spec:
  containers:
  - name: ...
    image: ...
    readinessProbe:
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 60
      timeoutSeconds: 5
    livenessProbe:
      httpGet:
        path: /liveness
        port: 8080
      initialDelaySeconds: 130
      timeoutSeconds: 10
      failureThreshold: 10
```
{: codeblock}

To avoid restart cycles, set `livenessProbe.initialDelaySeconds` to be safely longer than it takes your service to initialize. You can then use a shorter value for `readinessProbe.initialDelaySeconds` to route requests to the service as soon as it's ready.
