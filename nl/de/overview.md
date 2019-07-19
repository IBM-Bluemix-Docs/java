---

copyright:
  years: 2019
lastupdated: "2019-06-10"

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

# Lernprogramm zur Einführung
{: #getting-started}

Java&trade;-Anwendungen gibt es in allen Formen und Größen; sie reichen von eigenständigen Apps bis hin zu Apps in Java&trade; EE/Jakarta EE, MicroProfile oder Spring. 

## OpenJDK mit Open J9
{: #openjdk}

Das Projekt OpenJDK&trade; ist die Open-Source-Referenzimplementierung der Programmiersprache Java&trade; und bietet API-Stabilität und Kompatibilität mit Benutzerprogrammen. Die [Plattform Java&trade; SE](https://docs.oracle.com/javase/8/docs/){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") besteht aus APIs und der Java&trade; Virtual Machine (JVM). Die JVM ist die Ausführungsengine, die für die Ausführung der Java&trade;-App verantwortlich ist. 

Das [Eclipse OpenJ9-Projekt](https://www.eclipse.org/openj9/index.html){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") stellt eine alternative JVM bereit, die die HotSpot-JVM in OpenJDK ersetzt. Die OpenJ9-JVM wurde im September 2017 von {{site.data.keyword.IBM}} als Open-Source veröffentlicht. Ihre Ursprünge reichen aber wesentlich weiter zurück. Seit mehr als einem Jahrzehnt liegt OpenJ9 über 3000 {{site.data.keyword.IBM}} Produkten zugrunde, darunter {{site.data.keyword.appserver_short}}. 

Zusammen bieten OpenJDK und Eclipse OpenJ9 eine verbesserte Leistung und Servicefreundlichkeit für cloudnative Java&trade;-Apps, die über herkömmliche HotSpot-basierte Java&trade;-Laufzeiten hinausgehen. 

### OpenJDK mit OpenJ9 herunterladen
{: #downloads}

Binärdateien für OpenJDK mit OpenJ9 sind auf dem Java&trade;-Community-Hub unter [AdoptOpenJDK](https://adoptopenjdk.net/releases.html?variant=openjdk8&jvmVariant=openj9){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") frei verfügbar. AdoptOpenJDK bietet vorgefertigte OpenJDK-Binärdateien aus einem umfassenden Open-Source-Satz von Build-Scripts und Infrastruktur. Docker-Images sind auch auf dem [Docker-Hub](https://hub.docker.com/u/adoptopenjdk){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") verfügbar. 

## Java-Frameworks
{: #frameworks}

OpenJDK und OpenJ9 bieten eine solide Basis für jede Java&trade;-App, aber häufig werden App-Frameworks für allgemeine Problemstellungen verwendet. Unternehmens-Apps werden beispielsweise in der Regel entweder mit [Java&trade; EE (und Jakarta EE)](/docs/java?topic=java-jee-overview#jakarta-ee) oder mit [Spring](/docs/java?topic=java-spring-overview) erstellt. [MicroProfile](/docs/java?topic=java-jee-overview#microprofile) bietet ein schnelleres Entwicklungstempo und Unterstützung für cloudnative Konzepte in Java&trade; EE. 

[{{site.data.keyword.runtime_liberty_full}}](/docs/java?topic=java-liberty) ist ein schneller, dynamischer und benutzerfreundlicher Java&trade;-Anwendungsserver, der auf dem  Open-Source-Projekt Open Liberty basiert. Er bietet Unternehmen eine flexible, zweckmäßige Laufzeitumgebung für Java&trade; EE- und Spring-Anwendungen. 
