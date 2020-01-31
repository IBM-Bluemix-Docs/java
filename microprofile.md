---

copyright:
  years: 2019
lastupdated: "2019-04-04"

keywords: java ee, jakarta ee, microprofile overview, cloud native java, cloud native microprofile

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
{:external: target="_blank" .external}

# Overview
{: #jee-overview}

## Java EE and Jakarta EE
{: #jakarta-ee}

Technologies that are introduced in Java&trade; EE7 and EE8 are well-suited for creating microservices in Java&trade;, with support for annotations, light-weight protocols, and asynchronous interaction patterns. Concurrency utilities in Java&trade; EE7 provide a straight-forward mechanism for using reactive libraries like RxJava in a container-friendly way.

In September 2017, Oracle, supported by {{site.data.keyword.IBM}} and Red Hat, announced that Java&trade; EE was going to move to the Eclipse Foundation, as Jakarta EE. Oracle continues to own everything associated with Java&trade; EE, up to and including Java&trade; EE 8. But all future development of the Enterprise Java&trade; platform after Java&trade; EE 8 is handled by the Eclipse Foundation under the guidance of the Jakarta EE working group. In early 2018, Oracle started the large effort of relicensing and contributing the Java&trade; EE technologies that include APIs, RIs, TCKs, and associated project documentation to the EE4J project.

## MicroProfile
{: #microprofile}

While Java&trade; EE provides a solid foundation to create microservices, it needed technologies and programming models to better suit microservice applications. IBM and other companies worked together to launch MicroProfile, an open collaboration between developers, the community, and vendors.

The MicroProfile community is focused on rapid innovation around microservices and Enterprise Java&trade;. This community builds and integrates technologies that are best suited for cloud-native apps that follow microservices architectural patterns. Each MicroProfile release contains a set of technologies. MicroProfile 1.0, released in 2016, contained a subset of specifications from Java&trade; EE 7 that provide a set of core capabilities for lightweight microservices: CDI 1.2, JAX-RS 2.0, and JSON-P 1.0. The MicroProfile platform includes technologies to address unique cloud concerns, like configuration injection, fault tolerance, health checks, metrics, open tracing. It also defines vendor-agnostic standards for working with OpenAPI definitions and creating REST clients that make it easier to consume REST APIs, and for JWT propagation to address app security concerns.

To start participating in the open source group, visit [https://microprofile.io/](https://microprofile.io/){: external} or [https://projects.eclipse.org/projects/technology.microprofile](https://projects.eclipse.org/projects/technology.microprofile){: external}.
