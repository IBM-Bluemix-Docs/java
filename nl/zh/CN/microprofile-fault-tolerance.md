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

# 使用 MicroProfile 实现容错
{: #mp-fault-tolerance}

针对这些容错主题，建议的方法是使用 Istio 功能。请查看标题为“Istio - 容错”的书，并浏览到“创建微服务 - 多语言功能”章节。该部分解释了很多主题，包括重试、超时、断路器、隔板和速率限制。

MicroProfile 还提供了一种方法，在“其他重要 Java 功能”一章的“MicroProfile 容错”标题下对此方法进行了概述。该部分说明了如何将回退功能与 Istio 一起使用，以及有关使用 `mpFaultTolerance` 而非 Istio 的详细信息。

Istio 无法提供回退功能，因为回退需要业务知识。更高级的方法是将 Istio 容错与 MicroProfile 回退功能一起使用，从而实现最大的弹性。例如，可以在调用后端服务时指定回退备份。如果 Istio 不能设法获得成功的返回，那么将调用回退方法。

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

如果使用了 Istio 的容错功能，那么可以通过将配置属性 MP_Fault_Tolerance_NonFallback_Enabled 设置为 false，以关闭 MicroProfile 容错。

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

有关比较 Istio 容错和 MicroProfile 容错的更多信息，请参阅 Emily Jiang 撰写的[这篇博客](https://www.eclipse.org/community/eclipse_newsletter/2018/september/MicroProfile_istio.php){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")。
