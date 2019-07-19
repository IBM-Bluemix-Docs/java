---

copyright:
  years: 2019
lastupdated: "2019-06-10"

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

Die Spring-Plattform ist ein Ökosystem von Projekten, mit dem die Erstellung von Java&trade;-Anwendungen vereinfacht werden soll. Gewöhnlich bezieht sich der Begriff "Spring" auf das Spring-Framework. Er wird jedoch auch allgemein in Bezug auf Projekte verwendet, die Bestandteil der Spring-Technologien sind oder solche Technologien (z. B. Spring Boot) verwenden.

## Spring Framework
{: #spring-framework}

Das Spring-Framework wurde im Jahr 2002 eingeführt und etwa alle 3 Jahre wird eine Hauptversion freigegeben. Das Framework umfasst eine große Gruppe von Komponenten (die als Spring-Module bezeichnet werden), die alles abdecken, von REST-Endpunkten bis hin zu Datenbankabstraktionen. Mit seiner neuesten Veröffentlichung im Jahr 2017 führte Spring Framework 5 eine aktualisierte JDK-Unterstützung und WebFlux ein. WebFlux ist ein neues Modul für reaktive Programmierung basierend auf [Project Reactor](https://projectreactor.io/){: new_window}![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link").

Spring Framework 4.3 ist das aktuelle Release von Spring Framework 4.x und bietet Unterstützung bis 2020. Anwendungen, die ältere Versionen von Spring verwenden, müssen auf Spring Framework 5 migriert werden. 

Die umfassende Dokumentation für Spring Framework finden Sie hier:

* [Spring Framework-Referenzhandbuch für 5.1.x](https://docs.spring.io/spring/docs/5.1.x/spring-framework-reference/){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link")
* [Spring Framework-Referenzhandbuch für 4.3.x](https://docs.spring.io/spring/docs/4.3.x/spring-framework-reference/){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link")
* [Upgrade auf Spring Framework 5.x](https://github.com/spring-projects/spring-framework/wiki/Upgrading-to-Spring-Framework-5.x){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link")

## Spring Boot
{: #spring-boot}

Spring Boot wurde im Jahr 2014 eingeführt, um "Webanwendungsarchitekturen ohne Container zu verbessern". Ziel war, die Konfiguration anstelle in XML in Code durchzuführen. Basierend auf einer strikten Ansicht darüber, welche Technologien verwendet werden sollen, stellt Spring Boot Mechanismen für die Erstellung von Apps bereit. Spring Boot-Apps sind Spring-Apps und verwenden ein speziell auf dieses Ökosystem ausgerichtetes Programmiermodell. 

Spring Boot basiert auf dem Konvention-vor-Konfiguration-Prinzip und zum Aktivieren von Funktionen wird eine Kombination aus Annotationen und Klassenpfaderkennung verwendet. Standardmäßig werden Spring Boot-Anwendungen in eine eigenständige ausführbare JAR-Datei integriert, die alle erforderlichen Abhängigkeiten (einschließlich eines eingebetteten Tomcat-Servers) enthält. Anwendungen können alternativ als WAR-Dateien gepackt werden, die auf einem App-Server bereitgestellt werden. 

Die aktuelle Spring Boot-Version ist 2.0, die 2018 veröffentlicht wurde. Die Wartung der Unterversion 1.5.x wird im August 2019 eingestellt.

Die umfassende Dokumentation für Spring Boot finden Sie hier:

* [Spring Boot-Referenzhandbuch für 2.1.x](https://docs.spring.io/spring-boot/docs/2.1.x/reference/){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link")
* [Spring Boot-Referenzhandbuch für 1.5.x](https://docs.spring.io/spring-boot/docs/1.5.x/reference/){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link")

## Spring Framework oder Spring Boot auswählen
{: #spring-framework-or-boot}

Wenn Sie für neue cloudnative Apps das Spring Framework verwenden möchten, können Sie auch Spring Boot verwenden. Spring Boot vereinfacht die Konfiguration und Assemblierung von Spring-basierten Apps und stellt Funktionen wie Spring Actuator bereit, die das Erstellen von [cloudnativen Apps](/docs/java?topic=cloud-native-overview#overview) vereinfachen. 

### Spring Boot Actuator
{: #spring-boot-actuator}

Spring Boot Actuator definiert eine Sammlung von Endpunkten, die für die Überprüfung einer aktiven Spring-App nützlich sind. Eine Aktivierung dieser Endpunkte kann durch das einfache Hinzufügen einer einzelnen Abhängigkeit zur App erreicht werden. 

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```
{: codeblock}

Die Struktur und das Verhalten von Spring Boot Actuator haben sich zwischen Spring Boot 1 und Spring Boot 2 erheblich verändert. Der Actuator in Boot 1 existiert neben der App mit einer separaten Konfiguration und gesonderten Sicherheitsmechanismen. Die Version von Boot 2 ist leistungsfähiger und verwendet ein Sicherheitsmodell, das mit dem Rest der Spring-App integriert ist. 

Die Verhaltensänderungen sind relevant, wenn Sie eine Spring Boot 1.x-App auf Spring Boot 2.0 migrieren. Weitere Informationen finden Sie im Abschnitt zu Actuator im [Handbuch für die Migration von Boot 1 auf Boot 2](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.0-Migration-Guide#spring-boot-actuator){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link").
{: tip}

In Boot 2 fungiert der Actuator als Mini-REST-Framework. Alle Endpunkte sind dem Kontext `/actuator` zugeordnet. Es werden Endpunkte für die Durchführung von Statusprüfungen oder die Erfassung von Metriken sowie App- oder anderen Umgebungsinformationen bereitgestellt. Eine vollständige Liste finden Sie in der [Actuator-Dokumentation](https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/html/production-ready-features.html#production-ready){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link").

Es ist einfach, einen eigenen Actuator-Endpunkt unter Verwendung einer Spring-Annotation des Typs `@Component` zu erstellen:

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

Spring Boot verfügt über zwei Steuerelemente, um zu konfigurieren, welche Actuators zur Laufzeit aktiv sind. Actuators können "enabled" (aktiviert) und "exposed" (zugänglich) sein. Damit er erreichbar ist, muss ein Actuator beides sein. Standardmäßig sind viele Endpunkte aktiviert, aber nur die für Status und Informationen sind zugänglich. Wenn die Spring-Sicherheit für die App aktiviert ist, muss außerdem der Zugriff auf die Actuator-Endpunkte explizit zugelassen werden. 

Der Actuator im Spring-Boot 1 verfügt über eine eigene Sicherheitskonfiguration, die in der Regel in der Datei 'application.properties' konfiguriert ist. Anstelle von 'enabled' und 'exposed' stehen die Optionen 'enabled' und 'sensitive' (sensibel) zur Verfügung. Sensible Endpunkte erfordern eine Authentifizierung, wenn sie über HTTP bedient werden. Standardmäßig ist der '/metrics'-Endpunkt von Spring Boot Actuator in Spring Boot 1 aktiviert, wird jedoch als sensibel betrachtet und muss daher authentifiziert oder auf nicht sensibel festgelegt werden. In einigen Umgebungen kann die Konfiguration der Spring-Sicherheit die richtige Antwort sein, aber in Kubernetes bleiben die Metrikendpunkte intern im Cluster. Weitere Informationen finden Sie in der [Dokumentation zum Actuator von Spring Boot 1](https://docs.spring.io/spring-boot/docs/1.5.2.RELEASE/reference/htmlsingle/#production-ready){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link").
{: note}

## Spring Cloud
{: #spring-cloud}

Spring Cloud ist eine Sammlung von Integrationen zwischen Cloud-Technologien von Drittanbietern und dem Spring-Programmiermodell. Sie zielt darauf ab, Entwickler bei der Erstellung von Spring-Apps für die Bereitstellung in der Cloud zu unterstützen. Die von Spring Cloud erzielte großflächige Abdeckung wird durch die modulare Struktur ermöglicht. Jedes Projekt in Spring Cloud ist auf eine bestimmte Technologie oder eine Gruppe von Technologien ausgerichtet. 

Spring Cloud-Projekte folgen dem allgemeinen Spring-Prinzip Konvention-vor-Konfiguration. Die meisten Funktionen werden aktiviert, indem die richtige Abhängigkeit zur Buildzeit hinzugefügt wird.

Weitere Informationen zu Spring Cloud-Projekten finden Sie auf der offiziellen [Site für Spring Cloud-Projekte](https://spring.io/projects/spring-cloud){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link"). 

### Spring Cloud mit Kubernetes
{: #spring-cloud-kubernetes}

Spring Cloud Kubernetes ist ein Spring Cloud-Projekt, mit dem verschiedene Tasks für in Kubernetes ausgeführte Spring-Apps vereinfacht werden sollen. Spring Cloud Kubernetes bietet Implementierungen gängiger Spring-Schnittstellen, die Kubernetes-Konzepten und -Services zugeordnet werden.

In der Vergangenheit hat Spring Netflix-Bibliotheken wie Eureka als Service-Registry und Ribbon als eine clientseitige Lastausgleichsfunktion verwendet. In Kubernetes-Umgebungen werden diese beiden Rollen von Kubernetes selbst erfüllt, wodurch die Funktionen auf Anwendungsebene redundant sind. Spring Cloud Kubernetes passt Spring-Abstraktionen wie `DiscoveryClient` an zugrunde liegende Kubernetes-Mechanismen an.

Lassen Sie bei der Verwendung der Ribbon-Erkennung über Spring Cloud Kubernetes in einem Cluster, in dem Istio installiert ist und sich auf die Serviceweiterleitung auswirkt, Vorsicht walten. Der clientseitige Lastausgleich impliziert, dass die Clientseite den Zielservice selbst auswählen möchte, während Istio möglicherweise versucht, den Aufruf an eine andere Stelle weiterzuleiten. In diesem Fall treten Konflikte in Bezug auf die Weiterleitung eines Aufrufs auf. Diese Kombination sollte unbedingt vermieden werden.
{: note}

Darüber hinaus bietet Spring Cloud Kubernetes eine Integration von Kubernetes-`ConfigMaps` und -`Secrets` mit `@Autowired`-Konfigurations-Beans in Spring. Außerdem werden Strategien für die Behandlung dynamischer Änderungen an `ConfigMaps` und `Secrets`, die während der Ausführung der App vorgenommen werden, zur Verfügung gestellt. 

Schließlich erweitert Spring Cloud Kubernetes den Standardendpunkt für den Status von Spring Boot Actuator, um zusätzliche Informationen in Bezug auf die Implementierung einzuschließen.

Weitere Informationen finden Sie in der [Dokumentation zu Spring Cloud Kubernetes](https://cloud.spring.io/spring-cloud-static/spring-cloud-kubernetes/2.1.0.RC1/single/spring-cloud-kubernetes.html){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link")


<!--
### Spring Cloud Streams
{: #spring-cloud-streams}


:FIXME:
-->
