---

copyright:
  years: 2019
lastupdated: "2019-04-22"

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

La plataforma Spring Platform es un ecosistema de proyectos que tiene como objetivo facilitar la creación de aplicaciones Java. Por lo general, el término "Spring" se refiere a Spring Framework, pero también se utiliza genéricamente para referirse a cualquier proyecto que forma parte de Spring o utilice tecnologías basadas en Spring, incluido Spring Boot.

## Spring Framework
{: #spring-framework}

Spring Framework se presentó en 2002, y desde entonces se ha publicado una versión principal aproximadamente cada 3 años. La infraestructura comprende un amplio conjunto de componentes (denominados Módulos de Spring) que lo cubren todo, desde los puntos finales de REST hasta las abstracciones de base de datos. En el release más reciente, en 2017, Spring Framework 5 introdujo soporte actualizado de JDK y WebFlux. WebFlux es un nuevo módulo para la programación reactiva basado en [Project Reactor](https://projectreactor.io/){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo").

Spring Framework 4.3 es la última rama de funciones para Spring Framework 4.x, con soporte a lo largo de 2020. Las aplicaciones que utilicen versiones más antiguas de Spring deben migrarse a Spring Framework 5.

Esta es la documentación completa de Spring Framework:

* [Guía de referencia de Spring Framework para 5.1.x](https://docs.spring.io/spring/docs/5.1.x/spring-framework-reference/){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo")
* [Guía de referencia de Spring Framework para 4.3.x](https://docs.spring.io/spring/docs/4.3.x/spring-framework-reference/){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo")
* [Actualización a Spring Framework 5.x](https://github.com/spring-projects/spring-framework/wiki/Upgrading-to-Spring-Framework-5.x){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo")

## Spring Boot
{: #spring-boot}

Spring Boot se presentó en 2014 para "mejorar las arquitecturas de aplicación web sin contenedor". Su objetivo era cambiar la configuración de XML a código. Spring Boot proporciona mecanismos para crear aplicaciones basadas en una visión dogmática de las tecnologías que se deben utilizar. Las aplicaciones Spring Boot son aplicaciones Spring y utilizan un modelo de programación exclusivo para ese ecosistema.

Spring Boot enfatiza convención frente a la configuración y utiliza una combinación de anotaciones y descubrimiento de vía de acceso de clases para habilitar funciones adicionales. De forma predeterminada, las aplicaciones Spring Boot se crean en un archivo JAR ejecutable autocontenido que incluye todas las dependencias necesarias (incluido un servidor Tomcat integrado). Las aplicaciones se pueden empaquetar alternativamente como archivos war desplegados en un servidor de aplicaciones.

Spring Boot se encuentra ahora en la versión 2.0, publicada en 2018. El mantenimiento de la rama 1.5.x finalizará en agosto de 2019.

Esta es la documentación completa de Spring Boot:

* [Guía de referencia de Spring Boot para 2.1.x](https://docs.spring.io/spring-boot/docs/2.1.x/reference/){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo")
* [Guía de referencia de Spring Boot para 1.5.x](https://docs.spring.io/spring-boot/docs/1.5.x/reference/){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo")

## Selección de Spring Framework o Spring Boot
{: #spring-framework-or-boot}

Para las nuevas aplicaciones nativas de nube, si opta por utilizar Spring Framework también debe utilizar Spring Boot. Spring Boot simplifica la configuración y el ensamblaje de aplicaciones basadas en Spring y proporciona características, como Spring Actuator, que simplifican la creación de [aplicaciones nativas en la nube](/docs/java?topic=cloud-native-overview#overview).

### Spring Boot Actuator
{: #spring-boot-actuator}

Spring Boot Actuator define una recopilación de puntos finales que son útiles para inspeccionar una app Spring en ejecución. Habilitarlos es tan sencillo como añadir una dependencia única a la aplicación.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```
{: codeblock}

La estructura y el comportamiento de Spring Boot Actuator han cambiado significativamente entre Spring Boot 1 y Spring Boot 2. Boot 1 Actuator se ubicaba junto a la aplicación, con mecanismos de configuración y seguridad independientes. La versión de Boot 2 es más capaz y utiliza un modelo de seguridad que se integra con el resto de la aplicación Spring.

Los cambios de comportamiento son especialmente relevantes si va a migrar una aplicación de Boot 1 a Boot 2. Consulte la sección sobre Actuator en la [Guía de migración de Boot 1 a Boot 2](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.0-Migration-Guide#spring-boot-actuator){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo") para obtener más detalles.
{: tip}

En Boot 2, Actuator actúa como infraestructura miniREST. Todos los puntos finales están bajo un contexto `/actuator`. Se proporcionan puntos finales para realizar comprobaciones de estado, o recopilar métricas o información de aplicación o de otros entornos. Para obtener una lista completa, consulte la [Documentación de Actuator](https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/html/production-ready-features.html#production-ready){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo").

Es fácil crear su propio punto final de Actuator utilizando un `@Component` de Spring:

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

Spring Boot tiene dos controles para configurar qué Actuators están activos en tiempo de ejecución; pueden estar "habilitados" y "expuestos". Para ser accesible, un Actuator debe estar en los dos estados. De forma predeterminada, muchos puntos finales están habilitados, pero sólo se exponen el estado y la información. Además, si se habilita Spring Security para la aplicación, el acceso se debe permitir explícitamente a los puntos finales de Actuator.

Spring Boot 1 Actuator tiene su propia configuración de seguridad, que normalmente se configura en application.properties. En lugar de "habilitado" y "expuesto", puede estar "habilitado" y "sensible". Los puntos finales sensibles requieren autenticación si se suministran a través de HTTP. De forma predeterminada, el punto final /metrics de Spring Boot Actuator está habilitado en Spring Boot 1, pero se considera sensible y, por lo tanto, requiere autorización, o bien debe establecerse como no sensible. Para algunos entornos, configurar Spring Security puede ser la respuesta correcta, pero en Kubernetes, los puntos finales de métricas siguen quedando internos en el clúster. Para obtener más información, consulte los [documentos sobre Spring Boot 1 Actuator](https://docs.spring.io/spring-boot/docs/1.5.2.RELEASE/reference/htmlsingle/#production-ready){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo").
{: note}

## Spring Cloud
{: #spring-cloud}

Spring Cloud es una recopilación de integraciones entre tecnologías de nube de terceros y el modelo de programación Spring. Su objetivo es el de ayudar a los desarrolladores a construir aplicaciones Spring para su despliegue en la nube. La amplitud de la cobertura alcanzada en Spring Cloud se hace posible por su naturaleza modular, ya que cada proyecto dentro de Spring Cloud se centra en una tecnología o conjunto de tecnologías específico.

Los proyectos de Spring Cloud siguen el enfoque general de Spring de favorecer la convención frente a la configuración. La mayoría de las funciones se habilitan añadiendo la dependencia correcta en el momento de la compilación.

Consulte el [sitio oficial del proyecto Spring Cloud](https://spring.io/projects/spring-cloud){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo") para obtener más información sobre los proyectos de Spring Cloud y las tecnologías soportadas.

### Spring Cloud con Kubernetes
{: #spring-cloud-kubernetes}

Spring Cloud Kubernetes es un proyecto de Spring Cloud cuyo objetivo es facilitar diversas tareas para las aplicaciones de Spring que se ejecutan en Kubernetes. Spring Cloud Kubernetes ofrece implementaciones de interfaces comunes de Spring que se correlacionan con los conceptos y servicios de Kubernetes.

Históricamente, Spring ha utilizado bibliotecas de Netflix, como Eureka, como registro de servicio, y Ribbon como equilibrador de carga del lado del cliente. En entornos Kubernetes, el propio Kubernetes desempeña ambos roles, lo que hace que las prestaciones a nivel de aplicación sean redundantes. Spring Cloud Kubernetes adapta las abstracciones de Spring, como `DiscoveryClient`, a los mecanismos de Kubernetes subyacentes.

Se debe tener cuidado si se utiliza el descubrimiento de Ribbon a través de Spring Cloud Kubernetes en un clúster con Istio instalado, en el que se utiliza Istio para afectar al direccionamiento de servicios. El equilibrio de carga del lado del cliente implica que el propio cliente desea seleccionar el servicio de destino, mientras que Istio puede estar intentando direccionar la llamada a otro lugar. Solo uno de ellos puede ganar, lo que lleva a un desacuerdo sobre la forma en que se puede haber dirigido una llamada. Esta combinación debe evitarse, si es posible.
{: note}

Además, Spring Cloud Kubernetes ofrece integración entre ConfigMaps y Secrets de Kubernetes a los beans de configuración `@Autowired` de Spring. Esto incluye estrategias para manejar los cambios dinámicos en
`ConfigMaps` y `Secrets` realizados mientras se ejecuta la aplicación.

Por último, Spring Cloud Kubernetes aumenta el punto final de estado de Spring Boot Actuator predeterminado para incluir información adicional relacionada con el despliegue.

Para obtener más información, consulte la [documentación de Spring Cloud Kubernetes](https://cloud.spring.io/spring-cloud-static/spring-cloud-kubernetes/2.1.0.RC1/single/spring-cloud-kubernetes.html){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo")


<!--
### Spring Cloud Streams
{: #spring-cloud-streams}


:FIXME:
-->
