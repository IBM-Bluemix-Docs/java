---

copyright:
  years: 2019
lastupdated: "2019-06-10"

keywords: fault tolerance microprofile, retries microprofile, circuit breakers microprofile, bulkhead microprofile, microprofile limits

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

# Fault Tolerance with MicroProfile
{: #mp-fault-tolerance}

The recommended approach for these fault tolerance topics is to use the features of Istio. Check out the book titled "Istio - Fault Tolerance", and browse to the "Creating Microservices - Polyglot Capabilities" chapter. Many topics are explained that includes retries, timeouts, circuit breakers, bulkheads, and rate limits.

MicroProfile also offers an approach, which is outlined in the "Additional Java Features of Note" chapter under the heading "MicroProfile Fault Tolerance". That section shows how the fallback capability can be used along with Istio, as well as details about using `mpFaultTolerance` instead of Istio.

Istio is not capable of offering fallback capabilities because fallback requires business knowledge. A more advanced approach is to use Istio fault tolerance together with MicroProfile fallback capabilities to achieve the maximum of resilience. For instance, you can specify a fallback backup when you call a backend service. If Istio cannot manage to get a successful return, the fallback method is invoked.

```java
@GET
@Fallback(fallbackMethod="myFallback")
public String callService() {
    //call servicer b
    return  bclient.hello();
}

public String myFallback() {
    return "Greetings\... This is a fallback!";
}
```
{: codeblock}

If you use Istio's fault tolerance capability, you can turn them off MicroProfile Fault Tolerance by setting the config property MP_Fault_Tolerance_NonFallback_Enabled to false.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: service-config
data:
  serviceB_host: serviceb-service
  serviceB_http_port: "8080"
  lifetime: "0"
  failFrequency: "0"
  MP_Fault_Tolerance_NonFallback_Enabled: "false"
```
{: codeblock}

For more information on the comparison of Istio Fault Tolerance and MicroProfile Fault Tolerance, see [this blog](https://www.eclipse.org/community/eclipse_newsletter/2018/september/MicroProfile_istio.php){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon") that is written by Emily Jiang.
