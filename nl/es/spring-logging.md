---

copyright:
  years: 2018, 2019
lastupdated: "2019-06-10"

keywords: spring logging, spring logger, logback spring, debug spring, json log spring, consoleappender spring, spring boot log

subcollection: java

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}

# Registro en Spring
{: #spring-logging}

Los mensajes de registro son cadenas de información contextual que pertenecen al estado y actividad de un microservicio cuando se crea una entrada de registro. Puede utilizar el registro para diagnosticar cómo y por qué fallan los servicios y para ayudar con las métricas para supervisar el estado de las aplicaciones.

Las entradas de registro se deben escribir directamente en los flujos de salida estándar y de error, lo que hace que puedan visualizarse las entradas de registro utilizando herramientas de línea de mandatos. A continuación, puede utilizar servicios de reenvío de registro que estén configurados a nivel de infraestructura, como Logstash, Filebeat o Fluentd, para gestionar la recopilación de registros.

## Adición de soporte para Logback a las apps Spring
{: #spring-log4j}

[Logback](https://logback.qos.ch/){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo") es el motor de registro predeterminado que utiliza Spring Boot, y se utiliza automáticamente cuando se encuentra en la vía de acceso de clase. La mayoría de los arrancadores de Spring Boot dependen transitivamente del Logback a través de `spring-boot-starter-logging`.

El siguiente ejemplo (basado en un [Spring Boot de muestra](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-samples/spring-boot-sample-logback/src/main/java/sample/logback/SampleLogbackApplication.java)){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo") utiliza la API de registro SLF4J, que Logback implementa, para inicializar un `Logger` y emitir mensajes en varios niveles de registro:

```java
package sample.logging;

import javax.annotation.PostConstruct;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SampleLogbackApplication {
  private static final Logger logger = LoggerFactory.getLogger(SampleLogbackApplication.class);

  @PostConstruct
  public void logSomething() {
    logger.trace("Sample TRACE Message");
    logger.debug("Sample DEBUG Message");
    logger.info("Sample INFO Message");
    logger.warn("Sample WARN Message");
    logger.error("Sample ERROR Message");
  }

  public static void main(String[] args) {
    SpringApplication.run(SampleLogbackApplication.class, args).close();
  }
}
```
{: codeblock}

El nivel de registro predeterminado es `INFO`. Puede especificar distintos niveles de registro para paquetes de Java&trade; específicos en `application.properties`. Por ejemplo, para establecer el nivel de registro predeterminado en `WARN`, el nivel para el paquete `sample.logging` en `DEBUG`, y el nivel para el paquete Spring `org.springframework.web` en `ERROR`, añada lo siguiente en `application.properties`:

```properties
logging.level.root=WARN
logging.level.org.springframework.web=ERROR
logging.level. sample.logging=DEBUG
```
{: codeblock}

De forma predeterminada, Logback produce mensajes con los siguientes elementos:

- Fecha
- Tiempo con precisión de milisegundos
- Nivel de registro: `ERROR`, `WARN`, `INFO`, `DEBUG` o `TRACE`.
- ID de proceso.
- Un separador "`---`" para definir el inicio de los mensajes de registro reales.
- [Nombre de hebra (puede estar truncado para la salida de la consola)]
- Nombre del registrador: El nombre suele ser el nombre de la clase de origen (a menudo abreviado).
- El mensaje de registro.

Los mensajes de registro se parecen al siguiente ejemplo:

```
2018-12-12 09:59:34.751  INFO 11174 --- [           main] sample.logging.SampleLogbackApplication  : Sample INFO Message
2018-12-12 09:59:34.751  WARN 11174 --- [           main] sample.logging.SampleLogbackApplication  : Sample WARN Message
2018-12-12 09:59:34.752 ERROR 11174 --- [           main] sample.logging.SampleLogbackApplication  : Sample ERROR Message
```
{: screen}

### Creación de registros JSON
{: #spring-json-logs}

Habilitar el registro JSON en apps Spring utilizando `logstash-logback-encoder`.

Añadir el codificador de logback como una dependencia:

```xml
<dependency>
  <groupId>net.logstash.logback</groupId>
  <artifactId>logstash-logback-encoder</artifactId>
  <version>4.7</version>
</dependency>
```
{: codeblock}

Configurar logback para usar el nuevo codificador con un `ConsoleAppender` en `logback.xml`:

```xml
<configuration>
    <appender name="consoleAppender" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="net.logstash.logback.encoder.LogstashEncoder"/>
    </appender>
    <logger name="jsonLogger" additivity="false" level="DEBUG">
        <appender-ref ref="consoleAppender"/>
    </logger>
    <root level="INFO">
        <appender-ref ref="consoleAppender"/>
    </root>
</configuration>
```
{: codeblock}

Con esta configuración, puede ver los registros formateados en JSON:

```
{"@timestamp":"2018-10-11T23:48:57.215+00:00","@version":1,"message":"Sample TRACE Message","logger_name":"com.example.demo.LoggingExample","thread_name":"http-nio-8080-exec-1","level":"TRACE","level_value":5000}
{"@timestamp":"2018-10-11T23:48:57.216+00:00","@version":1,"message":"Sample DEBUG Message","logger_name":"com.example.demo.LoggingExample","thread_name":"http-nio-8080-exec-1","level":"DEBUG","level_value":10000}
{"@timestamp":"2018-10-11T23:48:57.216+00:00","@version":1,"message":"Sample INFO Message","logger_name":"com.example.demo.LoggingExample","thread_name":"http-nio-8080-exec-1","level":"INFO","level_value":20000}
{"@timestamp":"2018-10-11T23:48:57.217+00:00","@version":1,"message":"Sample WARN Message","logger_name":"com.example.demo.LoggingExample","thread_name":"http-nio-8080-exec-1","level":"WARN","level_value":30000}
{"@timestamp":"2018-10-11T23:48:57.217+00:00","@version":1,"message":"Sample ERROR Message","logger_name":"com.example.demo.LoggingExample","thread_name":"http-nio-8080-exec-1","level":"ERROR","level_value":40000}
```
{: screen}

[`logstash-logback-encoder`](https://github.com/logstash/logstash-logback-encoder){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo") puede personalizarse aún más para modificar la forma en que se capturan y envían los registros de JSON.

Utilizar `jq` para filtrar los registros JSON para una mejor lectura. Por ejemplo:

```
$ docker logs --tail 1 -f <container-id> | jq .
{
  "@timestamp": "2018-10-11T23:48:57.215+00:00",
  "@version": 1,
  "message": "Sample TRACE Message",
  "logger_name": "com.example.demo.LoggingExample",
  "thread_name": "http-nio-8080-exec-1",
  "level": "TRACE",
  "level_value": 5000
}

$ docker logs --tail 5 -f <container-id> | jq .message
"Sample TRACE Message"
"Sample DEBUG Message"
"Sample INFO Message"
"Sample WARN Message"
"Sample ERROR Message"
```
{: screen}

## Pasos siguientes
{: #spring-logging-next-steps notoc}

Para obtener más información sobre la personalización de los mensajes de registro con agregadores, niveles de registro y detalles de configuración, consulte la [referencia de Spring Boot para el registro](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-logging.html){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo").

Obtenga más información sobre la visualización de registros en cada uno de los siguientes entornos de despliegue:

* [Registros de Kubernetes](https://kubernetes.io/docs/concepts/cluster-administration/logging/){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo")
* [Registros y supervisión de {{site.data.keyword.openwhisk}}](/docs/openwhisk?topic=cloud-functions-logs)
* [{{site.data.keyword.cloud_notm}} Log Analysis](/docs/services/CloudLogAnalysis?topic=cloudloganalysis-log_analysis_ov#log_analysis_ov)
* [Pila de ELK de {{site.data.keyword.cloud_notm}} Private](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_2.1.0.2/manage_metrics/logging_elk.html){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo")
