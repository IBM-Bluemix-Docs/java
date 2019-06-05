---

copyright:
  years: 2019
lastupdated: "2019-05-20"

keywords: health check jax-rs, jax-rs endpoint, jax-rs status, readiness jax-rs, liveness jax-rs, microprofile health

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

# Statusprüfungen mit JAX-RS
{: #jaxrs-healthcheck}

Wie im vorherigen Abschnitt erläutert, bietet Kubernetes mehrere Methoden für die Integration von Statustests, einschließlich der Ausführung eines Befehls sowie der Netzüberprüfung über TCP- oder HTTP-Endpunkte. Obwohl es möglich ist, jeden dieser Tests in der Programmiersprache Java&trade; zu implementieren, wird hier im Detail auf eine JAX-RS-Implementierung von HTTP-Tests und MicroProfile Health eingegangen.

## JAX-RS-Readiness-Endpunkt definieren
{: #mp-readiness}

Ein minimaler Readiness-Endpunkt sieht in etwa wie folgendes Beispiel aus:

```java
@Path("ready")
public class ReadinessEndpoint {
  @GET
  @Produces(MediaType.APPLICATION_JSON)
  public Response testReadiness() {
    if ( ! ready ) {
      return Response.status(503).entity("{\"status\":\"DOWN\"}").build();
    }
    return Response.ok("{\"status\":\"UP\"}").build();
  }
}
```
{: codeblock}

Ein Endpunkt für eine Anwendung, die in WebSphere Liberty bereitgestellt ist, erreicht ohne großen Aufwand ein gewisses Maß an Bereitschaft ('readiness'). Liberty lässt Anforderungen an den Statusendpunkt mit einer entsprechenden Antwort vom Typ 503 fehlschlagen, bis die Anwendung ausgeführt wurde. Diese Basisimplementierung kann erweitert werden, um die erforderlichen Funktionen einzuschließen. Ziehen Sie beispielsweise in Betracht, CDI-Ressourcen des Typs `@ApplicationScoped` zu bewerten, um sicherzustellen, dass die Verarbeitung von Abhängigkeitsinjektionen abgeschlossen wurde. Nachdem der Test implementiert wurde, kann er aktiviert werden, indem der HTTP-Testtyp wie im vorherigen Abschnitt beschrieben konfiguriert wird.

Berücksichtigen Sie immer die Rolle und den Zweck der Fehlertoleranz: Von Mikroservices wird erwartet, dass sie Fehler und Ausfälle bei der Kommunikation mit Downstream-Services verarbeiten. Sie müssen Prüfungen für Funktionen einbeziehen, die keinen durchführbaren Fallback in einem Readiness-Test bieten.

## JAX-RS-Liveness-Endpunkt definieren
{: #jaxrs-liveness}

Ein Liveness-Test sollte mit Bedacht durchgeführt werden, da ein Fehler zu einer sofortigen Beendigung des Prozesses führt. Ein einfacher Liveness-Endpunkt kann in etwa wie folgt aussehen:

```java
@Path("liveness")
public class LivenessEndpoint {
    @GET
    @Produces(MediaType.APPLICATION_JSON)
    public Response testLiveness() {
      return Response.ok("{\"status\":\"UP\"}").build();
    }
}
```
{: codeblock}

Die Implementierung des Liveness-Tests kann erweitert werden, um die lokalen Bedingungen des Prozesses zu prüfen, die einen nicht wiederherstellbaren Prozessstatus angeben. Die Herausforderung bei diesen Prüfungen besteht jedoch häufig darin, dass der Server bei einer solchen Verarbeitung sehr wahrscheinlich normal arbeitet, um weitere Anforderungen durchführen zu können. Aus diesem Grund kann bei der Implementierung von Liveness-Tests das Motto 'Keep it simple' verfolgt werden. Nachdem die Liveness-Quellen codiert sind, um den Testendpunkt zu aktivieren, kann der HTTP-Liveness-Endpunkt in den Orchestrierungsquellen des Containers aktiviert werden, wie im letzten Kapitel erläutert.

Um Wiederanlaufzyklen zu vermeiden, muss das Attribut `initialDelaySeconds` für die Liveness-Prüfung länger sein als die längste erwartete Serverstartzeit. Für einen Java&trade;-Anwendungsserver, der normalerweise 30 Sekunden zum Starten benötigt, wählen Sie einen größeren Wert (z. B. 60 Sekunden) aus.

## MicroProfile Health-API verwenden
{: #mp-health-api}

Die [MicroProfile Health-API](https://www.ibm.com/support/knowledgecenter/en/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/twlp_microprofile_healthcheck.html){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") (mpHealth) definiert ein Framework, das die Implementierung von Statusprüfungen vereinfacht. In Version 1.0 der API `mpHealth` wird ein einzelner Endpunkt definiert. Die nächste Version wird zwei Endpunkte enthalten: einen für Liveness und einen für Readiness.

Um die MicroProfile Health-API mit Liberty zu verwenden, fügen Sie das Feature `mpHealth` zur Datei `server.xml` hinzu:

```xml
<featureManager>
   <feature>mpHealth-1.0</feature>
</featureManager>
```
{: codeblock}

Sie können dann den Status der Anwendung mit einem Browser überprüfen, indem Sie auf den Endpunkt `/health` zugreifen. Wenn alles in Ordnung ist, gibt der Code die Nutzdaten `{"outcome": "UP"}` zurück. Der folgende Code zeigt Ihnen, wie die APIs von MicroProfile Health verwendet werden, um anzugeben, ob der Pod normal arbeitet. Mit der Methode `call` können Entwickler einen Prüfungsalgorithmus bereitstellen, um den Allgemeinzustand eines Pods anzugeben. Eine Angabe, dass fast kein Speicherplatz mehr verfügbar ist, ist ein guter Indikator für den Zustand eines Pods. Die Überprüfung auf eine Downstream-Datenbankverbindung kann eine gute Überprüfung der Bereitschaft des Pods sein. Wenn es keine bestimmte Prüfung gibt, können Sie die folgenden Angaben verwenden, um anzugeben, ob ein Pod aktiv oder bereit ist.

```java
import org.eclipse.microprofile.health.Health;
import org.eclipse.microprofile.health.HealthCheck;
import org.eclipse.microprofile.health.HealthCheckResponse;

import javax.enterprise.context.ApplicationScoped;

@Health
@ApplicationScoped
public class HealthResource implements HealthCheck {

    @Override
    public HealthCheckResponse call() {
        return HealthCheckResponse.named("service-a").withData("service-a", "ok").up().build();
    }
}
```
{: codeblock}

Anschließend müssen Sie `livenessProbe` und `readinessProbe` für Kubernetes angeben, wie im folgenden Beispiel dargestellt.
```yaml
livenessProbe:
  exec:
    command: ["sh", "-c", "curl -s http://localhost:9080/health| grep -q service-a"]
  initialDelaySeconds: 60
readinessProbe:
  exec:
    command: ["sh", "-c", "curl -s http://localhost:9080/health | grep -q service-a"]
  initialDelaySeconds: 40
```
{: codeblock}

Wie bereits erwähnt, können Sie verschiedene Logiken verwenden, um Readiness und Liveness anzugeben. Sie können MicroProfile Health verwenden, um den Readiness-Test anzugeben und anschließend einen JAX-RS-Endpunkt für die Readiness-Prüfung zu erstellen (oder umgekehrt). In der nächsten Version von MicroProfile Health werden zwei Endpunkte bereitgestellt: Ein Endpunkt für Liveness und ein weiterer für Readiness.

Für Open Liberty existiert ein praktischer Leitfaden, der ein Beispiel dazu enthält, wie Sie angepasste Prüfungen zum MicroProfile Health-Endpunkt hinzufügen können. Sie finden diesen Leitfaden unter 'https://openliberty.io/guides/microprofile-health.html'.

Sie können das Standardverhalten des MicroProfile Health-Endpunkts überschreiben, wie in [MicroProfile-Vitalitätsprüfungen durchführen](https://www.ibm.com/support/knowledgecenter/en/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/twlp_microprofile_healthcheck.html){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") beschrieben.
