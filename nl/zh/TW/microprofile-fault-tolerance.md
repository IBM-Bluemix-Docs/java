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

# MicroProfile 的容錯
{: #mp-fault-tolerance}

針對這些容錯主題，建議的方法是使用 Istio 特性。請查看標題為「Istio - 容錯」的書籍，並瀏覽到『建立微服務 - Polyglot 功能』章節。該章節解釋了很多主題，包括重試、逾時、斷路器、隔板和速率限制。

MicroProfile 還提供了一種方法，在『其他重要 Java 特性』一章的「MicroProfile 容錯」標題下概述該方法。該小節顯示撤回功能如何與 Istio 一起使用，以及針對利用 `mpFaultTolerance` 而非 Istio 的方法，顯示更多詳細資料。

Istio 無法提供撤回功能，因為撤回需要業務知識。更進階的方法是搭配使用 Istio 容錯與 MicroProfile 撤回功能，以達到最大的備援力。例如，可以在呼叫後端服務時指定撤回備份。如果 Istio 無法用來取得成功的退貨，則將呼叫撤回方法。

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

如果使用了 Istio 的容錯功能，則可以透過將配置內容 MP_Fault_Tolerance_NonFallback_Enabled 設定為 false，以關閉 MicroProfile 容錯。

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

如需比較 Istio 容錯和 MicroProfile 容錯的相關資訊，請參閱 Emily Jiang 撰寫的[這篇部落格](https://www.eclipse.org/community/eclipse_newsletter/2018/september/MicroProfile_istio.php){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")。
