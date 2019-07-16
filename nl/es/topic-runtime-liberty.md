---

copyright:
  years: 2019
lastupdated: "2019-06-10"

keywords: open liberty java, websphere liberty java, jakarta, webpshere docker, liberty docker, liberty docker images, installutility, microprofile java, dual layer docker, develop microservices

subcollection: java

---

{:new_window: target="_blank"}
{:external: target="_blank" .external}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}

# Open Liberty y WebSphere Liberty
{: #liberty}

[Open Liberty](https://openliberty.io/){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo") es un entorno de ejecución Java&trade; ligero de código abierto construido mediante *características* modulares. [WebSphere Liberty](https://developer.ibm.com/wasdev/){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo") es una versión comercial de Open Liberty. 

Las características de Liberty dan soporte a las siguientes infraestructuras de aplicación:

* [MicroProfile](https://microprofile.io/){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo"), un proyecto de código abierto que define nuevos estándares y API para acelerar y simplificar la creación de microservicios.
* [Jakarta EE](https://jakarta.ee){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo") y [Java&trade; EE](https://www.oracle.com/technetwork/java/javaee/overview/index.html){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo"), incluidas las características de las especificaciones individuales, como JNDI o JAX-RS.
* [Spring Framework y Spring Boot](https://www.ibm.com/support/knowledgecenter/en/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/twlp_dep_springboot.html){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo"), incluidos los mecanismos para hacer contenedores compactos a partir de los fat .jar de Spring Boot.

Hay un amplio conjunto de guías prácticas de desarrollo para crear microservicios y apps nativas de nube con Liberty en [https://openliberty.io/guides/](https://openliberty.io/guides/){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo").

## Optimizado para Docker
{: #liberty-optimized}

Cuando los sistemas automatizados como Kubernetes envían imágenes de contenedores por push, el tamaño de la imagen comienza a ser importante. Las capas de las imágenes de Docker se almacenan en la memoria caché para ayudar con este problema. Dada su arquitectura modular, Liberty proporciona un conducto de empaquetado eficiente para contenedores de Docker, lo que hace que sea un gran ajuste operativo para entornos de nube. Se puede utilizar una imagen base para dar soporte a muchas cargas de trabajo, lo que permite que el espacio de los recursos de los contenedores en ejecución pueda variar en función de los requisitos del servicio.

Liberty proporciona herramientas y soporte optimizado para convertir los fat .jar de Spring Boot en contenedores de Docker optimizados y compactos que aprovechan las capas de imagen de Docker en memoria caché para mejorar el ciclo y los tiempos de publicación. Al enviar por push a una capa separada las dependencias de biblioteca que cambian con poca frecuencia y mantener sólo las clases de app en la capa superior, las reconstrucciones iterativas y los redespliegues serán mucho más rápidos. Consulte más información en [Creación de imágenes de Docker de capa dual para apps de Spring Boot](https://openliberty.io/blog/2018/07/02/creating-dual-layer-docker-images-for-spring-boot-apps.html){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo").

## Imágenes de Liberty Docker
{: #liberty-images}

Tanto Open Liberty como WebSphere Liberty proporcionan diversas imágenes base que se pueden utilizar como puntos de partida para los microservicios basados en Java. Hay imágenes que utilizan pequeños espacios o máquinas virtuales OpenJ9 con diferentes características preseleccionadas. Por ejemplo:

1. `open-liberty:microProfile2` o `websphere-liberty:microProfile2`
2. `open-liberty:javaee8` o `websphere-liberty:javaee8`
3. `open-liberty:springBoot2` o `websphere-liberty:springBoot2`

Consulte [`websphere-liberty`](https://hub.docker.com/_/websphere-liberty/){: external} u [`open-liberty`](https://hub.docker.com/_/open-liberty/){: external} en Docker Hub para ver la lista de imágenes base más reciente.

### Open Liberty y Docker
{: #openliberty-docker}

Elija la imagen más apropiada para su app en la [lista de imágenes de Docker Hub](https://hub.docker.com/_/open-liberty/){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo") (por ejemplo, `open-liberty:microProfile`), y cree un Dockerfile para el servicio que amplíe la opción `FROM`. Añada los mandatos para copiar la app y la configuración de servidor en la nueva imagen. Por ejemplo:

```docker
FROM open-liberty:microProfile\
COPY server.xml /config/\
ADD Sample1.war /config/dropins/
```
{: codeblock}

Para obtener más ejemplos y código funcional, consulte las guías siguientes de Open Liberty:

* [Uso de contenedores de Docker para desarrollar microservicios](https://openliberty.io/guides/docker.html){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo")
* [Ejecución de una aplicación en un contenedor de Docker](https://openliberty.io/guides/getting-started.html#running-the-application-in-a-docker-container){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo")

### WebSphere Liberty y Docker
{: #wlp-docker}

Existen algunas diferencias entre Open Liberty y la versión comercial, WebSphere Liberty. Una de las diferencias más significativas para la creación de imágenes de Docker es que el mandato `installUtility` no está disponible en Open Liberty.

WebSphere Liberty da soporte a los mismos patrones de personalización básicos que Open Liberty para imágenes de Docker. Sin embargo, el diseño modular de Liberty facilita (y normaliza) la creación de una imagen personalizada que contenga una app y el conjunto específico de características que necesita. WebSphere Liberty tiene una imagen para un kernel sin función, [`websphere-liberty:kernel`](https://github.com/WASdev/ci.docker/blob/9d28dfba4d20596f89b393bc9e3ae8295feec469/ga/developer/kernel/Dockerfile){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo"), que constituye una base sólida para una imagen realmente personalizada.

La [documentación de la imagen `websphere-liberty`](https://hub.docker.com/_/websphere-liberty/){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo") describe el siguiente Dockerfile sencillo de 3 líneas necesario para crear una imagen personalizada:

```docker
FROM websphere-liberty:kernel
COPY server.xml /config/
RUN installUtility install --acceptLicense defaultServer
```
{: codeblock}

La herramienta `installUtility` busca cualquier característica que necesite en el archivo `server.xml` que todavía no está disponible en la imagen de Liberty. A continuación, descarga esas características y las instala en la imagen de Docker. Este enfoque crea una imagen mínima, pero la lista de características implícitas en `server.xml` no funciona bien con las capas de Docker en memoria caché. Cualquier cambio en `server.xml` invalida la capa con las características instaladas, lo que requiere que las características que faltan se instalen de nuevo la próxima vez que se construya la imagen.

Una mejor forma de crear imágenes personalizadas es invocar `installUtility` con una lista explícita de características. Este método crea una imagen mínima igualmente, pero se comporta mucho mejor en el momento de la compilación, ya que la capa de la imagen se puede almacenar en memoria caché y se puede reutilizar independientemente de los cambios de configuración del servidor. Por ejemplo, el código siguiente crea una imagen personalizada utilizando un subconjunto de características para dar soporte a una app que utilice JAX-RS, JNDI y WebSockets:

```docker
FROM websphere-liberty:kernel
RUN /opt/ibm/wlp/bin/installUtility install --acceptLicense \
  cdi-1.2 \
  concurrent-1.0 \
  jaxrs-2.0 \
  jndi-1.0 \
  ssl-1.0 \
  websocket-1.1
COPY server.xml /config/
```
{: codeblock}

Cómo estructurar la imagen de Docker depende de varios factores:

1. ¿Hasta qué punto desea que la imagen base sea reutilizable o personalizada?
    Estos pasos producen la imagen más pequeña posible, potencialmente a costa de la reutilización si las características de cada app son diferentes. Sin embargo, tener imágenes personalizadas pequeñas significa que la app se puede desplegar rápidamente en los hosts de Docker. Si no está seguro, empiece con una de las imágenes pregeneradas de Docker Hub para aumentar el nivel de reutilización.
2. ¿Con qué frecuencia actualiza esta imagen?
    Si la app se actualiza con frecuencia, por ejemplo en un conducto CI/CD, es beneficioso para estructurar la imagen, de modo que la capa modificada más frecuentemente (a menudo, las clases de app o el binario de la app) esté en la parte superior. Esta estructura reduce el número de capas que se deben volver a crear cuando la app cambia y la imagen se reconstruye, y se acelera el tiempo de compilación y de despliegue.

En caso de duda, siga la [Guía de Open Liberty Docker](https://openliberty.io/guides/docker.html){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo") para empezar.
