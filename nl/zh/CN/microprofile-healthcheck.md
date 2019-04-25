---

copyright:
  years: 2019
lastupdated: "2019-03-27"

keywords: health check jax-rs, jax-rs endpoint, jax-rs status, readiness jax-rs, liveness jax-rs, microprofile health

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

# 使用 JAX-RS 进行运行状况检查
{: #jaxrs-healthcheck}

如前一节所述，Kubernetes 提供了多种用于集成运行状况探测器的方法，包括执行命令，以及通过 TCP 或 HTTP 端点进行网络检查。虽然可以用 Java 语言实现其中任何探测器，但在此将详细讨论 HTTP 探测器和 MicroProfile 运行状况的 JAX-RS 实现。

## 定义就绪性 JAX-RS 端点
{: #mp-readiness}

一个极简的就绪性端点类似于以下内容：

```java
@Path("ready")
public class ReadinessEndpoint {
  @GET
  @Produces(MediaType.APPLICATION_JSON)
  public Response testReadiness() {
    if ( ! ready ) {
      return Response.status(503).entity("{\"status\":\"DOWN\"}").build();
    }
    return Response.ok("{\"status\":\"UP\"}").build();
  }
}
```
{: codeblock}

在 WebSphere Liberty 中部署的应用程序中，像上例中这样的端点无需额外的工作，就可实现某种程度的就绪性行为。在应用程序启动之前，Liberty 向运行状况端点的请求会失败，并返回相应的 503 响应。应该对此基本实现进行扩展，以包含必需的功能。例如，考虑评估 `@ApplicationScoped` CDI 资源，以确保依赖项注入处理完成。实现了探测器后，可以通过配置 HTTP 探测器类型来启用该探测器，如前一节中所述。

请务必记住容错的作用和目的：微服务需要处理与下游服务的通信中的故障和中断。因此，在就绪性检查内仅包含针对没有可行回退的功能的检查内容。

## 定义活性 JAX-RS 端点
{: #jaxrs-liveness}

活性探测器应该仔细规划检查的内容，因为故障会导致进程立即终止。简单的活性端点可能类似于以下内容：

```java
@Path("liveness")
public class LivenessEndpoint {
    @GET
    @Produces(MediaType.APPLICATION_JSON)
    public Response testLiveness() {
      return Response.ok("{\"status\":\"UP\"}").build();
    }
}
```
{: codeblock}

可以扩展活性探测器实现，以查询指示不可恢复的进程状态的进程本地状况。然而，此类检查面临的挑战往往在于，如果能够执行这样的处理，说明服务器的运行状况可能足够正常，可以执行其他请求。出于这一原因，“极简”原则应随时应用于活性检查实现。对活性源进行编码以启用探测器端点后，就可以在容器编排源中启用 HTTP 活性端点，如上一章所述。

为了避免重新启动周期，活性检查的 initialDelaySeconds 属性应该长于预期的最长服务器启动时间。对于通常需要 30 秒才能启动的 Java 应用程序服务器，请选择更大的值，例如 60 秒。

## 使用 MicroProfile 运行状况 API
{: #mp-health-api}

[MicroProfile 运行状况 API](https://www.ibm.com/support/knowledgecenter/en/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/twlp_microprofile_healthcheck.html){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标") (mpHealth) 定义了一个框架，用于简化运行状况检查的实现。mpHealth API V1.0 仅定义了一个端点。下一个版本将提供两个端点，一个用于活性，一个用于就绪性。

要将 MicroProfile 运行状况 API 与 Liberty 配合使用，请将 `mpHealth` 功能添加到 `server.xml`：

```xml
<featureManager>
   <feature>mpHealth-1.0</feature>
</featureManager>
```
{: codeblock}

然后，可以使用浏览器通过访问 `/health` 端点来检查应用程序的状态。当一切正常时，代码会返回有效内容 `{"outcome": "UP"}`。下面说明了如何使用 MicroProfile 运行状况 API 来指示 pod 是处于运行正常还是就绪状态。`call` 方法允许开发者提供检查算法来指示 pod 的运行状态。像内存即将不足这样的情况可以明确地指示 pod 的运行状况。检查下游数据库连接可能是有效的 pod 就绪性检查方法。如果没有特定检查，您可以使用以下内容来指示 pod 是处于活动状态还是就绪状态。

```java
import org.eclipse.microprofile.health.Health;
import org.eclipse.microprofile.health.HealthCheck;
import org.eclipse.microprofile.health.HealthCheckResponse;

import javax.enterprise.context.ApplicationScoped;

@Health
@ApplicationScoped
public class HealthResource implements HealthCheck {

    @Override
    public HealthCheckResponse call() {
        return HealthCheckResponse.named("service-a").withData("service-a", "ok").up().build();
    }
}
```
{: codeblock}

然后，可以为 Kubernetes 指定 `livenessProbe` 和 `readinessProbe`，如下所示。
```yaml
livenessProbe:
  exec:
    command: ["sh", "-c", "curl -s http://localhost:9080/health| grep -q service-a"]
  initialDelaySeconds: 60
readinessProbe:
  exec:
    command: ["sh", "-c", "curl -s http://localhost:9080/health | grep -q service-a"]
  initialDelaySeconds: 40
```
{: codeblock}

如前所述，可以使用不同的逻辑来指示就绪性和活性。可以使用 MicroProfile 运行状况来指示就绪性检查，然后创建 JAX-RS 端点进行就绪性检查，反之亦然。MicroProfile 运行状况的下一个版本将提供两个端点，一个用于活性，另一个用于就绪性。

OpenLiberty 有一个实操指南，提供了如何向 MicroProfile 运行状况端点添加定制检查的示例：https://openliberty.io/guides/microprofile-health.html

可以覆盖 MicroProfile 运行状况端点的缺省行为，如[执行 MicroProfile 运行状况检查](https://www.ibm.com/support/knowledgecenter/en/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/twlp_microprofile_healthcheck.html){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标") 中所述。
