---

copyright:
  years: 2019
lastupdated: "2019-04-22"

keywords: dependency injection, cdi jakarta, cdi beans, cdi scopes, bean lifecycle, context injection microprofile, microprofile cdi

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

# CDI (Contexts and Dependency Injection, JSR-365)
{: #mp-cdi}

CDI è un elemento centrale sia in Jakarta-EE che in MicroProfile, poiché è utilizzato per collegare tra loro diversi componenti. CDI definisce i **contesti** per specificare e gestire i cicli di vita dei bean e utilizza l'**inserimento di dipendenze** per soddisfare le dipendenze dichiarate in modo dinamico e sicuro rispetto ai tipi. CDI utilizza anche un modello di eventi e intercettori debolmente accoppiati e fornisce un potente e flessibile meccanismo di estensione portatile per altri framework per definire i loro bean CDI o aggiornare i componenti esistenti.

Le specifiche MicroProfile (ad esempio, MicroProfile Config, MicroProfile JWT) assumono un approccio che dà la precedenza a CDI, basandosi sul meccanismo di estensione CDI per potenziare gli standard esistenti (come JAX-RS) con nuove funzionalità. CDI 1.2 fa parte della release MicroProfile 1.0 e seguenti (MicroProfile 2.0 viene portato a CDI 2.0).

## Bean CDI
{: #cdi-bean}

CDI opera sui _bean_. I bean CDI vengono creati utilizzando le [annotazioni che definiscono i bean](https://docs.jboss.org/cdi/spec/2.0/cdi-spec.html){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno"). Quasi tutti i POJO (Plain Old Java&trade Object) che hanno un constructor senza alcun argomento o un constructor con l'annotazione `@Inject` possono essere un bean.

Le annotazioni che definiscono i bean CDI devono essere rilevate. La scansione delle annotazioni CDI può essere abilitata utilizzando un file `beans.xml` nella cartella `META-INF` per un archivio .jar o nella cartella `WEB-INF` per un archivio .war. Questo file può essere vuoto o può avere al suo interno qualcosa di simile al seguente contenuto:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://xmlns.jcp.org/xml/ns/javaee"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
            http://xmlns.jcp.org/xml/ns/javaee/beans_1_1.xsd"
      version="1.1" bean-discovery-mode="all">
  // enable interceptors; decorators; alternatives
</beans>
```
{: codeblock}

La presenza di un file `beans.xml` vuoto o di un `beans.xml` con `bean-discovery-mode="all"`, rende tutti i potenziali oggetti dei bean CDI. Altrimenti, solo gli oggetti con le annotazioni che definiscono i bean CDI sono dei bean CDI.

## Ambiti CDI
{: #cdi-scopes}

Le annotazioni del tipo di ambito CDI controllano il ciclo di vita del bean:

* Utilizza l'annotazione di livello classe `@ApplicationScoped` se un'istanza deve durare fino a quando l'applicazione è attiva.
* Utilizza l'annotazione di livello classe `@RequestScoped` se deve essere creata una nuova istanza del bean per ogni richiesta (una richiesta servlet, ad esempio).
* Utilizza l'annotazione di livello classe `@Dependent` se la nuova istanza appartiene a un altro oggetto. Un'istanza di un bean dipendente non viene mai condivisa tra diversi client o diversi punti di inserimento. Ne viene eseguita l'istanziazione quando l'oggetto a cui appartiene viene creato e ne viene eseguita l'eliminazione quando l'oggetto a cui appartiene viene eliminato.
* Se non viene definita alcuna annotazione di livello classe, `@Dependent` è il valore predefinito.

Il contenitore CDI crea ed elimina le istanze bean in base al loro ambito definito, ne esegue l'associazione al contesto appropriato e ne esegue quindi l'inserimento come dipendenze in altri oggetti.

## Inserimento di bean CDI
{: #cdi-inject}

CDI inserisce i bean definiti negli altri componenti mediante `@Inject`. Ad esempio, il seguente POJO `MyBean` è un bean CDI.

```java
@ApplicationScoped
public class MyBean {
    int i=0;
    public String sayHello() {
        return "MyBean hello " + i++;
    }
}
```
{: codeblock}

Poiché è `@ApplicationScoped`, viene creata solo una singola istanza. Di seguito, `MyRestEndPoint` è `@RequestScoped`, il che significa che viene creata un'istanza per ciascuna richiesta. CDI inserisce la stessa istanza `MyBean` in ognuna di esse.

```java
@RequestScoped
@Path("/hello")
public class MyRestEndPoint {
    @Inject MyBean bean;

    @GET
    @Produces (MediaType.TEXT_PLAIN)
    public String sayHello() {
        return bean.sayHello();
    }
}
```
{: codeblock}

CDI sta gestendo il ciclo di vita di questi bean:

* Utilizza un metodo annotato con `@PostConstruct` invece di un constructor per inizializzare il tuo bean. Ne viene eseguito il richiamo dopo che il contenitore CDI inserisce le dipendenze e il bean associato al contesto appropriato.
* Utilizza un metodo annotato con `@PreDestroy` per ripulire il tuo bean. Ne viene eseguito il richiamo prima che le dipendenze vengano rimosse.
