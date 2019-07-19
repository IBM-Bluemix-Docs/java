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

# Fehlertoleranz mit MicroProfile
{: #mp-fault-tolerance}

Der empfohlene Ansatz für diese Fehlertoleranzthemen ist die Nutzung der Funktionen von Istio. Weitere Informationen finden Sie in der Veröffentlichung zur Fehlertoleranz bei Istio und insbesondere in dem Kapitel, das die Erstellung von Mikroservices und polyglotte Funktionen beschreibt. Hier werden Themen wie Wiederholungen, Zeitlimits, Circuit Breakers, Bulkheads und Durchsatzbegrenzungen behandelt. 

MicroProfile bietet ebenfalls einen Ansatz, der im Kapitel zu weiteren wichtigen Java-Funktionen im Abschnitt zur MicroProfile-Fehlertoleranz beschrieben ist. In diesem Abschnitt wird beschrieben, wie die Fallback-Funktion in Kombination mit Istio verwendet werden kann. Zudem sind Informationen zur Verwendung von `mpFaultTolerance` anstelle von Istio enthalten. 

Istio bietet keine Fallback-Funktionen, da Fallback Geschäftswissen erfordert. Ein erweiterter Ansatz ist die Verwendung der Istio-Fehlertoleranz zusammen mit Fallback-Funktionen von MicroProfile, um eine maximale Ausfallsicherheit zu erzielen. Sie können beispielsweise beim Aufrufen eines Back-End-Service eine Fallback-Sicherung angeben. Wenn Istio keine erfolgreiche Rückgabe empfängt, wird die Fallback-Methode aufgerufen. 

```java
@GET
@Fallback(fallbackMethod="myFallback")
public String callService() {
    //Servicer b aufrufen
    return  bclient.hello();
}

public String myFallback() {
    return "Greetings\... This is a fallback!";
}
```
{: codeblock}

Wenn Sie die Fehlertoleranzfunktionalität von Istio verwenden, können Sie die MicroProfile-Fehlertoleranz inaktivieren, indem Sie die Konfigurationseigenschaft 'MP_Fault_Tolerance_NonFallback_Enabled' auf 'false' setzen. 

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

Weitere Informationen zum Vergleich der Fehlertoleranz bei Istio und MicroProfile finden Sie in [diesem Blog](https://www.eclipse.org/community/eclipse_newsletter/2018/september/MicroProfile_istio.php){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") von Emily Jiang. 
