---

copyright:
  years: 2019
lastupdated: "2019-04-22"

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

Die Kontext- und Abhängigkeitsinjektion (Contexts and Dependency Injection, CDI) ist ein zentrales Element sowohl in Jakarta EE als auch in MicroProfile, da es verwendet wird, um verschiedene Komponenten miteinander zu verbinden. CDI definiert **Kontexte** zum Angeben und Verwalten von Bean-Lebenszyklen und verwendet die **Abhängigkeitsinjektion**, um deklarierte Abhängigkeiten in einer dynamischen und typsicheren Weise zu erfüllen. CDI verwendet außerdem ein lose verbundenes Ereignis- und Interceptormodell und stellt einen leistungsfähigen und flexiblen, portierbaren Erweiterungsmechanismus für andere Frameworks zur Verfügung, damit diese eigene CDI-Beans definieren oder vorhandene Komponenten aktualisieren können.

MicroProfile-Spezifikationen (z. B. MicroProfile Config, MicroProfile JWT) verfolgen den Ansatz 'CDI first' und verlassen sich auf den CDI-Erweiterungsmechanismus zur Optimierung vorhandener Standards (wie JAX-RS) mit neuen Funktionen. CDI 1.2 ist Bestandteil von MicroProfile-Releases ab Version 1.0 (MicroProfile 2.0 verwendet dann CDI 2.0).

## CDI-Beans
{: #cdi-bean}

CDI arbeitet mit _Beans_. CDI-Beans werden mithilfe von [Bean-definierenden Annotationen](https://docs.jboss.org/cdi/spec/2.0/cdi-spec.html){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") erstellt. Fast jedes Plain Old Java& Object (POJO), das entweder einen Konstruktor ohne Argumente oder einen Konstruktor mit der Annotation `@Inject` aufweist, kann eine Bean sein.

Annotationen, die CDI-Beans definieren, müssen aufgespürt werden. Das Scannen von CDI-Annotationen kann mithilfe der Datei `beans.xml` im Ordner `META-INF` für ein JAR-Archiv oder im Ordner `WEB-INF` für ein WAR-Archiv aktiviert werden. Diese Datei kann leer sein oder in etwa folgenden Inhalt enthalten:

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

Das Vorhandensein einer leeren Datei `beans.xml` oder einer Datei `beans.xml` mit `bean-discovery-mode="all"` macht alle potenziellen Objekte zu CDI-Beans. Andernfalls sind nur Objekte mit Annotationen, die CDI-Beans definieren, CDI-Beans.

## CDI-Scopes
{: #cdi-scopes}

Annotationen, die sich auf den CDI-Scope beziehen, steuern den Lebenszyklus der Bean:

* Verwenden Sie die Annotation `@ApplicationScoped` auf Klassenebene, wenn die Lebensdauer einer Instanz so lang wie die Ausführung der Anwendung sein soll.
* Verwenden Sie die Annotation `@RequestScoped` auf Klassenebene, wenn für jede Anforderung eine neue Instanz der Bean erstellt werden soll (z. B. eine Servlet-Anforderung).
* Verwenden Sie die Annotation `@Dependent` auf Klassenebene, wenn die neue Instanz zu einem anderen Objekt gehören soll. Eine Instanz einer abhängigen Bean wird nie von verschiedenen Clients oder unterschiedlichen Injektionspunkten gemeinsam genutzt. Sie wird instanziiert, wenn das Objekt, zu dem sie gehört, erstellt wird. Wenn das zugehörige Objekt gelöscht wird, wird auch die Instanz gelöscht.
* Wenn keine Annotation auf Klassenebene definiert ist, ist `@Dependent` der Standardwert.

Der CDI-Container erstellt und löscht Bean-Instanzen entsprechend ihrem definierten Scope, ordnet sie dem entsprechenden Kontext zu und injiziert sie dann als Abhängigkeiten in andere Objekten.

## CDI-Bean-Injektion
{: #cdi-inject}

CDI injiziert definierte Beans mithilfe von `@Inject` in andere Komponenten. Das folgende POJO `MyBean` ist beispielsweise eine CDI-Bean:

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

Da es sich um `@ApplicationScoped` handelt, wird nur eine Instanz erstellt. Im Folgenden hat `MyRestEndPoint` den Wert `@RequestScoped`. Dies bedeutet, dass eine Instanz für jede Anforderung erstellt wird. CDI injiziert die gleiche `MyBean`-Instanz in jede Anforderung.

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

CDI verwaltet den Lebenszyklus dieser Beans:

* Verwenden Sie anstelle eines Konstruktors eine mit `@PostConstruct` annotierte Methode, um Ihre Bean zu initialisieren. Sie wird aufgerufen, nachdem der CDI-Container Abhängigkeiten und die zugehörige Bean in den richtigen Kontext injiziert hat.
* Verwenden Sie eine Methode, die mit `@PreDestroy` annotiert ist, um Ihre Bean zu bereinigen. Sie wird aufgerufen, bevor Abhängigkeiten entfernt werden.
