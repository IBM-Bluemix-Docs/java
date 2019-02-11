---

copyright:
  years: 2019
lastupdated: "2019-02-06"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}

# Java on {{site.data.keyword.Bluemix_notm}}
{: #overview}

Java applications come in all shapes and sizes, everything from standalone applications to Java EE / Jakarta EE, MicroProfile or Spring applications.

## OpenJDK with Open J9
{: #openjdk}

The OpenJDK project is the open source reference implementation of the Java language and provides API stability and compatibility to user programs.  The [Java SE platform](https://docs.oracle.com/javase/8/docs/){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon"), consists of APIs and the Java virtual machine (JVM).  The JVM is the execution engine responsible for executing the Java application.

The Eclipse OpenJ9 project provides an alternative JVM that replaces the Hotspot JVM in OpenJDK. The OpenJ9 JVM was open sourced by IBM in Sept of 2017, though its lineage goes back much further than that. OpenJ9 has been powering over 3000 IBM products, including WebSphere Application Server, for over a decade.

The IBM Java runtime is 100% open source, based on OpenJDK and the Eclipse OpenJ9 project.  This pairing provides improved performance and serviceability for cloud-native Java applications above and beyond traditional Hotspot-based Java runtimes.

### Downloading OpenJDK with OpenJ9
{: #downloads}

OpenJDK with OpenJ9 binaries are freely available from the Java community hub at [AdoptOpenJDK](https://adoptopenjdk.net/?jvmVariant=openj9){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon").  AdoptOpenJDK provides prebuilt OpenJDK binaries from a fully open source set of build scripts and infrastructure. Docker images are also available on [Dockerhub](https://hub.docker.com/u/adoptopenjdk){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon").

## Many Java Frameworks
{: #frameworks}

OpenJDK and OpenJ9 provide a solid foundation for any Java application, but application frameworks are often used to manage common concerns. Enterprise applications, for example, are typically built using either [Java EE (and Jakarta EE)](/docs/java?topic=mp-overview), or [Spring](/docs/java?topic=java-spring-overview).  [MicroProfile](/docs/java?topic=mp-overview) brings a faster  development pace and support for cloud-native concepts to Java EE.

[WebSphere Liberty](/docs/java?topic=java-liberty) is a fast, dynamic, and easy-to-use Java application server, based on the open-source Open Liberty project. It provides a flexible, fit-for-purpose, enterprise grade runtime for Java EE and Spring applications.
