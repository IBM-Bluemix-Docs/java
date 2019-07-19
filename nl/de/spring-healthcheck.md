---

copyright:
  years: 2018, 2019
lastupdated: "2019-06-10"

keywords: health check spring, spring health endpoint, spring-boot-actuator, liveness probe spring, readiness probe spring, spring kubernetes probe

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

# Statusprüfungen mit Spring
{: #spring-healthcheck}

Der von Spring Boot Actuator bereitgestellte Statusendpunkt ist an den Anwendungslebenszyklus gebunden und ist erst dann erreichbar, wenn die Anwendung gestartet wird. Ebenso wird er inaktiviert, sobald die Anwendung gestoppt wird (dies geschieht vor der Beendigung des Prozesses). Spring Boot Actuator fügt automatisch Statusanzeiger für Technologien hinzu, die im Klassenpfad erkannt werden. Wenn Ihre Anwendung z. B. JDBC verwendet, enthält der Statusendpunkt einige Basistests, um sicherzustellen, dass der Sicherungsdatenspeicher erreicht werden kann. Der vom Actuator definierte Endpunkt ist aus diesem Grund besser geeignet als ein Readiness-Test.

Aktivieren Sie Spring Boot Actuator, indem Sie die Abhängigkeit `spring-boot-actuator` zur Datei `pom.xml` hinzufügen:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-actuator</artifactId>
</dependency>
```
{: codeblock}

Zwischen den verschiedenen Versionen von Spring Boot Actuator gibt es einige Unterschiede. In den beiden folgenden Abschnitten erfahren Sie, wie Sie in beiden Versionen von Spring Boot (beginnend mit der aktuellen Version 2) Liveness- und Readiness-Tests erstellen können. 

## Statusprüfungen in Spring Boot 2
{: #spring-health-boot2}

Der Actuator in Spring Boot 2 definiert den Statusendpunkt (`/actuator/health`). Wenn alles in Ordnung ist, werden die Nutzdaten `{"status": "UP"}` zurückgegeben. Dieser Endpunkt ist standardmäßig aktiviert und erfordert keinen Anwendungscode.

Beachten Sie das folgende Beispiel für eine nicht authentifizierte erfolgreiche Statusprüfung mithilfe des Actuator von Spring Boot 2: 
<!-- Spring Boot 2 test project: https://github.com/IBM/spring-alarm-application -->

```
$ curl -i localhost:8080/actuator/health
HTTP/1.1 200
Content-Type: application/vnd.spring-boot.actuator.v2+json;charset=UTF-8
Transfer-Encoding: chunked
Date: Fri, 07 Dec 2018 23:09:09 GMT

{"status":"UP"}
```
{: screen}

Wie dargestellt, gibt der Statusendpunkt standardmäßig einen einfachen Status des Typs UP oder DOWN zurück. Für automatisierte Statusprüfungen sind diese einfachen Nutzdaten in der Regel ausreichend. Detaillierte Statusinformationen können aktiviert werden, indem Sie die folgende Eigenschaft in `application.properties` festlegen:

```properties
management.endpoint.health.show-details=always
```
{: codeblock}

```
$ curl -i localhost:8080/actuator/health
HTTP/1.1 200
Content-Type: application/vnd.spring-boot.actuator.v2+json;charset=UTF-8
Transfer-Encoding: chunked
Date: Fri, 07 Dec 2018 23:09:09 GMT

{"status":"UP","details":{"diskSpace":{"status":"UP","details":{"total":500068036608,"free":407611420672,"threshold":10485760}},"db":{"status":"UP","details":{"database":"H2","hello":1}}}}
```
{: screen}

Beachten Sie, dass einige Informationen zur H2-Datenbank eingeschlossen sind. Bei diesem Beispiel für Spring Actuator werden automatisch Prüfungen für Unterstützungsservices hinzugefügt. In diesem Fall verwendet die Anwendung JDBC und der H2-Treiber wurde im Klassenpfad erkannt. 

Sie können das Standardverhalten des Statusendpunkts mit Eigenschaften oder Code außer Kraft setzen, wie im [Referenzhandbuch für Spring Boot 2.1.x](https://docs.spring.io/spring-boot/docs/2.1.x/reference/html/production-ready-endpoints.html#production-ready-health){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") beschrieben.

### Liveness-Test in Spring Boot 2
{: #spring-liveness-boot2}

Das Actuator-Framework in Spring Boot 2 ist ein selbstständiges Mini-REST-Framework, das mit angepassten Endpunkten erweitert werden kann. Dies bedeutet, dass Sie einen einfachen Liveness-Endpunkt hinzufügen können, der auf die gleiche Weise verwaltet wird wie der integrierte Statusanzeiger. Ein angepasster Liveness-Endpunkt kann sehr einfach deklariert werden, wie das folgende Beispiel zeigt: 

```java
@Endpoint(id="liveness")
@Component
public class Liveness {
    @ReadOperation
    public String testLiveness() {
            return "{\"status\":\"UP\"}";
    }
}
```
{: codeblock}

Dieser angepasste Endpunkt muss als Webendpunkt zugänglich gemacht werden:

```properties
management.endpoints.web.exposure.include=health,liveness,...
```
{: codeblock}

```
$ curl -i localhost:8080/actuator/liveness
HTTP/1.1 200
Content-Type: application/vnd.spring-boot.actuator.v2+json;charset=UTF-8
Content-Length: 15
Date: Thu, 07 Feb 2019 22:38:27 GMT

{"status":"UP"}
```
{: screen}

### Readiness- und Liveness-Tests von Spring Boot 2 in Kubernetes konfigurieren
{: #spring-probe-config-2}

Fügen Sie die Konfiguration für Readiness- und Liveness-Tests zur Containerdefinition in einem Kubernetes-Bereitstellungsmanifest hinzu (notieren Sie sich den Pfad und die Portwerte):

```yaml
spec:
  containers:
  - name: ...
    image: ...
    readinessProbe:
      httpGet:
        path: /actuator/health
        port: 8080
      initialDelaySeconds: 60
      timeoutSeconds: 5
    livenessProbe:
      httpGet:
        path: /actuator/liveness
        port: 8080
      initialDelaySeconds: 130
      timeoutSeconds: 10
      failureThreshold: 10
```
{: codeblock}

Setzen Sie zur Vermeidung von Neustartzyklen den Wert für `livenessProbe.initialDelaySeconds` so, dass er deutlich nach der Serviceinitialisierung liegt. Sobald der Service betriebsbereit ist, können Sie dann einen kleineren Wert für `readinessProbe.initialDelaySeconds` verwenden, um Anforderungen an diesen weiterzuleiten.

## Statusprüfungen in Spring Boot 1
{: #spring-health-boot1}

Der Actuator in Spring Boot 1 definiert den Statusendpunkt (`/health`). Wenn alles in Ordnung ist, werden die Nutzdaten `{"status": "UP"}` zurückgegeben. Für den Endpunkt ist keine Autorisierung erforderlich, aber wenn Autorisierung konfiguriert und der Aufrufende autorisiert ist, werden zusätzliche Informationen in die Antwort eingeschlossen. 

Beachten Sie das folgende Beispiel für eine nicht authentifizierte erfolgreiche Statusprüfung mithilfe des Actuator von Spring Boot 1: 
```
$ curl -i localhost:8080/health
HTTP/1.1 200
X-Application-Context: application
Content-Type: application/vnd.spring-boot.actuator.v1+json;charset=UTF-8
Content-Length: 15

{"status":"UP"}
```
{: screen}

Beispiel für eine authentifizierte erfolgreiche Statusprüfung:

```
$ curl -i localhost:8080/health
HTTP/1.1 200 OK
X-Application-Context: application
Content-Type: application/vnd.spring-boot.actuator.v1+json;charset=UTF-8
Content-Length: 221

{
    "status" : "UP",
  "diskSpace" : {
        "status" : "UP",
    "total" : 63251804160,
    "free" : 31316164608,
    "threshold" : 10485760
    },
  "db" : {
        "status" : "UP",
    "database" : "H2",
    "hello" : 1
    }
}
```
{: screen}

Sie können das Standardverhalten des Statusendpunkts mit Eigenschaften oder Code außer Kraft setzen, wie im [Referenzhandbuch für Spring Boot 1.5.x](https://docs.spring.io/spring-boot/docs/1.5.9.RELEASE/reference/htmlsingle/#_auto_configured_healthindicators){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") beschrieben.

### Liveness-Test in Spring Boot 1
{: #spring-liveness-boot1}

Für Spring Boot 1 würde ein einfacher HTTP-Endpunkt für Liveness, der immer {"status": "UP"} mit dem Statuscode 200 zurückgibt, in etwa wie im folgenden Snippet aussehen: 

```java
@RestController
public class Liveness {
    private static Map<String, String> alwaysOk = Collections.singletonMap("status", "OK");

    @GetMapping("/liveness")
    public Map<String,String> testLiveness() {
        return alwaysOk;
    }
}
```
{: codeblock}

```
$ curl -i localhost:8080/liveness
HTTP/1.1 200
Content-Type: application/json;charset=UTF-8
Transfer-Encoding: chunked
Date: Thu, 07 Feb 2019 22:43:32 GMT

{"status":"OK"}
```
{: screen}

### Readiness- und Liveness-Tests von Spring Boot 1 in Kubernetes konfigurieren
{: #spring-probe-config-1}

Fügen Sie die Konfiguration für Readiness- und Liveness-Tests zur Containerdefinition in einem Kubernetes-Bereitstellungsmanifest hinzu (notieren Sie sich den Pfad und die Portwerte):

```yaml
spec:
  containers:
  - name: ...
    image: ...
    readinessProbe:
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 60
      timeoutSeconds: 5
    livenessProbe:
      httpGet:
        path: /liveness
        port: 8080
      initialDelaySeconds: 130
      timeoutSeconds: 10
      failureThreshold: 10
```
{: codeblock}

Setzen Sie zur Vermeidung von Neustartzyklen den Wert für `livenessProbe.initialDelaySeconds` so, dass er deutlich nach der Serviceinitialisierung liegt. Sobald der Service betriebsbereit ist, können Sie dann einen kleineren Wert für `readinessProbe.initialDelaySeconds` verwenden, um Anforderungen an diesen weiterzuleiten.
