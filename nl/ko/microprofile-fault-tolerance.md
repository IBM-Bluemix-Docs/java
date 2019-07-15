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

# MicroProfile의 결함 허용
{: #mp-fault-tolerance}

결함 허용 주제에서 권장되는 방법은 Istio의 기능을 사용하는 것입니다. "Istio - 결함 허용"을 확인한 후 "마이크로서비스 작성 - Polyglot 기능" 장을 찾아보십시오. 재시도, 제한시간, 회로 차단기, 벌크헤드 및 비율 한계를 포함하는 여러 주제가 설명됩니다.

또한 MicroProfile은 "MicroProfile 결함 허용" 표제 아래 "중요한 추가 Java 기능" 장에 설명된 방식을 제공합니다. 이 절에서는 폴백 기능을 Istio와 함께 사용하는 방법과 Istio 대신 `mpFaultTolerance` 사용에 대한 세부사항을 보여줍니다.

폴백에 비즈니스 지식이 필요하므로 Istio는 폴백 기능을 제공할 수 없습니다. 고급 방식은 MicroProfile 폴백 기능과 함께 Istio 결함 허용을 사용하여 최대 복원력을 달성하는 방식입니다. 예를 들어, 백엔드 서비스를 호출할 때 폴백 백업을 지정할 수 있습니다. Istio가 성공적인 리턴 결과를 갖도록 관리할 수 없는 경우 폴백 메소드가 호출됩니다.

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

Istio의 결함 허용 기능을 사용하는 경우 구성 특성 MP_Fault_Tolerance_NonFallback_Enabled를 false로 설정하여 MicroProfile 결함 허용을 설정 해제할 수 있습니다.

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

Istio 결함 허용과 MicroProfile 결함 허용 비교에 대한 자세한 정보는 Emily Jiang이 작성한 [이 블로그](https://www.eclipse.org/community/eclipse_newsletter/2018/september/MicroProfile_istio.php){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")를 참조하십시오.
