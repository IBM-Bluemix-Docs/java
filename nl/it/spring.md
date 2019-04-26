---

copyright:
  years: 2019
lastupdated: "2019-03-15"

keywords: spring framework, spring reference, spring boot, boot actuator, spring kubernetes

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

# Spring
{: #spring-overview}

La piattaforma Spring è un ecosistema di progetti inteso a facilitare la creazione di applicazioni Java. Di norma, il termine "Spring" si riferisce allo Spring Framework, ma viene anche utilizzato genericamente per fare riferimento a qualsiasi progetto che fa parte delle tecnologie basate su Spring o che ne fa uso, compreso Spring Boot.

## Spring Framework
{: #spring-framework}

Lo Spring Framework è stato introdotto nel 2002 e, successivamente ha rilasciato una nuova versione principale circa ogni 3 anni. Il framework include un ampio insieme di componenti (indicati come moduli Spring) che coprono tutto, dagli endpoint REST alle astrazioni dei database. Con la sua release più recente nel 2017, Spring Framework 5 ha introdotto un supporto JDK aggiornato, insieme a WebFlux, un nuovo modulo per la programmazione reattiva basato su [Project Reactor](https://projectreactor.io/){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno").

Spring Framework 4.3 è l'ultimo ramo della funzione Spring Framework 4.x, con un supporto per tutto il 2020. Le applicazioni che utilizzano versioni meno recenti di Spring devono essere migrate a Spring Framework 5.

La documentazione completa per Spring Framework è qui:

* [Spring Framework Reference Guide for 5.1.x](https://docs.spring.io/spring/docs/5.1.x/spring-framework-reference/){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno")
* [Spring Framework Reference Guide for 4.3.x](https://docs.spring.io/spring/docs/4.3.x/spring-framework-reference/){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno")
* [Upgrading to Spring Framework 5.x](https://github.com/spring-projects/spring-framework/wiki/Upgrading-to-Spring-Framework-5.x){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno")

## Spring Boot
{: #spring-boot}

Spring Boot è stato introdotto nel 2014, per migliorare le architetture delle applicazioni web senza contenitore. Il suo obiettivo era quello di spostare la configurazione da XML al codice. Spring Boot fornisce dei meccanismi per la creazione di applicazioni basati su una visione dogmatica di quali tecnologie devono essere utilizzate. Le applicazioni Spring Boot sono applicazioni Spring e utilizzano un modello di programmazione unico per tale ecosistema.

Spring Boot enfatizza la convenzione rispetto alla configurazione e utilizza una combinazione di rilevamento di percorso classi e annotazioni per abilitare delle funzioni aggiuntive. Per impostazione predefinita, le applicazioni Spring Boot sono integrate in un file jar eseguibile autonomo che include tutte le dipendenze richieste (compreso un server Tomcat integrato). In alternativa, le applicazioni possono essere impacchettate come dei file war distribuiti a un server delle applicazioni.

Spring Boot è ora alla versione 2.0, rilasciata nel 2018. La manutenzione del ramo 1.5.x cesserà nel mese di agosto del 2019.

La documentazione completa per Spring Boot è qui:

* [Spring Boot Reference Guide for 2.1.x](https://docs.spring.io/spring-boot/docs/2.1.x/reference){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno")
* [Spring Boot Reference Guide for 1.5.x](https://docs.spring.io/spring-boot/docs/1.5.x/reference/){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno")

## Selezione di Spring Framework o Spring Boot
{: #spring-framework-or-boot}

Per le nuove applicazioni native cloud, se scegli di utilizzare Spring Framework, devi utilizzare anche Spring Boot. Spring Boot semplifica la configurazione e l'assemblaggio di applicazioni basate su Spring e fornisce delle funzioni, come Spring Actuator, che semplificano la creazione di [applicazioni native del cloud](/docs/cloud-native?topic=cloud-native-overview#overview).

### Spring Boot Actuator
{: #spring-boot-actuator}

Spring Boot Actuator definisce una raccolta di endpoint che sono utili per ispezionare un'applicazione Spring in esecuzione. La loro abilitazione è semplice quanto aggiungere una singola dipendenza all'applicazione.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```
{: codeblock}

La struttura e la modalità di funzionamento di Spring Boot Actuator sono cambiate in modo significativo tra Spring Boot 1 e Spring Boot 2. Il Boot 1 Actuator si affiancava all'applicazione, con meccanismi di configurazione e sicurezza separati. La versione Boot 2 è più capace e utilizza un modello di sicurezza che si integra con il resto dell'applicazione Spring.

Le modifiche della modalità di funzionamento sono particolarmente rilevanti se stai eseguendo la migrazione di un'applicazione Boot 1 a Boot 2. Per ulteriori dettagli, leggi la sezione relativa all'Actuator del manuale [Boot 1 to Boot 2 migration guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.0-Migration-Guide#spring-boot-actuator){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno").
{: tip}

In Boot 2, l'Actuator funge da mini-framework REST. Tutti gli endpoint sono sotto un contesto `/actuator`. Vengono forniti degli endpoint per eseguire i controlli di integrità o per raccogliere le informazioni relative alle metriche, alle applicazioni o di altro tipo relative all'ambiente. Per un elenco completo, consulta la [documentazione di Actuator](https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/html/production-ready-features.html#production-ready){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno").

È facile creare un tuo endpoint Actuator utilizzando un `@Component` Spring:

```java
@Endpoint(id="example")
@Component
public class Example {
    @ReadOperation
    public String example() {
        return "{\"example\":\"value\"}";
    }
}
```
{: codeblock}

Spring Boot ha due controlli per configurare quali Actuator sono attivi al runtime; gli Actuator possono essere "abilitati" o "esposti". Per poter essere raggiungibile, un Actuator deve essere entrambe le cose. Per impostazione predefinita, molti endpoint sono abilitati, ma sono esposti solo health e info. Inoltre, se Spring Security è abilitata per l'applicazione, l'accesso deve essere consentito esplicitamente agli endpoint Actuator.

L'Actuator in Spring Boot 1 ha una sua configurazione di sicurezza, di norma configurata in application.properties. Invece di "abilitate" ed "esposte", sono presenti "abilitate" e "sensibili". Gli endpoint sensibili richiedono l'autenticazione, se serviti su HTTP. Per impostazione predefinita, l'endpoint /metrics dello Spring Boot Actuator è abilitato in Spring Boot 1 ma è considerato sensibile e, pertanto, richiede l'autorizzazione o di essere impostato come non sensibile. Per alcuni ambienti, la configurazione della sicurezza di Spring potrebbe essere la risposta giusta ma, in Kubernetes, gli endpoint delle metriche rimangono interni al cluster. Per ulteriori informazioni, consulta la [documentazione dell'Actuator di Spring Boot 1](https://docs.spring.io/spring-boot/docs/1.5.2.RELEASE/reference/htmlsingle/#production-ready){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno").
{: note}

## Spring Cloud
{: #spring-cloud}

Spring Cloud è una raccolta di integrazioni tra tecnologie cloud di terze parti e il modello di programmazione Spring. La sua finalità è quella di fornire assistenza agli sviluppatori che creano applicazioni Spring per la distribuzione sul cloud. L'ampiezza della copertura raggiunta da Spring Cloud è resa possibile dalla sua natura modulare, in cui ciascun progetto in Spring Cloud è concentrato su una specifica tecnologia o su uno specifico insieme di tecnologie.

I progetti Spring Cloud seguono l'approccio generale di Spring di favorire la convenzione rispetto alla configurazione. La maggior parte delle funzionalità è abilitata aggiungendo la dipendenza corretta al momento della creazione.

Vedi il [sito del progetto Spring Cloud](https://spring.io/projects/spring-cloud){: new_window} ufficiale ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno") per ulteriori informazioni sui progetti Spring Cloud e sulle tecnologie supportate.

### Spring Cloud con Kubernetes
{: #spring-cloud-kubernetes}

Spring Cloud Kubernetes è un progetto Spring Cloud concepito per rendere diverse attività più facili per le applicazioni Spring in esecuzione in Kubernetes. Spring Cloud Kubernetes offre implementazioni di interfacce Spring comuni associate ai concetti e ai servizi Kubernetes.

In passato come ora, Spring ha utilizzato le librerie Netflix come Eureka in qualità di registro servizi e Ribbon come un programma di bilanciamento del carico lato client. Negli ambienti Kubernetes, entrambi questi ruoli sono soddisfatti da Kubernetes stesso, rendendo ridondanti le funzionalità a livello applicazione. Spring Cloud Kubernetes adatta le astrazioni Spring come `DiscoveryClient` ai sottostanti meccanismi Kubernetes.

Fai attenzione se utilizzi il rilevamento Ribbon tramite Spring Cloud Kubernetes in un cluster con Istio installato e dove Istio è utilizzato per influenzare l'instradamento dei servizi. Il bilanciamento del carico di lavoro client implica che il client desidera selezionare il servizio di destinazione stesso, mentre Istio potrebbe tentare di instradare la chiamata altrove. Solo uno di questi metodi può prevalere, con un conseguente disaccordo su come avrebbe dovuto essere instradata una chiamata. Questa combinazione dovrebbe essere evitata, nei limiti del possibile.
{: note}

Inoltre, Spring Cloud Kubernetes offre l'integrazione tra mappe di configurazione e segreti Kubernetes e bean di configurazione `@Autowired` Spring. Ciò include le strategie per gestire le modifiche dinamiche alle mappe di configurazione e ai segreti apportate mentre l'applicazione è in esecuzione.

Infine, Spring Cloud Kubernetes aumenta l'endpoint di integrità Spring Boot Actuator predefinito per includere informazioni aggiuntive correlate alla distribuzione.

Per ulteriori informazioni, consulta la [documentazione di Spring Cloud Kubernetes](https://cloud.spring.io/spring-cloud-static/spring-cloud-kubernetes/2.1.0.RC1/single/spring-cloud-kubernetes.html){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno")


<!--
### Spring Cloud Streams
{: #spring-cloud-streams}


:FIXME:
-->
