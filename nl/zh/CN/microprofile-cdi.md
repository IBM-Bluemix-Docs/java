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

# 上下文和依赖项注入（CDI，JSR-365）
{: #mp-cdi}

CDI 是 Jakarta-EE 和 MicroProfile 的核心元素，用于将各种组件连线在一起。CDI 定义**上下文**来指定和管理 Bean 生命周期，并使用**依赖项注入**以动态和类型安全的方式满足声明的依赖项。CDI 还使用松散耦合的事件和拦截器模型，并为其他框架提供强大而灵活的可移植扩展机制，以定义自己的 CDI Bean 或更新现有组件。

MicroProfile 规范（例如，MicroProfile Config、MicroProfile JWT 等）采用 CDI 优先方法，依赖 CDI 扩展机制通过新功能来增强现有标准（如 JAX-RS）。CDI 1.2 是 MicroProfile 1.0 发行版及后续发行版的一部分（MicroProfile 2.0 升级到 CDI 2.0）。

## CDI Bean
{: #cdi-bean}

CDI 在 _Bean_ 上运行。CDI Bean 是使用 [Bean 定义注释 ](https://docs.jboss.org/cdi/spec/2.0/cdi-spec.html){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标") 创建的。包含不带自变量的构造函数或包含带有 `@Inject` 注释的构造函数的几乎每个普通旧式 Java 对象 (POJO) 都可以是 Bean。

必须发现 CDI Bean 定义注释。可以使用 JAR 归档的 `META-INF` 文件夹中或 WAR 归档的 `WEB-INF` 文件夹中的 `beans.xml` 文件来启用 CDI 注释扫描。此文件可以为空，也可以包含类似下面的内容：

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

存在空的 `beans.xml` 或带有 `bean-discovery-mode="all"` 的 `beans.xml`，都会使所有潜在的对象成为 CDI Bean。在其他情况下，只有具有 CDI Bean 定义注释的对象才是 CDI Bean。

## CDI 作用域
{: #cdi-scopes}

CDI 作用域类型注释用于控制 Bean 的生命周期：

* 如果实例应该在应用程序运行期间保持活性，请使用 `@ApplicationScoped` 类级别注释。
* 如果应该为每个请求（例如，Servlet 请求）创建一个新的 Bean 实例，请使用 `@RequestScoped` 类级别注释。
* 如果新实例应该属于其他对象，请使用 `@Dependent` 类级别注释。从属 Bean 的实例绝不会在不同客户端之间或不同注入点之间共享。从属 Bean 在其所属对象创建时进行实例化，然后在其所属对象被销毁时会随之销毁。
* 如果未定义类级别注释，那么 `@Dependent` 是缺省设置。

CDI 容器根据其定义的作用域来创建和销毁 Bean 实例，将其与相应的上下文相关联，然后将其作为依赖项注入其他对象。


## CDI Bean 注入
{: #cdi-inject}

CDI 通过 `@Inject` 将定义的 Bean 注入其他组件。例如，以下 POJO `MyBean` 是 CDI Bean：

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

因为这是 `@ApplicationScoped`，因此只会创建一个实例。在以下示例中，`MyRestEndPoint` 是 `@RequestScoped`，这意味着将为每个请求创建一个实例。CDI 会在每个请求中注入相同的 `MyBean` 实例。

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

CDI 将管理这些 Bean 的生命周期：

* 使用通过 `@PostConstruct` 注释的方法而不是构造函数来初始化 Bean。在 CDI 容器将依赖项和关联的 Bean 注入相应的上下文后，将对其进行调用。
* 使用通过 `@PreDestroy` 注释的方法来清除 Bean。在除去依赖项之前，将对其进行调用。
