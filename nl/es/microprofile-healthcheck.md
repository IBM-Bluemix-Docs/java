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

# Comprobaciones de estado con JAX-RS
{: #jaxrs-healthcheck}

Tal como se ha descrito en la sección anterior, Kubernetes proporciona varios métodos para integrar pruebas de estado que incluyen la ejecución de un mandato y la comprobación de red a través de puntos finales TCP o HTTP. Aunque puede implementar cualquiera de estas pruebas en el lenguaje Java&trade;, aquí se va a detallar la implementación de JAX-RS de pruebas HTTP y MicroProfile Health.

## Definición de un punto final JAX-RS de preparación
{: #mp-readiness}

Los puntos finales de preparación mínimos tienen el aspecto del ejemplo siguiente:

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

Un punto final correspondiente a una aplicación desplegada en WebSphere Liberty consigue un determinado nivel de comportamiento de preparación sin esfuerzo adicional. Liberty mostrará solicitudes fallidas al punto final de estado con una respuesta 503 adecuada hasta que la aplicación esté en ejecución. Esta implementación básica se puede ampliar para incluir las prestaciones requeridas. Por ejemplo, considere evaluar los recursos de CDI `@ApplicationScoped` para asegurarse de que se ha completado el proceso de inyección de dependencias. Una vez que la prueba se ha implementado, se puede habilitar configurando el tipo de prueba HTTP como se indica en la sección anterior.

Recuerde siempre el papel y el propósito de la tolerancia a errores: se espera que los microservicios manejen las anomalías y las interrupciones en las comunicaciones con los servicios en sentido descendente. Debe incluir comprobaciones para las prestaciones que no tienen ninguna reserva factible dentro de una comprobación de preparación.

## Definición de un punto final JAX-RS de actividad
{: #jaxrs-liveness}

Una prueba de actividad, por el contrario, tiene en cuenta lo que comprueba, ya que un error provocará una terminación inmediata del proceso. Un punto final de actividad simple se puede parecer al del siguiente ejemplo:

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

La implementación de la prueba de actividad puede ampliarse para comprobar las condiciones locales del proceso, que indican un estado de proceso irrecuperable. Sin embargo, el reto que plantean estas comprobaciones suele recaer en el hecho de que, si el proceso se puede realizar, es probable que el servidor funciones con normalidad para procesar otras solicitudes. Por esta razón, se puede aplicar inmediatamente el mantra "simplificar" a la implementación de la comprobación de estado. Una vez que se han codificado los orígenes de preparación para habilitar el punto final de la prueba, el punto final de actividad HTTP se puede habilitar en los orígenes de orquestación de contenedor, tal como se explica en el último capítulo.

Para evitar ciclos de reinicio, el atributo `initialDelaySeconds` para la comprobación de preparación debe ser mayor que la hora de inicio del servidor esperada más larga. En el caso de un servidor de aplicaciones Java&trade;, que normalmente tarda 30 segundos en iniciarse, elija un valor mayor, como por ejemplo 60 segundos.

## Utilización de la API de MicroProfile Health
{: #mp-health-api}

La [API de MicroProfile Health](https://www.ibm.com/support/knowledgecenter/en/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/twlp_microprofile_healthcheck.html){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo") (mpHealth) define una infraestructura para simplificar la implementación de las comprobaciones de estado. La versión 1.0 de la API
`mpHealth` define un solo punto final. La siguiente versión proporcionará dos, uno para la actividad y otro para la preparación.

Para utilizar la API de MicroProfile Health con Liberty, añada la característica `mpHealth` a `server.xml`:

```xml
<featureManager>
   <feature>mpHealth-1.0</feature>
</featureManager>
```
{: codeblock}

Puede comprobar el estado de la aplicación con un navegador accediendo al punto final `/health`. El código devuelve una carga útil `{"outcome": "UP"}` si todo es correcto. El código siguiente le muestra cómo utilizar las API de MicroProfile Health para indicar si el pod funciona con normalidad. El método `call` permite a los desarrolladores proporcionar un algoritmo de comprobación para indicar el estado de salud de un pod. Algo como quedarse casi sin memoria es un buen indicador del buen estado de un pod. Comprobar la conexión de base de datos en sentido descendente puede ser una buena comprobación de la preparación del pod. Si no hay ninguna comprobación específica, puede utilizar lo siguiente para indicar si un pod está activo o preparado.

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

A continuación, tendrá que especificar `livenessProbe` y `readinessProbe` para Kubernetes, tal como se muestra en el siguiente ejemplo.
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

Como se ha mencionado anteriormente, puede utilizar diferentes lógicas para indicar la preparación y la actividad. Puede utilizar MicroProfile Health para indicar la comprobación de preparación y, a continuación, crear un punto final de JAX-RS para la comprobación de preparación, o viceversa. La próxima versión de MicroProfile Health proporcionará dos puntos finales, uno para la actividad y otro para la preparación.

OpenLiberty tiene una guía práctica que proporciona un ejemplo de cómo añadir comprobaciones personalizadas al punto final de MicroProfile Health: https://openliberty.io/guides/microprofile-health.html

Puede alterar temporalmente el comportamiento predeterminado del punto final de estado de MicroProfile, tal como se describe en [Realización de comprobaciones de estado de MicroProfile](https://www.ibm.com/support/knowledgecenter/en/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/twlp_microprofile_healthcheck.html){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo").
