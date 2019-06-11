---

copyright:
  years: 2019
lastupdated: "2019-06-10"

keywords: openjdk, open j9, download openjdk, java frameworks, adoptopenjdk, eclipse openj9, openj9 binaries, openjdk binaries, microprofile framework, jakarta

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

# Getting started tutorial
{: #getting-started}

Java&trade; applications come in all shapes and sizes, everything from stand-alone apps to Java&trade; EE / Jakarta EE, MicroProfile, or Spring apps.

## OpenJDK with Open J9
{: #openjdk}

The OpenJDK&trade; project is the open source reference implementation of the Java&trade; language and provides API stability and compatibility to user programs. The [Java&trade; SE platform](https://docs.oracle.com/javase/8/docs/){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon"), consists of APIs and the Java&trade; virtual machine (JVM). The JVM is the execution engine responsible for running the Java&trade; app.

The [Eclipse OpenJ9 project](https://www.eclipse.org/openj9/index.html){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon") provides an alternative JVM that replaces the Hotspot JVM in OpenJDK. The OpenJ9 JVM was open-sourced by {{site.data.keyword.IBM}} in September of 2017, though its lineage goes back much further. OpenJ9 powers over 3000 {{site.data.keyword.IBM}} products, including {{site.data.keyword.appserver_short}}, for over a decade.

Together, OpenJDK and Eclipse OpenJ9 provide improved performance and serviceability for cloud-native Java&trade; apps above and beyond traditional Hotspot-based Java&trade; runtimes.

### Downloading OpenJDK with OpenJ9
{: #downloads}

OpenJDK with OpenJ9 binaries are freely available from the Java&trade; community hub at [AdoptOpenJDK](https://adoptopenjdk.net/releases.html?variant=openjdk8&jvmVariant=openj9){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon"). AdoptOpenJDK provides prebuilt OpenJDK binaries from a fully open source set of build scripts and infrastructure. Docker images are also available on [Docker hub](https://hub.docker.com/u/adoptopenjdk){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon").

## Java frameworks
{: #frameworks}

OpenJDK and OpenJ9 provide a solid foundation for any Java&trade; app, but app frameworks are often used to address common concerns. Enterprise apps, for example, are typically built by using either [Java&trade; EE (and Jakarta EE)](/docs/java?topic=java-jee-overview#jakarta-ee), or [Spring](/docs/java?topic=java-spring-overview). [MicroProfile](/docs/java?topic=java-jee-overview#microprofile) brings a faster development pace and support for cloud-native concepts to Java&trade; EE.

[{{site.data.keyword.runtime_liberty_full}}](/docs/java?topic=java-liberty) is a fast, dynamic, and easy-to-use Java&trade; application server, based on the open source Open Liberty project. It provides a flexible, fit-for-purpose, enterprise grade runtime for Java&trade; EE and Spring apps.
