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

# Comprobaciones de estado con Spring
{: #spring-healthcheck}

El punto final de estado proporcionado por Spring Boot Actuator está vinculado al ciclo de vida de la aplicación y no estará disponible hasta que se inicie la aplicación. Del mismo modo, se inhabilita tan pronto como se detiene la aplicación (lo que ocurre antes de que concluya el proceso). Spring Boot Actuator añade automáticamente indicadores de estado para las tecnologías detectadas en la vía de acceso de clase. Por ejemplo, si la aplicación utiliza JDBC, el punto final de estado incluirá algunas pruebas básicas para asegurarse de que se puede acceder al almacén de datos de reserva. El punto final de estado definido por el Actuator es más adecuado para su uso como prueba de preparación por esta razón.

Habilite Spring Boot Actuator añadiendo la dependencia `spring-boot-actuator` al archivo `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-actuator</artifactId>
</dependency>
```
{: codeblock}

Hay algunas diferencias de comportamiento con Spring Boot Actuator entre las versiones. Las dos secciones siguientes exploran cómo crear comprobaciones de actividad y preparación en ambas versiones de Spring Boot, empezando por la 2 como la más actual.

## Comprobaciones de estado en Spring Boot 2
{: #spring-health-boot2}

Spring Boot 2 Actuator define un punto final `/actuator/health endpoint`, que devuelve una carga útil `{"status": "UP"}` cuando todo está bien. Este punto final está habilitado de forma predeterminada, y no requiere ningún código de aplicación.

Consulte el ejemplo siguiente de comprobación de estado correcta sin autenticación que utiliza Spring Boot 2 Actuator:
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

Tal como se muestra, el punto final de estado devuelve de forma predeterminada un estado simple UP o DOWN. Para la comprobación de estado automatizada, esta simple carga útil suele ser suficiente. La información de estado detallada se puede habilitar estableciendo la propiedad siguiente en `application.properties`:

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

Observe la inclusión de cierta información de base de datos H2. Este ejemplo de Spring Actuator añade automáticamente comprobaciones para los servicios de reserva. En este caso, la aplicación utiliza JDBC, y el controlador de H2 se ha descubierto en la vía de acceso de clases.

Puede alterar temporalmente el comportamiento predeterminado del punto final health con propiedades o código, tal como se describe en la [Guía de referencia de Spring Boot para 2.1.x](https://docs.spring.io/spring-boot/docs/2.1.x/reference/html/production-ready-endpoints.html#production-ready-health){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo").

### Prueba de actividad en Spring Boot 2
{: #spring-liveness-boot2}

La infraestructura de Actuator en Spring Boot 2 es su propia infraestructura miniREST que se puede ampliar con puntos finales personalizados. Esto significa que puede añadir un punto final de actividad simple que se gestiona de la misma forma que el indicador de estado incorporado. Un punto final de actividad personalizado puede declararse de forma de forma similar al ejemplo siguiente:

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

Este punto final personalizado debe estar expuesto como un punto final web:

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

### Configuración de pruebas de actividad y preparación de Spring Boot 2 en Kubernetes
{: #spring-probe-config-2}

Añada la configuración de prueba de preparación y actividad a la definición de contenedor en un manifiesto de despliegue de Kubernetes (observe los valores de vía de acceso y puerto):

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

Para evitar los ciclos de reinicio, establezca `livenessProbe.initialDelaySeconds` para que sea seguro más tiempo del que necesita el servicio para inicializarse. Puede utilizar un valor más corto para `readinessProbe.initialDelaySeconds`, para direccionar solicitudes al servicio en cuanto esté listo.

## Comprobaciones de estado en Spring Boot 1
{: #spring-health-boot1}

Spring Boot 1 Actuator define un punto final `/health`, que devuelve una carga útil `{"status": "UP"}` cuando todo está bien. El punto final no requiere autorización, pero si la autorización está configurada y el emisor de la llamada está autorizado, se incluye información adicional en la respuesta.

Consulte el ejemplo siguiente de comprobación de estado correcta sin autenticación que utiliza Spring Boot 1 Actuator:
```
$ curl -i localhost:8080/health
HTTP/1.1 200
X-Application-Context: application
Content-Type: application/vnd.spring-boot.actuator.v1+json;charset=UTF-8
Content-Length: 15

{"status":"UP"}
```
{: screen}

Un ejemplo de una comprobación de estado correcta con autenticación:

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

Puede alterar temporalmente el comportamiento predeterminado del punto final health con propiedades o código, tal como se describe en la [Referencia de Spring Boot v1.5.x](https://docs.spring.io/spring-boot/docs/1.5.9.RELEASE/reference/htmlsingle/#_auto_configured_healthindicators){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo").

### Prueba de actividad en Spring Boot 1
{: #spring-liveness-boot1}

Para Spring Boot 1, un punto final HTTP simple para la actividad que siempre devuelva {"status": "UP"} con el código de estado 200 sería algo similar al fragmento de código siguiente:

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

### Configuración de pruebas de actividad y preparación de Spring Boot 1 en Kubernetes
{: #spring-probe-config-1}

Añada la configuración de prueba de preparación y actividad a la definición de contenedor en un manifiesto de despliegue de Kubernetes (observe los valores de vía de acceso y puerto):

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

Para evitar los ciclos de reinicio, establezca `livenessProbe.initialDelaySeconds` para que sea seguro más tiempo del que necesita el servicio para inicializarse. Puede utilizar un valor más corto para `readinessProbe.initialDelaySeconds`, para direccionar solicitudes al servicio en cuanto esté listo.
