---

copyright:
  years: 2019
lastupdated: "2019-08-26"

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

The Spring Platform is an ecosystem of projects that aim to make it simpler to create Java&trade; applications. The term "Spring" refers to the Spring Framework, and is also used generically to refer to any project that uses Spring-based technologies, including Spring Boot.

## Spring Framework
{: #spring-framework}

The Spring Framework was introduced in 2002, and releases a major version approximately every 3 years. The framework comprises a large set of components (referred to as Spring Modules) that cover everything from REST endpoints to database abstractions. With its most recent release in 2017, Spring Framework 5 introduced updated JDK support, and WebFlux. WebFlux is a new module for reactive programming based on [Project Reactor](https://projectreactor.io/){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon").

Spring Framework 4.3 is the last feature branch for Spring Framework 4.x, with support through 2020. Applications that use older versions of Spring must be migrated to Spring Framework 5.

The comprehensive documentation for Spring Framework is here:

* [Spring Framework Reference Guide for 5.1.x](https://docs.spring.io/spring/docs/5.1.x/spring-framework-reference/){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon")
* [Spring Framework Reference Guide for 4.3.x](https://docs.spring.io/spring/docs/4.3.x/spring-framework-reference/){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon")
* [Upgrading to Spring Framework 5.x](https://github.com/spring-projects/spring-framework/wiki/Upgrading-to-Spring-Framework-5.x){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon")

## Spring Boot
{: #spring-boot}

Spring Boot was introduced in 2014, to "improve containerless web application architectures". Its goal was to move configuration from XML into code. Spring Boot provides mechanisms for creating apps based on an opinionated view of what technologies are to be used. Spring Boot apps are Spring apps, and use a programming model unique to that ecosystem.

Spring Boot emphasizes convention over configuration, and uses a combination of annotations and class path discovery to enable functions. By default, Spring Boot applications are built into a self-contained runnable JAR file that includes all required dependencies (including an embedded Tomcat server). Apps can alternately be packaged as war files deployed to an app server.

Spring Boot is now at version 2.0, which released in 2018. Maintenance of 1.5.x branch ends in August of 2019.

The comprehensive documentation for Spring Boot is here:

* [Spring Boot Reference Guide for 2.1.x](https://docs.spring.io/spring-boot/docs/2.1.x/reference/){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon")
* [Spring Boot Reference Guide for 1.5.x](https://docs.spring.io/spring-boot/docs/1.5.x/reference/){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon")

## Selecting Spring Framework or Spring Boot
{: #spring-framework-or-boot}

For new cloud native apps, if you choose to use Spring Framework, you can also use Spring Boot. Spring Boot simplifies configuration and assembly of Spring-based apps by using features like Spring Actuator that simplify creating [cloud native apps](/docs/java?topic=cloud-native-overview#overview).

### Spring Boot Actuator
{: #spring-boot-actuator}

Spring Boot Actuator defines a collection of endpoints that are useful for inspecting a running Spring app. Enabling them is as simple as adding a single dependency to the app.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```
{: codeblock}

The structure and behavior of Spring Boot Actuator changed significantly between Spring Boot 1.0 and Spring Boot 2.0. The Boot 1.0 Actuator sat alongside the app, with separate configuration and security mechanisms. The Boot 2.0 version is more capable, and uses a security model that integrates with the rest of the Spring app.

The behavior changes are relevant if you are migrating a Spring Boot 1.x app to Spring Boot 2.0. Read the Actuator section of the [Boot 1.0 to Boot 2.0 migration guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.0-Migration-Guide#spring-boot-actuator){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon") for more details.
{: tip}

In Boot 2, the Actuator acts as mini-REST framework. All endpoints are under an `/actuator` context. Endpoints for performing health checks, or gathering metrics, app, or other environment information are provided. For a complete list, check the [Actuator documentation](https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/html/production-ready-features.html#production-ready){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon").

It is easy to create your own Actuator endpoint by using a Spring `@Component`:

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

Spring Boot has two controls to configure which Actuators are active at run time. Actuators can be "enabled" and "exposed". To be reachable, an Actuator must be both. By default many endpoints are enabled, but only health and information are exposed. Additionally if Spring Security is enabled for the app, access must be explicitly allowed to the Actuator endpoints.

The Actuator in Spring Boot 1 has its own security configuration, usually configured in application.properties. Instead of "enabled" and "exposed", there is "enabled", and "sensitive". Sensitive endpoints require authentication if served over HTTP. By default, the /metrics endpoint of the Spring Boot Actuator is enabled in Spring Boot 1, but it is considered sensitive and thus requires authorization, or to be set as not sensitive. For some environments, configuring Spring Security might be the right answer, but in Kubernetes, metrics endpoints remain internal to the cluster. For more information, see [Spring Boot 1.0 Actuator](https://docs.spring.io/spring-boot/docs/1.5.2.RELEASE/reference/htmlsingle/#production-ready){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon").
{: note}

## Spring Cloud
{: #spring-cloud}

Spring Cloud is a collection of integrations between third-party cloud technologies and the Spring programming model. It aims to assist developers that build Spring apps for deployment to the cloud. The coverage that is attained by Spring Cloud is made possible by its modular nature, as each project within Spring Cloud is focused on a particular technology or set of technologies.

Spring cloud projects follow the general Spring approach of favoring convention over configuration. Most capabilities are enabled by adding the right dependency at build time.

For more information about Spring Cloud projects, see the official [Spring Cloud Project Site](https://spring.io/projects/spring-cloud){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon").

### Spring Cloud with Kubernetes
{: #spring-cloud-kubernetes}

Spring Cloud Kubernetes is a Spring Cloud project that is intended to make various tasks easier for Spring apps that run in Kubernetes. Spring Cloud Kubernetes offers implementations of common Spring interfaces that map onto Kubernetes concepts and services.

Historically, Spring uses Netflix libraries like Eureka, as a Service Registry, and Ribbon as a client-side Load Balancer. In Kubernetes environments, both of these roles are fulfilled by Kubernetes itself, making the application-level capabilities redundant. Spring Cloud Kubernetes adapts Spring abstractions like `DiscoveryClient` to underlying Kubernetes mechanisms.

Use caution if you are using Ribbon discovery through Spring Cloud Kubernetes in a cluster with Istio installed, where Istio is being used to affect service routing. Client-side load balancing implies the client wants to select the destination service itself, while Istio can be attempting to route the call elsewhere. Only one method can win, leading to disagreement over how a call can be routed. This combination is to be avoided if at all possible.
{: note}

In addition, Spring Cloud Kubernetes offers integration between Kubernetes `ConfigMaps` and `Secrets` to Spring `@Autowired` configuration beans. Strategies are included for handling dynamic changes to `ConfigMaps` and `Secrets` while the app is running.

Finally, Spring Cloud Kubernetes augments the default Spring Boot Actuator health endpoint to include additional information that is related to the deployment.

For more information, check the [Spring Cloud Kubernetes documentation](https://cloud.spring.io/spring-cloud-static/spring-cloud-kubernetes/2.1.0.RC1/single/spring-cloud-kubernetes.html){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon")


<!--
### Spring Cloud Streams
{: #spring-cloud-streams}


:FIXME:
-->
