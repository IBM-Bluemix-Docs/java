---

copyright:
  years: 2019
lastupdated: "2019-03-15"

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

對於這些容錯主題，我們建議的方法是主要利用 Istio 的特性。請參閱本書中『建立微服務 -- Polyglot 功能』章節中的「Istio -- 容錯」。這涵蓋了重試、逾時、斷路器、隔板及速率限制等事項。

MicroProfile 也提供一個方法，『附註的其他 Java 特性』章節中的「MicroProfile 容錯」標題下有概述該方法。該小節顯示撤回功能如何與 Istio 一起使用，並針對利用 mpFaultTolerance（而非 Istio）的方法，顯示更多詳細資料。

Istio 無法提供撤回功能，因為撤回需要商業知識。更進階的方法是搭配使用 Istio 容錯與 MicroProfile 撤回功能，以達到最大的備援力。例如，您可以在呼叫後端服務時指定撤回備份。如果 Istio 無法用來取得成功的退貨，則將呼叫撤回方法。

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

如果您使用 Istio 的容錯功能，您可以將 MP_Fault_Tolerance_NonFallback_Enabled 內容設為 false，來關閉「MicroProfile 容錯」。

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

如需「Istio 容錯」與「MicroProfile 容錯」的比較相關資訊，請參閱 Emily Jiang 所撰寫的[此部落格](https://www.eclipse.org/community/eclipse_newsletter/2018/september/MicroProfile_istio.php){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")。
