---

copyright:
  years: 2019
lastupdated: "2019-03-15"

keywords: open liberty java, websphere liberty java, jakarta, webpshere docker, liberty docker, liberty docker images, installutility, microprofile java, dual layer docker, develop microservices

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

# Open Liberty e WebSphere Liberty
{: #liberty}

[Open Liberty](https://openliberty.io/){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno") è un runtime Java open-source leggero creato utilizzando le *funzioni* modulari. [WebSphere Liberty](https://developer.ibm.com/wasdev/){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno") è una versione commerciale di Open Liberty. Le funzioni Liberty supportano i seguenti framework di applicazioni:

* [MicroProfile](https://microprofile.io/){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno"), un progetto open source che definisce nuovi standard e API per accelerare e semplificare la creazione di microservizi.
* [Jakarta EE](https://jakarta.ee){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno") e [Java EE](https://www.oracle.com/technetwork/java/javaee/overview/index.html){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno"), incluse le funzioni per le singole specifiche, come JNDI o JAX-RS.
* [Spring Framework e Spring Boot](https://www.ibm.com/support/knowledgecenter/en/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/twlp_dep_springboot.html){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno"), inclusi i meccanismi per creare contenitori compatti dai fat jar di Spring Boot.

È disponibile un'ampia gamma di pratiche guide per gli sviluppatori per creare microservizi e applicazioni native del cloud con Liberty all'indirizzo [https://openliberty.io/guides/](https://openliberty.io/guides/){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno").

## Ottimizzato per Docker
{: #liberty-optimized}

Quando i sistemi automatizzati come Kubernetes eseguono il push di immagini del contenitore, la dimensione dell'immagine inizia ad essere importante. I livelli nelle immagini Docker vengono memorizzati nella cache per poterti essere di supporto. Data la sua architettura modulare, Liberty offre una pipeline di impacchettamento efficiente per i contenitori Docker, rendendolo un'ottima struttura operativa per gli ambienti cloud. Un'immagine di base può essere utilizzata per supportare molti carichi di lavoro, consentendo nel contempo al footprint della risorsa dei contenitori in esecuzione di variare in base ai requisiti del servizio.

Liberty offre strumenti e supporto ottimizzato per convertire i fat jar di Spring Boot in contenitori Docker ottimizzati e compatti che sfruttano i livelli dell'immagine Docker memorizzati nella cache per migliorare la durata del ciclo e i tempi di pubblicazione. Eseguendo il push di dipendenze della libreria che cambiano raramente in un livello separato e conservando solo le classi dell'applicazione nel livello superiore, le ricreazioni e le ridistribuzioni iterative saranno molto più veloci. Vedi ulteriori informazioni su [Creating Dual Layer Docker images for Spring Boot apps](https://openliberty.io/blog/2018/07/02/creating-dual-layer-docker-images-for-spring-boot-apps.html){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno").

## Immagini Liberty Docker
{: #liberty-images}

Open Liberty e WebSphere Liberty forniscono una gamma di immagini di base che puoi utilizzare come punto di partenza per i tuoi microservizi basati su Java. Ci sono immagini che utilizzano footprint o VM OpenJ9 di piccole dimensioni con funzioni preselezionate diverse. Ad esempio:

1. open-liberty:microProfile2 o websphere-liberty:microProfile2
2. open-liberty:javaee8 o websphere-liberty:javaee8
3. open-liberty:springBoot2 o websphere-liberty:springBoot2

Controlla [websphere-liberty](https://hub.docker.com/_/websphere-liberty/){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno") o [open-liberty](https://hub.docker.com/_/open-liberty/){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno") su Docker Hub per l'elenco più aggiornato delle immagini di base.

### Open Liberty e Docker
{: #openliberty-docker}

Scegli l'immagine più appropriata per la tua applicazione dall'[elenco di immagini sul Docker Hub](https://hub.docker.com/_/open-liberty/){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno") (ad esempio, `open-liberty:microProfile`) e crea un Dockerfile per il tuo servizio che esegue l'estensione da (`FROM`) esso. Aggiungi i comandi per copiare la tua configurazione dell'applicazione e del server nella nuova immagine. Ad esempio:

```docker
FROM open-liberty:microProfile\
COPY server.xml /config/\
ADD Sample1.war /config/dropins/
```
{: codeblock}

Per altri esempi e altro codice funzionante, consulta le seguenti guide di Open Liberty:

* [Using Docker containers to develop microservices](https://openliberty.io/guides/docker.html){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno")
* [Running an application in a Docker container](https://openliberty.io/guides/getting-started.html#running-the-application-in-a-docker-container){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno")

### WebSphere Liberty e Docker
{: #wlp-docker}

Ci sono alcune differenze tra Open Liberty e la versione commerciale, WebSphere Liberty. Una delle più significative per la creazione delle immagini Docker è che il comando `installUtility` non è disponibile in Open Liberty.

WebSphere Liberty supporta gli stessi modelli di personalizzazione di base di OpenLiberty per le immagini Docker, ma la progettazione intrinsecamente modulare di Liberty rende semplice (e normale) la creazione di un'immagine personalizzata che contiene un'applicazione e la serie specifica di funzioni che richiede. WebSphere Liberty dispone di un'immagine per un kernel senza funzione, [websphere-liberty:kernel](https://github.com/WASdev/ci.docker/blob/9d28dfba4d20596f89b393bc9e3ae8295feec469/ga/developer/kernel/Dockerfile){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno"), che costituisce una solida base per un'immagine effettivamente personalizzata.

La [documentazione sull'immagine websphere-liberty](https://hub.docker.com/_/websphere-liberty/){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno") descrive il seguente Dockerfile semplice di 3 righe necessario per creare un'immagine personalizzata:

```docker
FROM websphere-liberty:kernel
COPY server.xml /config/
RUN installUtility install --acceptLicense defaultServer
```
{: codeblock}

Lo strumento `installUtility` ricerca le funzioni di cui hai bisogno nel tuo file `server.xml` che non sono già disponibili nella tua immagine Liberty. Scaricherà quindi tali funzioni e le installerà nella tua immagine Docker. Questo approccio creerà un'immagine minima, ma l'elenco di funzioni implicite nel file `server.xml` non funzionerà correttamente con i livelli Docker memorizzati nella cache. Qualsiasi modifica al file `server.xml` invaliderà il livello con le funzioni installate, richiedendo la reinstallazione delle funzioni mancanti alla successiva creazione dell'immagine.

Un modo migliore per creare immagini personalizzate è quello di richiamare `installUtility` con un elenco esplicito di funzioni. Ciò creerà ancora un'immagine minima, ma funzionerà molto meglio al momento della creazione, poiché il livello dell'immagine può essere memorizzato nella cache e riutilizzato indipendentemente dalle modifiche alla configurazione del server. Ad esempio, quanto segue creerà un'immagine personalizzata utilizzando una sottoserie delle funzioni per supportare un'applicazione che utilizza JAX-RS, JNDI e WebSocket:

```docker
FROM websphere-liberty:kernel
RUN /opt/ibm/wlp/bin/installUtility install --acceptLicense \
  cdi-1.2 \
  concurrent-1.0 \
  jaxrs-2.0 \
  jndi-1.0 \
  ssl-1.0 \
  websocket-1.1
COPY server.xml /config/
```
{: codeblock}

Il modo in cui strutturi la tua immagine Docker dipende da alcuni fattori:

1. In che misura desideri che la tua immagine di base sia riutilizzabile o personalizzata?
    Questi passi producono l'immagine più piccola possibile, potenzialmente a spese del riutilizzo se le funzioni di ogni applicazione sono diverse. Tuttavia, avere immagini personalizzate molto piccole significa che l'applicazione può essere distribuita rapidamente tra gli host Docker. Se non sei sicuro, inizia con uno delle immagini pregenerate dal Docker Hub per il riutilizzo massimo.
2. Con quale frequenza aggiorni questa immagine?
    Se l'applicazione viene aggiornata molto spesso, ad esempio in una pipeline CI/CD, è utile strutturare l'immagine in modo che il livello modificato più di frequente (spesso le tue classi applicazione o il file binario dell'applicazione) si trovi all'inizio. Ciò riduce il numero di livelli che devono essere ricreati quando l'applicazione viene modificata e l'immagine viene ricreata, velocizzando i tempi di creazione e di distribuzione.

In caso di dubbio, segui la [guida Open Liberty Docker](https://openliberty.io/guides/docker.html){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno") per iniziare.
