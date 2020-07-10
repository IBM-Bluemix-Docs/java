---

copyright:
  years: 2019
lastupdated: "2019-08-26"

keywords: health check jax-rs, jax-rs endpoint, jax-rs status, readiness jax-rs, liveness jax-rs, microprofile health

subcollection: java

---

{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}
{:external: target="_blank" .external}

# Health checks with JAX-RS
{: #jaxrs-healthcheck}

As discussed in the prior section, Kubernetes provides multiple methods for integrating health probes that include execution of a command, and network checking through TCP or HTTP endpoints. Although you can implement any of these probes in the Java&trade; language, a JAX-RS implementation of HTTP probes and MicroProfile Health is discussed in detail here.

## Defining a readiness JAX-RS endpoint
{: #mp-readiness}

A minimal readiness endpoint looks something like the following example:

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

An endpoint for an application that is deployed in WebSphere Liberty achieves a certain level of readiness behavior without extra effort. Liberty fails requests to the health endpoint with an appropriate 503 response until the application is running. This basic implementation can be extended to include required capabilities. For example, consider evaluating `@ApplicationScoped` CDI resources to ensure that dependency injection processing is complete. After the probe is implemented, it can be enabled by configuring the HTTP probe type as noted in the prior section.

Always remember the role and purpose of fault tolerance: microservices are expected to handle failure and disruption in communications with downstream services. You must include checks for capabilities that have no feasible fallback inside a readiness check.

## Defining a liveness JAX-RS endpoint
{: #jaxrs-liveness}

A liveness probe is deliberate about what it checks, as a failure results in immediate termination of the process. A simple liveness endpoint can look something like the following example:

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

The liveness probe implementation can be extended to check process local conditions, which indicate an unrecoverable process state. However, the challenge with these checks often lies in the fact that if such processing is possible, the server is likely working normally to process other requests. For this reason, a "keep it simple" mantra can be readily applied to the liveness check implementation. After the liveness sources are coded to enable the probe endpoint, the HTTP liveness endpoint can be enabled in the container orchestration sources as explained in the last chapter.

To avoid restart cycles, the `initialDelaySeconds` attribute for the liveness check must be longer than the longest expected server start time. For a Java&trade; application server that commonly takes 30 seconds to start, choose a larger value, such as 60 seconds.

## Using the MicroProfile Health API
{: #mp-health-api}

The [MicroProfile Health API](https://www.ibm.com/support/knowledgecenter/en/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/twlp_microprofile_healthcheck.html){: external} (mpHealth) defines a framework to simplify the implementation of health checks. Version 1.0 of the `mpHealth` API defines a single endpoint. The next version will provide two, one for liveness, and one for readiness.

To use the MicroProfile Health API with Liberty, add the `mpHealth` feature to `server.xml`:

```xml
<featureManager>
   <feature>mpHealth-1.0</feature>
</featureManager>
```
{: codeblock}

You can then check the status of the application with a browser by accessing the `/health` endpoint. The code returns a payload `{"outcome": "UP"}` when all is well. The following code shows you how to use MicroProfile Health APIs to indicate whether the pod is working normally. The `call` method allows developers to provide check algorithm to indicate the health status of a pod. Something like nearly running out of memory is a good indicator for the healthiness of a pod. Checking for downstream database connection might be a good check for the readiness of the pod. If there is no specific check, you can use the following to indicate whether a pod is alive or ready.

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

Then, you need to specify the `livenessProbe` and `readinessProbe` for Kubernetes, as shown in the following example.
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

As mentioned earlier, you can use different logics to indicate readiness and liveness. You can use MicroProfile Health to indicate readiness check and then create a JAX-RS endpoint for readiness check or vice versa. The next version of MicroProfile Health will provide two end points, one for liveness and the other for readiness.

OpenLiberty has a hands-on guide that provides an example of how to add custom checks to the MicroProfile Health endpoint: https://openliberty.io/guides/microprofile-health.html

You can override the default behavior of the MicroProfile health endpoint as described in [MicroProfile health checks](https://www.ibm.com/support/knowledgecenter/en/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/twlp_microprofile_healthcheck.html){: external}.
