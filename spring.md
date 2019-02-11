---

copyright:
  years: 2019
lastupdated: "2019-02-05"

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

The Spring Platform is an ecosystem of projects that aim to make it simpler to create Java applications. Usually the term "Spring" refers to the Spring Framework, but it is also used generically to refer to any project that is part of, or uses Spring based technologies, including Spring Boot.

## Spring Framework
{: #spring-framework}

The Spring Framework was introduced in 2002, and has released a new major version approximately every 3 years since. The framework comprises a large set of components (referred to as Spring Modules) that cover everything from REST Endpoints to Database Abstractions. With its most recent release in 2017, Spring Framework 5 introduced updated JDK support, alongside with WebFlux an new module for reactive programming based on [Project Reactor](https://projectreactor.io/{: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon").

Spring Framework 4.3 is the last feature branch, with support through 2020. Applications using older versions of Spring should be migrated to Spring Framework 5.

The comprehensive documentation for Spring Framework is here:

* [Spring Framework Reference Guide for 5.1.x](https://docs.spring.io/spring/docs/5.1.x/spring-framework-reference/){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon")
* [Spring Framework Reference Guide for 4.3.x](https://docs.spring.io/spring/docs/4.3.x/spring-framework-reference/){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon")
* [Upgrading to Spring Framework 5.x](https://github.com/spring-projects/spring-framework/wiki/Upgrading-to-Spring-Framework-5.x){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon")

## Spring Boot
{: #spring-boot}

Spring Boot was introduced in 2014, to "improve containerless web application architectures". Its goal was to move configuration from XML into code. Spring Boot provides mechanisms for creating applications based on an opinionted view of what technologies should be used. Spring Boot applications are Spring applications, and use a programming model unique to that ecosystem.

Spring Boot emphasizes convention over configuration, and uses a combination of annotations and classpath discovery to enable additional functions. By default, Spring Boot applications are built into a self-contained runnable jar file that includes all required dependencies (including an embedded Tomcat server). Applications can alternately be packaged as war files deployed to an application server.

Spring Boot is now at version 2.0, released in 2018. Maintenance of 1.5.x branch will stop in August of 2019.

The comprehensive documentation for Spring Boot is here:

* [Spring Boot Reference Guide for 2.1.x](https://docs.spring.io/spring-boot/docs/2.1.x/reference){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon")
* [Spring Boot Reference Guide for 1.5.x](https://docs.spring.io/spring-boot/docs/1.5.x/reference/){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon")

## Selecting Spring Framework or Spring Boot
{: #spring-framework-or-boot}

For new cloud native applications, if you choose to use Spring Framework, you should also use Spring Boot. Spring Boot simplifies configuration and assembly of Spring-based applications, and provides features, like Spring Actuator, that simplify creating [cloud native applications](/docs/java?topic=cloud-native-overview#overview).

### Spring Boot Actuator
{: #spring-boot-actuator}

Spring Boot Actuator defines a collection of endpoints that are useful for inspecting a running Spring app. Enabling them is as simple as adding a single dependency to the application.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```
{: codeblock}

The structure and behavior of Spring Boot Actuator changed significantly between Spring Boot 1 and Spring Boot 2. The Boot 1 Actuator sat alongside the application, with separate configuration and security mechanisms. The Boot 2 version is more capable, and uses a security model that integrates with the rest of the Spring application.

The behavior changes are particularly relevant if you are migrating a Boot 1 application to Boot 2. Read the Actuator section of the [Boot 1 to Boot 2 migration guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.0-Migration-Guide#spring-boot-actuator) for more details.
{: tip}

In Boot 2, the Actuator acts as mini-REST framework. All endpoints are under an `/actuator` context. Endpoints for performing health checks, or gathering metrics, application or other environment information are provided. For a complete list, check the [Actuator documentation](https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/html/production-ready-features.html#production-ready).

It is easy to create your own Actuator endpoint using a `@SpringComponent`:

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

Spring Boot has two controls to configure which Actuators are active at runtime, Actuators can be "enabled", and "exposed". To be able be reachable, an Actuator must be both. By default many endpoints are enabled, but only health and info are exposed. Additionally if Spring Security is enabled for the application, access must be explicitly allowed to the Actuator endpoints.

The Actuator in Spring Boot 1 has its own security configuration, usually configured in application.properties. Instead of "enabled" and "exposed", there is "enabled", and "sensitive". Sensitive endpoints require authentication if served over HTTP. By default, the /metrics endpoint of the Spring Boot Actuator is enabled in Spring Boot 1, but it is considered sensitive and thus requires authorization, or to be set as not sensitive. For some environments, configuring Spring Security might be the right answer, but in Kubernetes, metrics endpoints remain internal to the cluster. For more information check the [Spring Boot 1 Actuator docs](https://docs.spring.io/spring-boot/docs/1.5.2.RELEASE/reference/htmlsingle/#production-ready)
{: note}

## Spring Cloud
{: #spring-cloud}

Spring Cloud is a collection of integrations between third-party cloud technologies and the Spring programming model. It aims to provide assistance to developers building Spring applications for deployment to the cloud. The breadth of coverage attained Spring Cloud is made possible by its modular nature, each project within Spring Cloud is focused on a particular technology or set of technologies.

Spring Cloud projects follow the general Spring approach of favouring convention over configuration. Most capabilities are enabled by adding the right dependency at build time.

See the official [Spring Cloud Project Site]
(https://spring.io/projects/spring-cloud){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon") for more information about Spring Cloud projects and supported technologies.

### Spring Cloud with Kubernetes
{: #spring-cloud-kubernetes}

Spring Cloud Kubernetes is a Spring Cloud project intended to make various tasks easier for Spring applications running in Kubernetes. Spring Cloud Kubernetes offers implementations of common Spring interfaces that map onto Kubernetes concepts and services.

Historically, Spring has used Netflix libraries like Eureka as a Service Registry and Ribbon as a client side Load Balancer. In Kubernetes environments, both of these roles are fulfilled by Kubernetes itself, making the application-level capabilities redundant. Spring Cloud Kubernetes adapts Spring abstractions like `DiscoveryClient` to underlying Kubernetes mechanisms.

Care should be taken if using Ribbon discovery via Spring Cloud Kubernetes in a cluster with Istio installed where Istio is being used to affect service routing. Client side load balancing implies the client wishes to select the destination service itself, while Istio may be attempting to route the call elsewhere. Only one of these can win, leading to disagreement over how a call should have been routed. This combination should be avoided if at all possible.
{: note}

In addition, Spring Cloud Kubernetes offers integration between Kubernetes ConfigMaps and Secrets to Spring `@Autowired` configuration beans. This includes strategies for handling dynamic changes to ConfigMaps and Secrets made while the application is running.

Finally, Spring Cloud Kubernetes augments the default Spring Boot Actuator health endpoint to include additional information related to the deployment.

For more information, check the [Spring Cloud Kubernetes documentation](https://cloud.spring.io/spring-cloud-static/spring-cloud-kubernetes/2.1.0.RC1/single/spring-cloud-kubernetes.html){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon")


<!--
### Spring Cloud Streams
{: #spring-cloud-streams}


:FIXME:
-->
