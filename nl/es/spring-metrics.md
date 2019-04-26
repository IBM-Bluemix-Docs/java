---

copyright:
  years: 2019
lastupdated: "2019-04-09"

keywords: spring metrics, configure metrics spring, micrometer spring, micrometer, spring boot 2, spring actuator, prometheus spring

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

# Métricas con Spring
{: #spring-metrics}

A partir de Spring Framework 5, las métricas de Spring las maneja Micrometer. Micrometer es una infraestructura que se describe a sí misma como "SLF4J para métricas". Igual que SLF4J actúa como una API neutral de proveedor para el registro que puede conectarse a varios programas de fondo de registro diferentes, Micrometer proporciona una API neutral de proveedor con la que instrumentar y medir el código, y luego proporcionar esas métricas en adelante a una variedad de agregadores de métricas, como Prometheus, DataDog o Influx/Telegraph. 

La infraestructura de Micrometer permite que Spring se integre en una gran variedad de arquitecturas nativas en la nube. La adición de soporte para Prometheus o Statsd sólo es cuestión de modificar de dependencias, y si el recopilador de métricas está basado en push, proporcionar información de destino en `application.properties`. Spring y Micrometer determinan qué hacer en tiempo de ejecución en función de las dependencias que encuentran en la vía de acceso de clases de la aplicación.

## Habilitación de métricas
{: #spring-metrics-enabling}

Spring Boot Actuator suministra métricas básicas listas para utilizar, y necesita la dependencia `spring-boot-starter-actuator` en el archivo `pom.xml`:  

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-actuator</artifactId>
</dependency>
```
{: codeblock}

Si aún utiliza Spring Boot 1.5.x, las métricas (y los Actuators) funcionan de forma diferente. Micrometer se ha convertido en el estándar para las métricas de Spring en Spring Boot 2.0, pero hay disponibles puertos de respaldo para Boot 1.5.x, lo que permite prácticas coherentes para crear y recopilar métricas en aplicaciones Spring Boot nativas de la nube. Lo que es más importante, Micrometer da soporte a varios sistemas de supervisión, incluido Prometheus, que lo convierten en una opción más adecuada que el sistema de métricas de Boot 1.5.x para los despliegues en la nube.
{: note}

## Métricas de Micrometer
{: #spring-metrics-micrometer}

Micrometer abstrae el concepto de destino para las métricas mediante su interfaz `MeterRegistry`. `MeterRegistry` proporciona el modo de que la aplicación obtenga contadores personalizados, medidores, temporizadores, etc. Si se necesitan varios destinos, Micrometer suministra una implementación de `CompositeMeterRegistry` que permite aislar la aplicación de los destinos reales configurados para las métricas.

En cualquiera de las versiones de Spring Boot, cuando se habilita un punto final de métrica basado en Micrometer, Spring Boot crea automáticamente un `CompositeMeterRegistry` y lo pone a disposición de la aplicación como un `Bean`. El registro se llena automáticamente con las implementaciones de `MeterRegistry` soportadas que se encuentran en la vía de acceso de clases. Para dar soporte a varios sistemas de supervisión, o cambiar entre ellos, basta con modificar las dependencias en el archivo pom.xml. 

Spring permite la personalización del bean `MeterRegistry` mediante `MeterRegistryCustomizer`, que también es un Bean. Cuando Spring identifica la presencia de un bean de este tipo de Bean, se le da la oportunidad de configurar el `MeterRegistry` global antes de que se informe de cualquier métrica. A menudo se utiliza este método para añadir etiquetas comunes a `MeterRegistry`, como DataCenter o EnvironmentType.

Cuando se nombran métricas, Spring y Micrometer recomiendan seguir un convenio de denominación que divide palabras con `.` en lugar de utilizar escritura camelCase o `_`, u otros métodos. Esto se debe a que Micrometer transmitirá esas métricas a los destinos configurados y realizará las conversiones de nombres según sea necesario para cumplir los requisitos de los destinos.

## Configuración de métricas con Spring Boot
{: #spring-metrics-configuration}

En las secciones siguientes se describe cómo habilitar las métricas de Spring Boot Actuator utilizando Micrometer para recopilar métricas y proporcionar un punto final para Prometheus para cada release, empezando por Spring Boot 2 como el más actual.

### Configuración de métricas en Spring Boot 2
{: #spring-metrics-boot2}

El punto final de métricas de Spring Boot 2 Actuator debe estar habilitado y expuesto para el acceso web. De forma predeterminada, el punto final está habilitado, pero no expuesto (consulte [la documentación de punto final de Spring](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-endpoints.html){: new_window}![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo") para obtener más información).  

Añada lo siguiente a `application.properties` para exponer el punto final de métricas:

```properties
management.endpoints.web.exposure.include=health,metrics
```
{: codeblock}

Verifique que el punto final de métricas se ha habilitado comprobando el punto final `/actuator`. La salida será similar a la siguiente, al ejecutar la aplicación localmente (utilizando `jq` para mostrar la respuesta json):

```
$ curl -s localhost:8080/actuator | jq .
{
  "_links": {
    "self": {
      "href": "http://localhost:8080/actuator",
      "templated": false
    },
    ...
    "metrics": {
      "href": "http://localhost:8080/actuator/metrics",
      "templated": false
    },
    "metrics-requiredMetricName": {
      "href": "http://localhost:8080/actuator/metrics/{requiredMetricName}",
      "templated": true
    },
    ...
  }
}
```
{: screen}

Al visitar el punto final `/actuator/metrics` se emitirá una lista con formato json de métricas disponibles que será similar a la siguiente:

```
$ curl -s localhost:8080/actuator/metrics | jq .
{
  "names": [
    "jvm.memory.max",
    "jvm.threads.states",
    "process.files.max",
    "jvm.gc.memory.promoted",
    ...
  ]
}
```
{: screen}

Se necesita una solicitud adicional para ver el valor real de una métrica individual (tal como se puede inferir de la lista de URL de actuator). Para recuperar los estados de hebra jvm, por ejemplo:

```
$ curl -s localhost:8080/actuator/metrics/jvm.threads.states | jq .
{
  "name": "jvm.threads.states",
  "description": "The current number of threads having TERMINATED state",
  "baseUnit": "threads",
  "measurements": [
    {
    "statistic": "VALUE",
    "value": 24
    }
  ],
  "availableTags": [
    {
      "tag": "state",
      "values": [
        "timed-waiting",
        "new",
        "runnable",
        "blocked",
        "waiting",
        "terminated"
      ]
    }
  ]
}
```
{: screen}

Para obtener más información acerca de cómo personalizar el comportamiento del actuator de métricas, consulte la [documentación oficial](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-metrics.html){: new_window}![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo").

### Habilitación del soporte de Prometheus en Spring Boot 2
{: #spring-prometheus-boot2}

Prometheus requiere que la aplicación aloje un punto final que el servidor Prometheus lea para obtener las métricas. El alojamiento de este punto final en Spring se basa en que la aplicación ya tiene la capacidad de suministrar los datos, lo que significa que el punto final de Prometheus sólo soporta aplicaciones Spring Boot que utilizan Spring MVC, Spring WebFlux o Jersey.

Para habilitar el punto final de Prometheus, añada la dependencia adicional siguiente en `pom.xml`:

```xml
<dependency>
  <groupId>io.micrometer</groupId>
  <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```
{: codeblock}

Spring notará la dependencia y configurará automáticamente PrometheusMeterRegistry como uno de los registros configurados en el bean MeterRegistry global. Sin embargo, al igual que con el punto final de métricas, el punto final de Prometheus debe estar habilitado y expuesto para el acceso web añadiendo lo siguiente a application.properties:

```properties
management.endpoints.web.exposure.include=health,metrics,prometheus
```
{: codeblock}

Verifique que el punto final de Prometheus se ha habilitado comprobando el punto final `/actuator`. La salida será similar a la siguiente, al ejecutar la aplicación localmente:

```
$ curl -s localhost:8080/actuator | jq .
{
  "_links": {
    "self": {
      "href": "http://localhost:8080/actuator",
      "templated": false
    },
    ...
    "prometheus": {
      "href": "http://localhost:8080/actuator/prometheus",
      "templated": false
    },
    ...
  }
}
```
{: screen}

El punto final `/actuator/prometheus` emitirá todas las métricas configuradas en el formato de recopilación de Prometheus:

```
$ curl -s localhost:8080/actuator/prometheus
# HELP system_cpu_usage The "recent cpu usage" for the whole system
# TYPE system_cpu_usage gauge
system_cpu_usage 0.046231032221805926
...
```
{: screen}

Añada las anotaciones siguientes a la definición de servicio para permitir la recopilación de Prometheus en el punto final de métricas:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: ...
  annotations:
    prometheus.io/scrape: 'true'
    prometheus.io/scheme: http
    prometheus.io/path: /actuator/prometheus
    prometheus.io/port: "8080"
  ...
```
{: codeblock}

### Configuración de métricas en Spring Boot 1
{: #spring-metrics-boot1}

El punto final `/metrics` de Spring Boot Actuator está habilitado de forma predeterminada en Spring Boot 1, pero se considera “sensible” y requiere autorización. Para algunos entornos, la protección del punto final de métricas puede ser la respuesta correcta, pero la autorización de cada solicitud en el punto final de métricas introduce demasiada latencia para la comprobación automatizada con Kubernetes. Inhabilite el atributo sensible del punto final de métricas para eliminar el requisito de autorización añadiendo lo siguiente a application.properties:

```properties
endpoints.metrics.sensitive=false
```
{: codeblock}

Esto habilita el soporte de métricas de Spring Boot 1 predeterminado, que produce una salida que tiene un aspecto similar al siguiente:

```
$ curl -s localhost:8080/metrics | jq .
{
  "names": [
    "jvm.memory.max",
    "jvm.threads.states",
    "process.files.max",
    "jvm.gc.memory.promoted",
    ...
  ]
}
```
{: screen}

Para obtener más información acerca de cómo personalizar el comportamiento del actuator de métricas de Spring Boot 1, consulte la [documentación oficial](https://docs.spring.io/spring-boot/docs/1.5.x/reference/html/production-ready-metrics.html){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo").

### Habilitación del soporte de Prometheus en Spring Boot 1
{: #spring-prometheus-boot1}

El soporte de Prometheus no se basa en las métricas de Spring Boot 1 Actuator, pero podemos añadir soporte para él a través de Micrometer.

Tal y como se menciona en la [descripción general](#spring-metrics), Micrometer se trasladó a Spring Boot 1 porque tiene un conjunto más rico de primitivos de medidor y un mejor soporte para sistemas de supervisión como Prometheus y StatsD. El uso de Micrometer con Spring Boot 1 es bastante directo: requiere una biblioteca de adaptadores específica, y al menos un registro. Añada lo siguiente a `pom.xml` para incluir dependencias para la biblioteca de adaptadores y el registro de Prometheus:

```xml
<properties>
  ...
  <micrometer.version>1.1.1</micrometer.version>
</properties>
<dependencies>
  <dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-spring-legacy</artifactId>
    <version>${micrometer.version}</version>
  </dependency>
  <dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
    <version>${micrometer.version}</version>
  </dependency>
</dependencies>
```
{: codeblock}

Estas dependencias crearán y expondrán un punto final de métricas `/prometheus`. Al igual que con el punto final `/metrics` predeterminado, el punto final `/prometheus` se considera sensible de forma predeterminada. Para eliminar el requisito de autorización, añada lo siguiente a application.properties:

```properties
endpoints.prometheus.sensitive=false
```
{: codeblock}

La salida producida por el punto final /prometheus es similar a la de Spring Boot 2:

```
$ curl -s localhost:8080/prometheus
# HELP jvm_memory_used_bytes The amount of used memory
# TYPE jvm_memory_used_bytes gauge
jvm_memory_used_bytes{area="heap",id="PS Survivor Space",} 2.2594152E7
...
```
{: screen}

Añada las anotaciones siguientes a la definición de servicio para permitir la recopilación de Prometheus en el punto final de métricas:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: …
  annotations:
    prometheus.io/scrape: 'true'
    prometheus.io/scheme: http
    prometheus.io/path: /prometheus
    prometheus.io/port: "8080"
  ...
```
{: codeblock}

Para obtener más información sobre el uso de Micrometer con Spring Boot 1, consulte [https://micrometer.io/docs/ref/spring/1.5](https://micrometer.io/docs/ref/spring/1.5){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo").

## Invocaciones de temporización con Micrometer
{: #spring-metrics-timing}

Spring MVC, WebFlux y Jersey Server proporcionan configuración automática para Spring que recopila automáticamente información de temporización para todas las solicitudes cuando la propiedad `management.metrics.web.server.auto-time-requests` se establece en `true`. Este es el valor predeterminado.

```properties
management.metrics.web.server.auto-time-requests=true
```
{: codeblock}

Micrometer también da soporte a la anotación `@Timed`, que se puede utilizar para recopilar información de temporización para llamadas de método soportadas. Cabe señalar que esta anotación no se puede utilizar para métodos arbitrarios de tiempo, ya que requiere el soporte de contenedor para recopilar los datos. Las anotaciones `@Timed` están soportadas para las operaciones con el respaldo de un contenedor de servlet, por ejemplo, los métodos de servicio de Spring MVC, Webflux o Jersey.

Para ser más selectivo sobre qué información de temporización se recopila, establezca la propiedad `management.metrics.web.server.auto-time-requests` en `false` y utilice la anotación `@Timed` en los métodos de servicio individuales o controladores:

```java
@Timed
@GetMapping("/myendpoint")
public String getSomething() { ... }
```
{: codeblock}

Si `management.metrics.web.server.auto-time-requests` es `true`, se puede utilizar la anotación `@Timed` para añadir etiquetas adicionales a una métrica de informes determinada, por ejemplo:

```java
@Timed(extraTags = { "mytagname", "mytagvalue"})
@GetMapping("/myendpoint")
public String getSomething() { ... }
```
{: codeblock}

## Métricas personalizadas con Spring y Micrometer
{: #spring-metrics-custom}

Para realizar las medidas específicas del dominio de aplicaciones, tendrá que crear sus propias métricas personalizadas. Micrometer define algunos tipos básicos de medidor (`Meter`):

* Un valor `Counter` realiza un seguimiento de un valor que sólo puede aumentar.
* Un valor `Gauge` mide y devuelve el valor observado cuando se publica el medidor (o se consulta).
* Un valor `Timer` realiza un seguimiento del número de veces que se ha producido un suceso y el tiempo transcurrido acumulado para dicho suceso. 
* Un valor `DistributionSummary` es similar a un valor `Timer`, también realiza un seguimiento de una distribución de sucesos, pero no mide el tiempo.

Los nuevos medidores se crean utilizando métodos de ayuda en el valor `MeterRegistry`, o bien en el compilador fluido del medidor. 

Al trabajar en el código de aplicación, cree el `Meter` en el constructor (pasando `MeterRegistry` como un parámetro) o en un método `@PostConstruct` después de que se haya inyectado `MeterRegistry`.

Por ejemplo, para utilizar un valor `Counter` dentro de `RestController`, puede indicar algo como lo siguiente:

```java
@RestController
public class MyController {
    
    private final Counter myCounter;
    
    public MyController(MeterRegistry meterRegistry) {
    
        // Crear el contador utilizando el método de ayudante en el compilador
        myCounter = meterRegistry.counter("my.counter", "mytagname", "mytagvalue");
        
    }
```
{: codeblock}

A continuación, dentro de los métodos de servicio, puede actualizar el valor de medidor como corresponda:

```java
@GetMapping("/")
public String getSomething() {
    myCounter.increment();
    ...
}
```
{: codeblock}


Para obtener más información sobre la creación de métricas personalizadas en Spring:

* [Métricas de Spring Boot](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-metrics.html){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo")
* [Conceptos de Micrometer](https://micrometer.io/docs/concepts){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo")
