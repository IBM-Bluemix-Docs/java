---

copyright:
  years: 2019
lastupdated: "2019-03-15"

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

Spring Platform 是一种项目生态系统，旨在简化创建 Java 应用程序的过程。通常，术语“Spring”是指 Spring Framework，但也用于泛指属于基于 Spring 的技术（包括 Spring Boot）一部分或使用这些技术的任何项目。

## Spring Framework
{: #spring-framework}

Spring Framework 于 2002 年推出，此后大约每 3 年发布一个新的主版本。该框架包含大量组件（称为 Spring 模块），涵盖了从 REST 端点到数据库抽象的所有内容。在 2017 年的最新发行版中，Spring Framework 5 推出了更新的 JDK 支持，与 WebFlux 一起作为一个新模块，用于基于 [Project Reactor](https://projectreactor.io/){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标") 的反应式编程。

Spring Framework 4.3 是最新的功能分支，会一直支持到 2020 年。使用旧版本 Spring 的应用程序应该迁移到 Spring Framework 5。

Spring Framework 的综合文档如下：

* [Spring Framework 5.1.x 参考指南](https://docs.spring.io/spring/docs/5.1.x/spring-framework-reference/){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")
* [Spring Framework 4.3.x 参考指南](https://docs.spring.io/spring/docs/4.3.x/spring-framework-reference/){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")
* [Upgrading to Spring Framework 5.x](https://github.com/spring-projects/spring-framework/wiki/Upgrading-to-Spring-Framework-5.x){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")

## Spring Boot
{: #spring-boot}

Spring Boot 于 2014 年推出，用于“改进无容器 Web 应用程序体系结构”。其目标是将配置从 XML 移至代码。Spring Boot 提供了基于应该使用哪些技术的固有观点来创建应用程序的机制。Spring Boot 应用程序是 Spring 应用程序，并使用该生态系统独有的编程模型。

相对于配置，Spring Boot 更注重惯例，并使用注释和类路径发现的组合来启用其他功能。缺省情况下，Spring Boot 应用程序会构建成包括所有必需依赖项（包括嵌入的 Tomcat 服务器）的自包含可运行 JAR 文件。应用程序可选择打包为部署到应用程序服务器的 WAR 文件。

Spring Boot 现在为 V2.0，于 2018 年发布。对 1.5.x 分支的维护将于 2019 年 8 月停止。

Spring Boot 的综合文档如下：

* [Spring Boot 2.1.x 参考指南](https://docs.spring.io/spring-boot/docs/2.1.x/reference){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")
* [Spring Boot 1.5.x 参考指南](https://docs.spring.io/spring-boot/docs/1.5.x/reference/){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")

## 选择 Spring Framework 或 Spring Boot
{: #spring-framework-or-boot}

对于新的云本机应用程序，如果选择使用 Spring Framework，那么还应该同时使用 Spring Boot。Spring Boot 简化了基于 Spring 的应用程序的配置和组合，并提供了多种功能（如 Spring Actuator）来简化创建[云本机应用程序](/docs/cloud-native?topic=cloud-native-overview#overview)。

### Spring Boot Actuator
{: #spring-boot-actuator}

Spring Boot Actuator 定义了对于检查运行中的 Spring 应用程序很有用的一组端点。启用这些端点很简单，只需将单个依赖项添加到应用程序即可。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```
{: codeblock}

Spring Boot Actuator 的结构和行为在 Spring Boot 1 和 Spring Boot 2 之间有很大变化。Boot 1 Actuator 与应用程序绑定在一起，但具有单独的配置和安全机制。Boot 2 版本功能更多，并使用与 Spring 应用程序的其余部分集成在一起的安全模型。

如果要从 Boot 1 应用程序迁移到 Boot 2，那么行为更改尤其重要。请阅读 [Boot 1 到 Boot 2 迁移指南](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.0-Migration-Guide#spring-boot-actuator){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标") 中的 Actuator 部分，以获取更多详细信息。
{: tip}

在 Boot 2 中，Actuator 充当微型 REST 框架。所有端点都位于 `/actuator` 上下文下。提供了用于执行运行状况检查或收集度量值、应用程序或其他环境信息的端点。有关完整列表，请查看 [Actuator 文档](https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/html/production-ready-features.html#production-ready){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")。

使用 Spring `@Component` 可轻松创建您自己的 Actuator 端点：

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

Spring Boot 具有两个控件，用于配置哪些 Actuator 在运行时处于活动状态，哪些 Actuator 可以为“已启用”和“已公开”。Actuator 必须已启用并公开后，才可对其进行访问。缺省情况下，会启用许多端点，但仅公开 health 和 info 端点。此外，如果应用程序启用了 Spring 安全性，那么必须显式允许访问 Actuator 端点。

Spring Boot 1 中的 Actuator 具有自己的安全配置，通常在 application.properties 中进行配置。Actuator 的配置不是“已启用”和“已公开”，而是“已启用”和“敏感”。敏感端点需要认证（如果通过 HTTP 提供）。缺省情况下，Spring Boot Actuator 的 /metrics 端点在 Spring Boot 1 中已启用，但被视为敏感端点，因此需要授权，或者将其设置为不敏感。对于某些环境，配置 Spring 安全性可能是正确的应对措施，但在 Kubernetes 中，度量值端点仍是集群内部端点。有关更多信息，请查看 [Spring Boot 1 Actuator 文档](https://docs.spring.io/spring-boot/docs/1.5.2.RELEASE/reference/htmlsingle/#production-ready){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")。
{: note}

## Spring Cloud
{: #spring-cloud}

Spring Cloud 是第三方云技术与 Spring 编程模型之间的一组集成，旨在帮助开发者构建 Spring 应用程序以部署到云。Spring Cloud 借助其模块化性质达到了一定广度的覆盖范围，Spring Cloud 中的每个项目都侧重于一种特定的技术或一组技术。

Spring Cloud 项目遵循惯例重于配置的一般 Spring 方法。大多数功能通过在构建时添加正确的依赖项来启用。

请参阅官方 [Spring Cloud 项目站点](https://spring.io/projects/spring-cloud){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")，以获取有关 Spring Cloud 项目和所支持技术的更多信息。

### 使用 Kubernetes 的 Spring Cloud
{: #spring-cloud-kubernetes}

Spring Cloud Kubernetes 是一个 Spring Cloud 项目，旨在为 Kubernetes 中运行的 Spring 应用程序简化各种任务。Spring Cloud Kubernetes 提供了对用于映射到 Kubernetes 概念和服务的常用 Spring 接口的实现。

一直以来，Spring 都是使用 Netflix 库（如 Eureka）作为服务注册表，使用 Ribbon 作为客户机端负载均衡器。在 Kubernetes 环境中，这两个角色都由 Kubernetes 自身履行，从而使应用程序级别的功能成为冗余。Spring Cloud Kubernetes 调整了 Spring 抽象（如 `DiscoveryClient`），使其适用于底层 Kubernetes 机制。

如果在安装了 Istio（用于影响服务路由）的集群中，通过 Spring Cloud Kubernetes 来使用 Ribbon 发现，那么应格外谨慎。客户机端负载均衡意味着客户机希望选择目标服务本身，而 Istio 却可能在尝试将调用路由到其他位置。这两者只有一个可以成功，导致对调用应该如何进行路由产生分岐。应该尽可能避免此组合。
{: note}

此外，Spring Cloud Kubernetes 提供了 Kubernetes ConfigMap 和 Secret 与 Spring `@Autowired` 配置 Bean 之间的集成。这包括用于处理应用程序运行期间对 ConfigMap 和 Secret 进行的动态更改的策略。

最后，Spring Cloud Kubernetes 会扩充缺省 Spring Boot Actuator 运行状况端点，以包含与部署相关的其他信息。

有关更多信息，请查看 [Spring Cloud Kubernetes 文档](https://cloud.spring.io/spring-cloud-static/spring-cloud-kubernetes/2.1.0.RC1/single/spring-cloud-kubernetes.html){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")。


<!--
### Spring Cloud Streams
{: #spring-cloud-streams}


:FIXME:
-->
