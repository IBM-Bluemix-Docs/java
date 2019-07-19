---

copyright:
  years: 2019
lastupdated: "2019-04-04"

keywords: java ee, jakarta ee, microprofile overview, cloud native java, cloud native microprofile

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

# Übersicht
{: #jee-overview}

## Java EE und Jakarta EE
{: #jakarta-ee}

In Java&trade; EE7 und EE8 eingeführte Technologien eignen sich gut für die Erstellung von Mikroservices in Java&trade;. Sie unterstützen Annotationen, einfache Protokolle und asynchrone Interaktionsmuster. Die Dienstprogramme für den gemeinsamen Zugriff in Java&trade; EE7 stellen einen Mechanismus für die Verwendung von reaktiven Bibliotheken wie RxJava auf containerfreundliche Art und Weise zur Verfügung. 

Im September 2017 gab Oracle (unterstützt von {{site.data.keyword.IBM}} und Red Hat) bekannt, dass Java&trade; EE als Jakarta EE an die Eclipse Foundation übergeben wird. Oracle ist weiterhin Eigner aller Java&trade; EE-Komponenten bis einschließlich Java&trade; EE 8. Alle zukünftigen Entwicklungen dieser Enterprise Java&trade;-Plattform nach Java&trade; EE 8 unterliegen jedoch der Eclipse Foundation unter Führung der Jakarta EE-Arbeitsgruppe. Anfang 2018 startete Oracle die aufwändige Neulizenzierung und Übergabe der Java&trade; EE-Technologien einschließlich APIs, RIs, TCKs und der zugehörigen Produktdokumentation an das EE4J-Projekt. 

## MicroProfile
{: #microprofile}

Java&trade; EE bietet zwar eine solide Grundlage für die Erstellung von Mikroservices, es benötigte jedoch Technologien und Programmiermodelle für eine bessere Anpassung an Mikroserviceanwendungen. Aus diesem Grund haben IBM und andere Unternehmen zusammen daran gearbeitet, MicroProfile ins Leben zu rufen, einer offenen Onlinezusammenarbeit zwischen Entwicklern, der Community und Softwareanbietern.

Die MicroProfile-Community konzentriert sich auf schnelle Innovationen rund um Mikroservices und Enterprise Java&trade;. Diese Community erstellt und integriert Technologien, die sich am besten für cloudnative Apps gemäß den Architekturmustern von Mikroservices eignen. Jedes MicroProfile-Release enthält eine Reihe von Technologien. Das im Jahr 2016 freigegebene MicroProfile 1.0 enthält eine Untergruppe von Spezifikationen aus Java&trade; EE 7, die eine Reihe von Kernfunktionen für einfache Mikroservices bereitstellen: CDI 1.2, JAX-RS 2.0 und JSON-P 1.0. Die MicroProfile-Plattform umfasst Technologien, die auf die Herausforderungen von Cloud-Umgebungen zugeschnitten sind, wie Konfigurationsinjektion, Fehlertoleranz, Statusprüfungen, Metriken und offenes Tracing. Sie definiert zudem anbieterübergreifende Standards für die Arbeit mit OpenAPI-Definitionen und die Erstellung von REST-Clients, um die Nutzung von REST-APIs zu vereinfachen, sowie für die JWT Propagation, um die App-Sicherheit zu verbessern. 

Wenn Sie in der Open-Source-Gruppe mitarbeiten möchten, besuchen Sie die Seiten [https://microprofile.io/](https://microprofile.io/){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") oder [https://projects.eclipse.org/projects/technology.microprofile](https://projects.eclipse.org/projects/technology.microprofile){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link").
