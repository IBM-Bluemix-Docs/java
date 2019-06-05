---

copyright:
  years: 2019
lastupdated: "2019-05-20"

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

# Open Liberty und WebSphere Liberty
{: #liberty}

[Open Liberty](https://openliberty.io/){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") ist eine einfache Open-Source-Java&trade; Runtime, die auf modularen *Features* aufbaut. [WebSphere Liberty](https://developer.ibm.com/wasdev/){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") ist eine kostenpflichtige Version von Open Liberty. 

Liberty-Features unterstützen die folgenden Anwendungsframeworks:

* [MicroProfile](https://microprofile.io/){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link"), ein Open-Source-Projekt, das neue Standards und APIs definiert, um die Erstellung von Mikroservices zu beschleunigen und zu vereinfachen.
* [Jakarta EE](https://jakarta.ee){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") und [Java EE](https://www.oracle.com/technetwork/java/javaee/overview/index.html){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link"), einschließlich Features für einzelne Spezifikationen, z. B. JNDI oder JAX-RS.
* [Spring Framework und Spring Boot](https://www.ibm.com/support/knowledgecenter/en/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/twlp_dep_springboot.html){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link"), einschließlich Mechanismen zum Erstellen von Containern aus umfangreichen JAR-Dateien von Spring Boot.

Es gibt eine Reihe praktischer Entwicklungshandbücher, die sich mit dem Erstellen von Mikroservices und cloudnativen Anwendungen mit Liberty beschäftigen. Diese finden Sie unter [https://openliberty.io/guides/](https://openliberty.io/guides/){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link").

## Optimiert für Docker
{: #liberty-optimized}

Wenn automatisierte Systeme wie Kubernetes Container-Images übertragen, spielt die Image-Größe eine wichtige Rolle. Unterstützend werden zur Lösung des Problems die Layer in Docker-Images in den Cache gestellt. Mit seiner modularen Architektur stellt Liberty eine effiziente Paketierungspipeline für Docker-Container zur Verfügung und ist somit hervorragend für Cloud-Umgebungen geeignet. Ein Basisimage kann verwendet werden, um viele Workloads zu unterstützen, die es ermöglichen, dass der Ressourcenbedarf von aktiven Containern abhängig von den Anforderungen des Service variieren kann.

Liberty bietet Tools und optimierte Unterstützung für die Konvertierung umfangreicher Spring Boot-JAR-Dateien in kompakte, optimierte Docker-Container, die die Vorteile der in den Cache gestellten Layer von Docker-Images nutzen, um Zyklus- und Publizierungszeiten zu verbessern. Iterative Neuerstellungen und Neuimplementierungen werden wesentlich schneller ausgeführt, wenn sich selten ändernden Bibliotheksabhängigkeiten in einen separaten Layer gestellt und nur die Anwendungsklassen im obersten Layer beibehalten werden. Weitere Informationen finden Sie in der Veröffentlichung zum [Erstellen von Dual-Docker-Images für Spring Boot-Apps](https://openliberty.io/blog/2018/07/02/creating-dual-layer-docker-images-for-spring-boot-apps.html){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link").

## Liberty-Docker-Images
{: #liberty-images}

Sowohl Open Liberty als auch WebSphere Liberty stellen verschiedene Basisimages bereit, die Sie als Ausgangspunkt für Ihre Java-basierten Mikroservices verwenden können. Es gibt Images, die einen geringeren Speicherbedarf haben, oder OpenJ9-VMs, bei denen verschiedene Features vorausgewählt sind. Beispiel:

1. `open-liberty:microProfile2` oder `websphere-liberty:microProfile2`
2. `open-liberty:javaee8` oder `websphere-liberty:javaee8`
3. `open-liberty:springBoot2` oder `websphere-liberty:springBoot2`

Unter [`websphere-liberty`](https://hub.docker.com/_/websphere-liberty/){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") oder [`open-liberty`](https://hub.docker.com/_/open-liberty/){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") in Docker Hub finden Sie die aktuelle Liste der Basisimages.

### Open Liberty und Docker
{: #openliberty-docker}

Wählen Sie das für Ihre Anwendung am besten geeignete Image aus der [Liste der Images in Docker Hub](https://hub.docker.com/_/open-liberty/){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link"), z. B. `open-liberty:microProfile`, aus und erstellen Sie eine Dockerfile für Ihren Service, der `FROM` erweitert. Fügen Sie die Befehle hinzu, um die App- und Serverkonfiguration in das neue Image zu kopieren. Beispiel:

```docker
FROM open-liberty:microProfile\
COPY server.xml /config/\
ADD Sample1.war /config/dropins/
```
{: codeblock}

Weitere Beispiele und funktionierenden Code finden Sie in den folgenden Open Liberty-Leitfäden:

* [Docker-Container zum Entwickeln von Mikroservices verwenden](https://openliberty.io/guides/docker.html){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link")
* [Anwendung in einem Docker-Container ausführen](https://openliberty.io/guides/getting-started.html#running-the-application-in-a-docker-container){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link")

### WebSphere Liberty und Docker
{: #wlp-docker}

Es gibt einige Unterschiede zwischen Open Liberty und der kommerziellen Version, WebSphere Liberty. Einer der wichtigsten Unterschiede für die Erstellung von Docker-Images ist, dass der Befehl `installUtility` in Open Liberty nicht verfügbar ist.

WebSphere Liberty unterstützt dieselben grundlegenden Anpassungsmuster, die Open Liberty für Docker-Images verwendet. Das inhärente modulare Design von Liberty macht es einfach (und typisch für Liberty), ein angepasstes Image zu erstellen, das eine Anwendung und die erforderliche Gruppe von Features enthält. WebSphere Liberty verfügt über ein Image für einen Feature-losen Kernel, [`websphere-liberty:kernel`](https://github.com/WASdev/ci.docker/blob/9d28dfba4d20596f89b393bc9e3ae8295feec469/ga/developer/kernel/Dockerfile){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link"), der eine solide Basis für ein richtiges angepasstes Image bildet.

Die [Image-Dokumentation für `websphere-liberty`](https://hub.docker.com/_/websphere-liberty/){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") beschreibt die folgende einfache dreizeilige Dockerfile, die zum Erstellen eines angepassten Image erforderlich ist:

```docker
FROM websphere-liberty:kernel
COPY server.xml /config/
RUN installUtility install --acceptLicense defaultServer
```
{: codeblock}

Das Tool `installUtility` sucht nach Features, die Sie in der Datei `server.xml` benötigen, die noch nicht in Ihrem Liberty-Image verfügbar sind. Anschließend werden diese Features heruntergeladen und in Ihrem Docker-Image installiert. Bei diesem Ansatz wird ein minimales Image erstellt, aber die implizite Featureliste in der Datei `server.xml` funktioniert nicht gut mit den Cache-Docker-Layern. Jede Änderung in der Datei `server.xml` macht den Layer mit den installierten Features ungültig, so dass die fehlenden Features bei der nächsten Image-Installation erneut installiert werden müssen.

Eine bessere Methode zum Erstellen von angepassten Images besteht darin, `installUtility` mit einer expliziten Liste der Features aufzurufen. Durch diese Methode wird zwar immer noch ein minimales Image erstellt, dieses wird sich jedoch zur Buildzeit besser verhalten, da der Layer des Images zwischengespeichert wird und unabhängig von Änderungen der Serverkonfiguration wiederverwendet werden kann. Mit dem folgenden Code wird beispielsweise ein angepasstes Image erstellt, das eine Untergruppe von Features für die Unterstützung einer Anwendung nutzt, die JAX-RS, JNDI und WebSockets verwendet:

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

Wie Sie Ihr Docker-Image strukturieren, hängt von einigen Faktoren ab:

1. In welchem Umfang soll das Basisimage wiederverwendbar oder angepasst sein?
    Mit diesen Schritten wird das kleinstmögliche Image erstellt, möglicherweise auf Kosten der Wiederverwendung, wenn die Features der einzelnen Anwendungen unterschiedlich sind. Die Verwendung von kleinen angepassten Images bedeutet jedoch, dass die Anwendung schnell über Docker-Hosts hinweg implementiert werden kann. Wenn Sie sich nicht sicher sind, beginnen Sie mit einem der vorgenerierten Images von Docker Hub, um eine möglichst hohe Wiederverwendung zu erreichen.
2. Wie oft aktualisieren Sie dieses Image?
    Wenn die Anwendung häufig aktualisiert wird, z. B. in einer CI/CD-Pipeline, empfiehlt es sich, das Image so zu strukturieren, dass sich der am häufigsten geänderte Layer (oft die Anwendungsklassen oder App-Binärdatei) ganz oben befindet. Durch diese Struktur wird die Anzahl der Layer reduziert, die neu erstellt werden müssen, wenn die Anwendung geändert und das Image erneut erstellt wird, wodurch die Build- und Implementierungszeiten beschleunigt werden.

Im Zweifelsfall folgen Sie als Einführung dem [Docker-Leitfaden von Open Liberty](https://openliberty.io/guides/docker.html){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link").
