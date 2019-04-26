---

copyright:
  years: 2018, 2019
lastupdated: "2019-04-09"

keywords: spring environment, spring credentials, ibm-cloud-spring-boot-service-bind, service bindings spring, vcap_services spring, access credential spring

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

# Configuración del entorno Spring
{: #spring-configuration}

La gestión de la configuración de servicio y las credenciales (enlaces de servicio) varía entre plataformas. Cloud Foundry almacena detalles de enlace de servicio en un objeto JSON en serie que se pasa a la aplicación como una variable de entorno `VCAP_SERVICES`. Kubernetes almacena los enlaces de servicio como atributos JSON o sin formato en serie en `ConfigMaps` o `Secrets`, que se pueden pasar a la aplicación contenerizada como variables de entorno o montarse como volúmenes temporales. El desarrollo local variará. Trabajar con estas variaciones de una forma portátil sin tener vías de acceso de código específicas del entorno puede ser un reto.

## Adición de la configuración de {{site.data.keyword.cloud_notm}} a aplicaciones de Spring existentes
{: #spring-cloud-env}

La biblioteca [{{site.data.keyword.cloud_notm}} Spring Service Bindings](https://github.com/ibm-developer/ibm-cloud-spring-bind){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo") proporciona una forma coherente para que la aplicación Spring acceda a la configuración abstrayendo información de distintos entornos de nube, que incluyen entornos locales, Cloud Foundry Enterprise Environment, Kubernetes, máquinas virtuales y {{site.data.keyword.openwhisk}} en una matriz de patrones de búsqueda en un archivo `mappings.json`. Cuando se inicia la aplicación Spring, la biblioteca lee el archivo y aplica los patrones con el fin de descubrir y asignar valores de configuración.

Para obtener más información sobre el archivo `mappings.json`, consulte [Trabajo con credenciales de servicio inyectadas](/docs/java?topic=cloud-native-configuration#portable-credentials).

### Adición de la biblioteca a la aplicación Spring
{: #spring-add-service-library}

Para utilizar la biblioteca {{site.data.keyword.cloud_notm}} Spring Service Bindings, incluya la dependencia `ibm-cloud-spring-boot-service-bind` en el archivo `pom.xml` o `build.gradle`:

**Ejemplo de Maven**:

```xml
<dependency>
   <groupId>com.ibm.cloud</groupId>
   <artifactId>ibm-cloud-spring-boot-service-bind</artifactId>
   <version>1.1.1</version>
</dependency>
```
{: codeblock}

**Ejemplo de Gradle**:

```json
dependencies {
    compile group: 'com.ibm.cloud', name: 'ibm-cloud-spring-boot-service-bind', version: '1.1.1'
}
```
{: codeblock}

### Acceso a credenciales
{: #spring-access-credentials}

Para acceder a la configuración del servicio en el código, puede utilizar la anotación `@Value` o utilizar el método `getProperty()` de la clase Environment de la infraestructura de Spring.

Ejemplo: Acceso a los datos de configuración de un servicio IBM Cloudant NoSQL DB:

```java
   // Utilizar la anotación @Value
   @Value("cloudant.url")
   String cloudantUrl;

   // Utilizar el objeto Environment de Spring
   @Autowired
   Environment env;

   String cloudantUsername =
        env.getProperty("cloudant.username");
```
{: codeblock}

## Pasos siguientes
{: #spring-config-next-steps notoc}

Para obtener más información sobre Spring Boot:

* [Spring Boot 1.5.x](https://docs.spring.io/spring-boot/docs/1.5.x/reference/html/){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo")
* [Spring Boot 2.1.x](https://docs.spring.io/spring-boot/docs/2.1.x/reference/html/){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo")

Para obtener más información acerca de IBM y Spring:

* [IBM Developer: Spring](https://developer.ibm.com/technologies/spring/){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo")
* [IBM Spring Service Bindings](https://github.com/ibm-developer/ibm-cloud-spring-bind){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo")
