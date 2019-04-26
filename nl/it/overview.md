---

copyright:
  years: 2019
lastupdated: "2019-04-01"

keywords: openjdk, open j9, download openjdk, java frameworks, adoptopenjdk, eclipse openj9, openj9 binaries, openjdk binaries, microprofile framework, jakarta

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

# Java su {{site.data.keyword.cloud_notm}}
{: #overview}

Le applicazioni Java&trade; sono disponibili in un'ampia gamma di modelli e dimensioni che va dalle applicazioni autonome alle applicazioni Java EE / Jakarta EE, MicroProfile o Spring.

## OpenJDK con Open J9
{: #openjdk}

Il progetto OpenJDK&trade; è l'implementazione di riferimento open source del linguaggio Java e fornisce compatibilità e stabilità delle API ai programmi utente. La [piattaforma Java SE](https://docs.oracle.com/javase/8/docs/){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno"), è composta da API e dalla JVM. La JVM è il motore di esecuzione responsabile dell'esecuzione dell'applicazione Java.

Il [progetto Eclipse OpenJ9](https://www.eclipse.org/openj9/index.html){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno") offre una JVM alternativa che sostituisce la JVM Hotspot in OpenJDK. La JVM OpenJ9 è stata resa disponibile come open source da {{site.data.keyword.IBM}} nel settembre del 2017, anche se per esso si può risalire a una data molto antecedente.OpenJ9 fornisce la sua tecnologia a 3000 prodotti {{site.data.keyword.IBM}}, incluso {{site.data.keyword.appserver_short}}, da più di dieci anni.

Insieme, OpenJDK e Eclipse OpenJ9 offrono prestazioni e servizi migliorati per le applicazioni Java native del cloud che vanno ben oltre i tradizionali runtime Java basati su Hotspot.

### Download di OpenJDK con OpenJ9
{: #downloads}

I file binari OpenJDK con OpenJ9 sono disponibili gratuitamente dall'hub della community Java in [AdoptOpenJDK](https://adoptopenjdk.net/releases.html?variant=openjdk8&jvmVariant=openj9){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno"). AdoptOpenJDK fornisce file binari OpenJDK precostruiti da una serie open source completa di script di build e di infrastrutture. Le immagini Docker sono disponibili anche su [Dockerhub](https://hub.docker.com/u/adoptopenjdk){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno").

## Framework Java
{: #frameworks}

OpenJDK e OpenJ9 offrono una base solida per qualsiasi applicazione Java, ma i framework applicativi sono spesso utilizzati per risolvere problemi comuni. Le applicazioni aziendali, ad esempio, di solito vengono create utilizzando [Java EE (e Jakarta EE)](/docs/java?topic=java-jee-overview#jakarta-ee) oppure [Spring](/docs/java?topic=java-spring-overview).  [MicroProfile](/docs/java?topic=java-jee-overview#microprofile) offre ritmo di sviluppo e supporto più veloci per i concetti nativi del cloud in Java EE.

[{{site.data.keyword.runtime_liberty_full}}](/docs/java?topic=java-liberty) è un server delle applicazioni Java veloce, dinamico e facile da utilizzare, basato sul progetto Open Liberty open-source. Offre un runtime di livello aziendale flessibile e appropriato per le applicazioni Java EE e Spring.
