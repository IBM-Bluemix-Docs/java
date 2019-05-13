---

copyright:
  years: 2018, 2019
lastupdated: "2019-04-22"

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

# 使用 Spring 进行运行状况检查
{: #spring-healthcheck}

Spring Boot Actuator 提供的运行状况端点与应用程序生命周期绑定：在应用程序启动之前，无法访问该端点，并且在应用程序停止（在进程关闭之前就会停止）后，该端点即不可用。Spring Boot Actuator 将自动为在类路径上检测到的技术添加其他运行状况指示器。例如，如果应用程序使用了 JDBC，那么运行状况端点将包含一些基本测试，以确保可以访问支持数据存储。由于此原因，Actuator 定义的运行状况端点更适合用作就绪性探测器。

通过将 `spring-boot-actuator` 依赖项添加到 `pom.xml` 文件来启用 Spring Boot Actuator：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-actuator</artifactId>
</dependency>
```
{: codeblock}

Spring Boot Actuator 的不同版本之间存在一些行为差异。接下来的两个部分将用于探讨如何在两个版本的 Spring Boot 中创建活性和就绪性检查，首先描述最新的 Spring Boot 2。

## Spring Boot 2 中的运行状况检查
{: #spring-health-boot2}

Spring Boot 2 Actuator 定义了 `/actuator/health` 端点，当一切正常时，此端点会返回 `{"status": "UP"}` 有效内容。缺省情况下，此端点已启用，并且不需要应用程序代码。

使用 Spring Boot 2 Actuator 的未经认证的成功运行状况检查示例：
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

如上所示，缺省情况下，运行状况端点会返回简单的 UP 或 DOWN 状态。对于自动运行状况检查，此简单有效内容通常已足够。通过在 `application.properties` 中设置以下属性，可以启用详细的运行状况信息：

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

请注意，上例包含了一些 H2 数据库信息。这是 Spring Actuator 自动添加对支持服务的检查的示例。在本例中，应用程序使用的是 JDBC，并且在类路径上发现了 H2 驱动程序。

可以使用属性或代码来覆盖运行状况端点的缺省行为，如 [Spring Boot 2.1.x 参考指南](https://docs.spring.io/spring-boot/docs/2.1.x/reference/html/production-ready-endpoints.html#production-ready-health){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标") 中所述。

### Spring Boot 2 中的活性探测器
{: #spring-liveness-boot2}

Spring Boot 2 的 Actuator 框架是其自己的微型 REST 框架，可以使用定制端点进行扩展。这意味着可以添加一个简单的活性端点，其管理方式与内置运行状况指示器相同。定制活性端点可以如下所示进行详细声明：

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

此定制端点必须公开为 Web 端点：

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

### 在 Kubernetes 中配置 Spring Boot 2 就绪性和活性探测器
{: #spring-probe-config-2}

将就绪性和活性探测器配置添加到 Kubernetes 部署清单中的容器定义（请记下路径和端口值）：

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

要避免重新启动循环，请将 `livenessProbe.initialDelaySeconds` 设置为比服务初始化更长且安全的时间。然后您可以对 `readinessProbe.initialDelaySeconds` 使用更短的值，以在服务就绪后尽快将请求路由到服务。

## Spring Boot 1 中的运行状况检查
{: #spring-health-boot1}

Spring Boot 1 Actuator 定义了 `/health` 端点，当一切正常时，此端点会返回 `{"status": "UP"}` 有效内容。此端点不要求授权，但是如果配置了授权并且已授权调用者，响应中将包括其他信息。

使用 Spring Boot 1 Actuator 的未经认证的成功运行状况检查示例：

```
$ curl -i localhost:8080/health
HTTP/1.1 200
X-Application-Context: application
Content-Type: application/vnd.spring-boot.actuator.v1+json;charset=UTF-8
Content-Length: 15

{"status":"UP"}
```
{: screen}

经认证的成功运行状况检查示例：

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

可以使用属性或代码来覆盖运行状况端点的缺省行为，如 [Spring Boot V1.5.x 参考指南](https://docs.spring.io/spring-boot/docs/1.5.9.RELEASE/reference/htmlsingle/#_auto_configured_healthindicators){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标") 中所述。

### Spring Boot 1 中的活性探测器
{: #spring-liveness-boot1}

对于 Spring Boot 1，用于始终返回 {"status": "UP"} 和状态码 200 的简单活性 HTTP 端点将类似于以下内容：

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

### 在 Kubernetes 中配置 Spring Boot 1 就绪性和活性探测器
{: #spring-probe-config-1}

将就绪性和活性探测器配置添加到 Kubernetes 部署清单中的容器定义（请记下路径和端口值）：

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

要避免重新启动循环，请将 `livenessProbe.initialDelaySeconds` 设置为比服务初始化更长且安全的时间。然后您可以对 `readinessProbe.initialDelaySeconds` 使用更短的值，以在服务就绪后尽快将请求路由到服务。
