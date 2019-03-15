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

# Contexts and Dependency Injection (CDI, JSR-365)
{: #mp-cdi}

CDI is a central element in both Jakarta-EE and MicroProfile, as it is used to wire various components together. CDI defines **contexts** to specify and manage bean lifecycles, and uses **dependency injection** to satisfy declared dependencies in a dynamic and typesafe way. CDI also uses a loosely-coupled event and interceptor model, and provides a powerful and flexible portable extension mechanism for other frameworks to define their own CDI beans or update existing components.

MicroProfile specifications (e.g. MicroProfile Config, MicroProfile JWT etc.) take CDI-first approach, relying on the CDI extension mechanism to enhance existing standards (like JAX-RS) with new capabilities. CDI 1.2 is part of MicroProfile 1.0 release and thereafter (MicroProfile 2.0 moves up to CDI 2.0).

## CDI Beans
{: #cdi-bean}

CDI operates on _beans_. CDI beans are created using [Bean-defining annotations](https://docs.jboss.org/cdi/spec/2.0/cdi-spec.html){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon"). Almost every plain old java object (POJO) that has either a constructor with no arguments or a constructor with the `@Inject` annotation can be a bean.

CDI Bean defining annotations must be discovered. CDI annotation scanning can be enabled using a `beans.xml` file ia the `META-INF` folder for a jar archive or the `WEB-INF` folder for a war archive. This file can be empty, or it can contain something like the following content:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://xmlns.jcp.org/xml/ns/javaee"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
            http://xmlns.jcp.org/xml/ns/javaee/beans_1_1.xsd"
      version="1.1" bean-discovery-mode="all">
  // enable interceptors; decorators; alternatives
</beans>
```
{: codeblock}

The presence of an empty `beans.xml` or a `beans.xml` with `bean-discovery-mode="all"` makes all potential objects CDI beans. Otherwise, only objects with CDI bean-defining annotations are CDI beans.

## CDI Scopes
{: #cdi-scopes}

CDI scope type annotations control the bean's lifecycle:

* Use the `@ApplicationScoped` class-level annotation if an instance should live as long as the Application is running.
* Use the `@RequestScoped` class-level annotation if a new instance of the bean should be created for every request (a Servlet request, for example).
* Use the `@Dependent` class-level annotation if the new instance should belong to another object. An instance of a dependent bean is never shared between different clients or different injection points. It is instantiated when the object it belongs to is created, and then destroyed when the object it belongs is destroyed.
* If no class-level annotation is defined, `@Dependent` is the default.

The CDI container creates and destroys bean instances according to their defined scope, associates them with the appropriate context, and then injects them as dependencies in other objects.

## CDI Bean injection
{: #cdi-inject}

CDI will inject defined beans into other components via `@Inject`. For instance, the following POJO `MyBean` is a CDI bean:

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

As it is `@ApplicationScoped`, only one instance will be created. In the following, `MyRestEndPoint` is `@RequestScoped`, which means an instance will be created for each request. CDI will inject the same `MyBean` instance into each.

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

CDI is managing the lifecycle of these beans:

* Use a method annotated with `@PostConstruct` instead of a constructor to initialize your bean. It will be invoked after the CDI container has injected dependencies and associated bean to the proper context.
* Use a method annotated with `@PreDestroy` to clean up your bean. It will be invoked before dependencies are removed.
