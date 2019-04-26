---

copyright:
  years: 2019
lastupdated: "2019-03-15"

keywords: dependency injection, cdi jakarta, cdi beans, cdi scopes, bean lifecycle, context injection microprofile, microprofile cdi

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

# Inyección de dependencias y contexto (CDI, JSR-365)
{: #mp-cdi}

CDI es un elemento central tanto en Jakarta-EE como en MicroProfile, ya que se utiliza para conectar varios componentes entre sí. CDI define **contextos** para especificar y gestionar ciclos de vida de bean y utiliza **inyección de dependencias** para satisfacer las dependencias declaradas de una forma dinámica y con seguridad de tipos. CDI también utiliza un modelo de suceso e interceptor acoplado ligeramente, y proporciona un mecanismo de ampliación portátil potente y flexible para otras infraestructuras para definir sus propios beans de CDI o actualizar los componentes existentes.

Las especificaciones de MicroProfile (por ejemplo, MicroProfile Config, MicroProfile JWT, etc.) adoptan el primer enfoque de CDI, basándose en el mecanismo de ampliación de CDI para mejorar los estándares existentes (como JAX-RS) con nuevas prestaciones. CDI 1.2 forma parte del release MicroProfile 1.0 y posteriores (MicroProfile 2.0 se traslada a CDI 2.0).

## Beans de CDI
{: #cdi-bean}

CDI opera en _beans_. Los beans de CDI se crean utilizando [Anotaciones de definición de bean](https://docs.jboss.org/cdi/spec/2.0/cdi-spec.html){: new_window}![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo"). Casi todos los objetos POJO (plain old Java object) que tienen un constructor sin argumentos o un constructor con la anotación `@Inject` pueden ser un bean.

Se deben descubrir las anotaciones de definición de bean de CDI. La exploración de anotaciones de CDI se puede habilitar utilizando un archivo `beans.xml` mediante la carpeta `META-INF` para un archivado jar o la carpeta `WEB-INF` para un archivado war. Este archivo puede estar vacío, o incluir un contenido similar al siguiente:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://xmlns.jcp.org/xml/ns/javaee"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
            http://xmlns.jcp.org/xml/ns/javaee/beans_1_1.xsd"
      version="1.1" bean-discovery-mode="all">
  // habilitar interceptores; decoradores; alternativas
</beans>
```
{: codeblock}

La presencia de un `beans.xml` vacío o un `beans.xml` con `bean-discovery-mode="all"` convierte a todos los objetos en beans de CDI potenciales. De lo contrario, sólo los objetos con anotaciones de definición de bean de CDI son beans de CDI.

## Ámbitos de CDI
{: #cdi-scopes}

Las anotaciones de tipo de ámbito de CDI controlan el ciclo de vida del bean:

* Utilice la anotación de nivel de clase `@ApplicationScoped` si el ciclo de vida de una instancia debe durar mientras se esté ejecutando la aplicación.
* Utilice la anotación de nivel de clase `@RequestScoped` si se debe crear una nueva instancia del bean para cada solicitud (por ejemplo, una solicitud de servlet).
* Utilice la anotación de nivel de clase `@Dependent` si la nueva instancia debe pertenecer a otro objeto. Una instancia de un bean dependiente nunca se comparte entre diferentes clientes o diferentes puntos de inyección. Se crea una instancia cuando se crea el objeto al que pertenece, y luego se destruye cuando el objeto al que pertenece se destruye.
* Si no se define ninguna anotación de nivel de clase, `@Dependent` es el valor predeterminado.

El contenedor de CDI crea y destruye instancias de bean de acuerdo con su ámbito definido, las asocia con el contexto adecuado y, a continuación, las inyecta como dependencias en otros objetos.

## Inyección de bean de CDI
{: #cdi-inject}

CDI inyectará beans definidos en otros componentes a través de `@Inject`. Por ejemplo, el siguiente POJO `MyBean` es un bean de CDI:

```java
@ApplicationScoped
public class MyBean {
    int i=0;
    public String sayHello() {
        return "MyBean hello " + i++;
    }
}
```
{: codeblock}

Dado que se trata de `@ApplicationScoped`, sólo se creará una instancia. En el caso siguiente, se utiliza `@RequestScoped` en `MyRestEndPoint`, lo que significa que se creará una instancia para cada solicitud. CDI inyectará la misma instancia de `MyBean` en cada una de ellas.

```java
@RequestScoped
@Path("/hello")
public class MyRestEndPoint {
    @Inject MyBean bean;

    @GET
    @Produces (MediaType.TEXT_PLAIN)
    public String sayHello() {
        return bean.sayHello();
    }
}
```
{: codeblock}

CDI gestiona el ciclo de vida de estos beans:

* Utilice un método anotado con `@PostConstruct` en lugar de un constructor para inicializar el bean. Se invocará después de que el contenedor de CDI haya inyectado dependencias y el bean asociado en el contexto adecuado.
* Utilice un método anotado con `@PreDestroy` para limpiar el bean. Se invocará antes de que se eliminen las dependencias.
