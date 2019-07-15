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

# Tolleranza di errore con MicroProfile
{: #mp-fault-tolerance}

L'approccio consigliato per questi argomenti di tolleranza di errore è di utilizzare le funzioni di Istio. Consulta la guida intitolata "Istio - Fault Tolerance" e seleziona il capitolo "Creating Microservices - Polyglot Capabilities". Vengono spiegati molti argomenti inclusi nuovi tentativi, timeout, interruttori, bulkhead e limitazioni della frequenza.

MicroProfile offre anche un approccio, descritto nel capitolo "Additional Java Features of Note" sotto l'intestazione "MicroProfile Fault Tolerance". Tale sezione illustra come la funzionalità di fallback può essere utilizzata insieme a Istio, oltre ai dettagli sull'utilizzo di `mpFaultTolerance` invece di Istio.

Istio non è in grado di offrire funzionalità di fallback in quanto il fallback richiede una conoscenza dell'attività di business. Un approccio più avanzato consiste nell'utilizzare la tolleranza dell'errore Istio insieme alle funzionalità di fallback di MicroProfile per raggiungere il massimo della resilienza. Ad esempio, puoi specificare un backup di fallback quando richiami un servizio di backend. Se Istio non è in grado di ottenere una risposta corretta, viene richiamato il metodo di fallback. 

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

Se utilizzi la funzionalità di tolleranza dell'errore di Istio, puoi disattivare le impostazioni di MicroProfile impostando la proprietà di configurazione MP_Fault_Tolerance_NonFallback_Enabled su false.

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

Per ulteriori informazioni sul confronto tra la tolleranza dell'errore di Istio e quella di MicroProfile, vedi [questo blog](https://www.eclipse.org/community/eclipse_newsletter/2018/september/MicroProfile_istio.php){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno") scritto da Emily Jiang.
