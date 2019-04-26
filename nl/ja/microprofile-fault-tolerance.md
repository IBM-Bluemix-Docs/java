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

# MicroProfile によるフォールト・トレランス
{: #mp-fault-tolerance}

このフォールト・トレランスのトピックで推奨されているアプローチでは、主に Istio の機能を活用します。このマニュアルの『マイクロサービスの作成 -- 多言語機能』の章の『Istio -- フォールト・トレランス』を参照してください。そこでは、再試行、タイムアウト、回路ブレーカー、バルクヘッド、レート制限などについて扱われています。

MicroProfile は、『MicroProfile フォールト・トレランス』の見出しの下にある『注目すべきその他の Java 機能』の章で概説されているアプローチも提供しています。そのセクションでは、フォールバック機能を Istio と組み合わせて使用する方法や、Istio の代わりに mpFaultTolerance を利用するアプローチに関心を持つ人に対する詳細がさらに示されています。

フォールバックにはビジネス上の知識が必要なため、Istio はフォールバック機能を提供できません。MicroProfile フォールバック機能と共に Istio フォールト・トレランスを使用するという、より高度なアプローチがあります。これによって、最大の回復力を達成できます。例えば、バックエンド・サービスを呼び出すときにフォールバック・バックアップを指定できます。Istio では正常な結果を返すことができない場合に、フォールバック・メソッドが呼び出されます。

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

Istio のフォールト・トレランス機能を使用している場合は、config プロパティー MP_Fault_Tolerance_NonFallback_Enabled を false に設定することで MicroProfile フォールト・トレランスをオフにすることができます。

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

Istio フォールト・トレランスと MicroProfile フォールト・トレランスの比較について詳しくは、Emily Jiang によって書かれた[このブログ](https://www.eclipse.org/community/eclipse_newsletter/2018/september/MicroProfile_istio.php){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン")を参照してください。
