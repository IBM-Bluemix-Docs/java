---

copyright:
  years: 2019
lastupdated: "2019-05-20"

keywords: java logging, log level java, debug java, json log java, json log help, kibana liberty, liberty messages

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

# Protokollierung
{: #mp-logging}

Der empfohlene Ansatz für die Protokollierung mit MicroProfile-Anwendungen ist der Java&trade;-Protokollierungsstandard JSR-47. Sie können dazu die folgenden Importe durchführen:

```java
import java.util.logging.Level;
import java.util.logging.Logger;
```
{: codeblock}

Erstellen Sie dann eine Instanz einer Protokollfunktion auf der Klassenebene:

```java
private static Logger logger = Logger.getLogger(PortfolioService.class.getName());
```
{: codeblock}

Fügen Sie Aufrufe zur Instanz der `Protokollfunktion` vor und nach und in den Operationen in Ihrem Code hinzu. Die Methoden der Protokollierungsschnittstelle (`Logger`) selbst sind benannt, um die Wichtigkeit oder 'Ebene' der protokollierten Informationen anzugeben.

```java
logger.info("Creating portfolio for "+owner);
logger.warning("Unable to send message to JMS provider. Continuing without notification of change in loyalty level.");
```
{: codeblock}

Die Protokollebene wird angezeigt, wenn diese Nachrichten an die Konsole ausgegeben werden.

```
[INFO] Creating portfolio for John

[WARNING] Unable to send message to JMS provider. Continuing without notification of change in loyalty level.
```
{: screen}

Protokollebenen bieten Ihnen die Flexibilität, dynamisch auszuwählen, welche Protokolle Ihre Anwendung schreibt. Durch dieses Feature können Sie Protokollcode schreiben, der sowohl den allgemeinen Anwendungsstatus als auch den detaillierten Debuginhalt ohne Aufwand beschreibt. Sie können also den ausführlicheren Debuginhalt herausfiltern, bis Sie ihn benötigen. Die Protokollebene `info` ist in der Regel die minimale Ausgabeebene, gefolgt von `fine`, `finer`, `finest` und `debug`.

Wenn ein Protokolleintrag mehrere Codezeilen erfordert oder teure Operationen, wie z. B. Verkettung von Zeichenfolgen, erforderlich sind, müssen Sie sie mit einem Test überprüfen, um festzustellen, ob die Protokollebene aktiviert ist. Dadurch wird sichergestellt, dass die Anwendung nicht wichtige Zeit damit verbringt, Protokollnachrichten zu erstellen, die letztendlich wieder herausgefiltert werden. Im folgenden Beispiel wird die gewünschte Protokollebene `fine` aktiviert, bevor die Nachrichtenausgabe erstellt wird.

```java
if (logger.isLoggable(Level.FINE)) {
    StringWriter writer = new StringWriter();
    exception.printStackTrace(new PrintWriter(writer));
    logger.fine(writer.toString());
}
```
{: codeblock}

Weitere Informationen zu Protokollebenen und Konfigurationsdetails finden Sie im [Leitfaden zur Fehlerbehebung in WebSphere Liberty](https://www.ibm.com/support/knowledgecenter/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/rwlp_logging.html){: new_window}![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") und in der [API-Dokumentation zu 'java.util.logging' ](https://docs.oracle.com/javase/8/docs/api/java/util/logging/package-summary.html){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link").

## JSON-Protokollierung mit Liberty
{: #mp-json-logging}

Liberty unterstützt die JSON-formatierte Protokollierung. Wenn diese Option aktiviert ist, werden Protokollnachrichten im JSON-Format in die Konsole geschrieben. Aktivieren Sie diese Option unter Verwendung der folgenden Protokollierungszeilengruppe in der Datei `server.xml`:

```xml
<logging consoleLogLevel="INFO" consoleFormat="json" consoleSource="message,trace,accessLog,ffdc" />
```
{: codeblock}

Obwohl `accessLog` in der Liste der Konsolenquellen enthalten ist, muss die HTTP-Zugriffsprotokollierung aktiviert sein, bevor diese Protokolle in die Konsole geschrieben werden. Das folgende Snippet zeigt, wie das Unterelement `accessLogging` zum Element `httpEndpoint` in der Datei `server.xml` hinzugefügt wird:

```xml
<httpEndpoint id="defaultHttpEndpoint" host="\*" httpPort="9080" httpsPort="9443">
  <accessLogging
    filepath="${server.output.dir}/logs/http_defaultEndpoint_access.log"
    logFormat='%h %u %t "%r" %s %b %D %{User-agent}i'>
  </accessLogging>
</httpEndpoint>
```
{: codeblock}

Wenn Sie nun diesen Code zu Ihrer Anwendung hinzufügen:

```java
if (logger.isLoggable(Level.AUDIT)) {
    logger.audit("Initialization complete");
}
```

weisen die Protokolle die folgende Ausgabe auf:

```json
{ "type":"liberty_message",
  "host":"trader-54b4d579f7-4zvzk",
  "ibm_userDir":"\/opt\/ol\/wlp\/usr\/",
  "ibm_serverName":"defaultServer",
  "ibm_datetime":"2018-06-21T19:23:21.356+0000",
  "ibm_threadId":"00000028",
  "module":"com.trader.Main",
  "loglevel":"AUDIT",
  "message":"Initialization complete"}
```
{: codeblock}

### JSON-Protokollausgabe lesen
{: #mp-json-log-output}

Die gesamte JSON-Ausgabe ist zwar hilfreich für die Protokollspeicherung und -suche, jedoch nicht so einfach zu lesen. Sie können den Inhalt des Protokolls in einem Terminalfenster unter Verwendung des Befehls `kubectl` überprüfen. Das Befehlszeilentool `jq` kann Sie dabei unterstützen.

Mit dem Befehl `jq` können Sie nach unten filtern und sich auf das Feld oder die Felder konzentrieren, die Sie benötigen. Wenn Sie das Feld `message` anzeigen möchten und alles andere herausfiltern möchten, ziehen Sie das folgende Beispiel zurate:

```
kubectl logs trader-54b4d579f7-4zvzk -n stock-trader -c trader | grep message | jq .message -r
```
{: pre}

Liberty verfügt über einige einfache Konsolennachrichten, die nicht im JSON-Format vorliegen. Sie können den Befehl `grep` verwenden, um sicherzustellen, dass mit `jq` speziell für Zeilen ein Parsing durchgeführt wird, die ein Nachrichtenfeld enthalten.

## Zusätzliche Funktionen
{: #mp-log-features}

Richtlinien für die Verwendung von Protokollebenen, wie z. B. die Verwendung von `logger.info` oder `logger.fine`, sind Aspekte, anhand derer jede Organisation oder jedes Projekt Entscheidungen treffen muss. Im Allgemeinen sind diese Schnittstellen in nahezu allen Projekten erforderlich und nützlich.

Ein bewährtes Verfahren ist die Verwendung von Umgebungsvariablen (die über Kubernetes-Konfigurationszuordnungen oder -Secrets in den Pod gespeist werden) in jedem relevanten Feld in der Datei `server.xml`. Durch diese Methode können Sie die Protokollierungskonfiguration ändern, ohne dass Sie das Docker-Image erneut erstellen und erneut bereitstellen müssen.

Wenn Sie z. B. Umgebungsvariablen verwenden möchten, um differenzierte Protokollierungsattribute festzulegen, würden Sie die Zeilengruppe

```xml
<logging consoleLogLevel="INFO" consoleFormat="json" consoleSource="message,trace,accessLog,ffdc" />
```
{: codeblock}

wie folgt ändern:

```xml
<logging consoleLogLevel="${env.LOG_LEVEL}" consoleFormat="${env.LOG_FORMAT}" consoleSource="${env.LOG_SOURCE}" />
```
{: codeblock}

Alternativ dazu können Sie die Umgebungsvariable `WLP_LOGGING_CONSOLE_FORMAT` verwenden, wie in der [Dokumentation zur Protokollierung und Traceerstellung](https://www.ibm.com/support/knowledgecenter/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/rwlp_logging.html){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") beschrieben. Diese Methode ähnelt dem vorherigen Beispiel: Sie können die Variable `WLP_LOGGING_CONSOLE_FORMAT` entweder auf `basic` (Standardwert) oder `json` setzen.

## Kibana-Dashboards für Liberty
{: #liberty-kibana}

Zusammen mit der neuen JSON-Protokollierungsfunktion stellt Liberty vordefinierte Kibana-Dashboards bereit, [die Sie von GitHub herunterladen können ](https://www.ibm.com/support/knowledgecenter/en/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/twlp_icp_json_logging.html){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link"). Folgen Sie den Anweisungen unter dem Link, um diese Dashboards zu installieren. Es sind nun zwei neue Dashboards verfügbar:

![Kibana-Dashboards](images/microprofile-logging-image4.png "Kibana-Dashboards")

Wenn Sie das Dashboard zur Problembestimmung auswählen, wird Folgendes angezeigt:

![Details zum Kibana-Dashboard](images/microprofile-logging-image5.png "Details zum Kibana-Dashboard")

Das Dashboard ist interaktiv. Wenn Sie beispielsweise in der Legende für das Widget **Liberty-Nachricht** die Option **INFO** auswählen, werden im Widget **Liberty-Nachrichtensuche** nur die Nachrichten des Typs `loglevel = INFO` herausgefiltert. Das Dashboard bindet die Protokolldaten aus allen Liberty-basierten Mikroservices ein und filtert andere Systemprotokolle heraus.

Es gibt noch mehr Kibana- und Grafana-Dashboards, die dem Liberty-Helmdiagramm zugeordnet sind. Diese sind als [Erweiterungen für das Liberty-Cloud-Pak](https://github.com/IBM/charts/tree/master/stable/ibm-websphere-liberty/ibm_cloud_pak/pak_extensions/dashboards){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") verfügbar.

## Nächste Schritte
{: #mp-logging-next-steps notoc}

Weitere Informationen zum Anpassen von Protokollnachrichten mit Appendern, Protokollebenen und Konfigurationsdetails finden Sie in der offiziellen [Spring Boot-Referenz für die Protokollierung](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-logging.html){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link").

Hier finden Sie Informationen zum Anzeigen der Protokolle in den folgenden Bereitstellungsumgebungen:

* [Kubernetes-Protokolle](https://kubernetes.io/docs/concepts/cluster-administration/logging/){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link")
* [{{site.data.keyword.openwhisk}}-Protokolle & -Überwachung](/docs/openwhisk?topic=cloud-functions-openwhisk_logs#openwhisk_logs)
* [{{site.data.keyword.cloud_notm}} Log Analysis](/docs/services/CloudLogAnalysis?topic=cloudloganalysis-log_analysis_ov#log_analysis_ov)
* [{{site.data.keyword.cloud_notm}} Private-ELK-Stack](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_2.1.0.2/manage_metrics/logging_elk.html){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link")
