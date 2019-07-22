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

# 入门教程
{: #getting-started}

Java&trade; 应用程序具有各种形状和大小，从独立应用程序到 Java&trade; EE/Jakarta EE、MicroProfile 或 Spring 应用程序，应有尽有。

## OpenJDK（包含 OpenJ9）
{: #openjdk}

OpenJDK&trade; 项目是 Java&trade; 语言的开放式源代码参考实现，提供了 API 稳定性以及与用户程序的兼容性。[Java&trade; SE 平台](https://docs.oracle.com/javase/8/docs/){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标") 由 API 和 Java&trade; 虚拟机 (JVM) 组成。JVM 是负责执行 Java&trade; 应用程序的执行引擎。

[Eclipse OpenJ9 项目](https://www.eclipse.org/openj9/index.html){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标") 提供了替代 JVM，用于替换 OpenJDK 中的热点 JVM。2017 年 9 月，{{site.data.keyword.IBM}} 开放了 OpenJ9 JVM 的源代码，不过其发展谱系可追溯到比这早得多的日期。10 多年来，OpenJ9 为 3000 多种 {{site.data.keyword.IBM}} 产品提供支持，包括 {{site.data.keyword.appserver_short}}。

OpenJDK 与 Eclipse OpenJ9 一起使用，为云本机 Java&trade; 应用程序提供的性能和可维护性远超过基于热点的传统 Java&trade; 运行时。

### 下载 OpenJDK（包含 OpenJ9）
{: #downloads}

OpenJDK（包含 OpenJ9）二进制文件可从 [AdoptOpenJDK](https://adoptopenjdk.net/releases.html?variant=openjdk8&jvmVariant=openj9){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标") 上的 Java&trade; 社区中心免费获取。AdoptOpenJDK 通过一组完全开放源代码的构建脚本和基础架构提供预构建的 OpenJDK 二进制文件。Docker 映像还可在 [Docker Hub](https://hub.docker.com/u/adoptopenjdk){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标") 上获取。

## Java 框架
{: #frameworks}

OpenJDK 和 OpenJ9 可为任何 Java&trade; 应用程序提供坚实的基础，但应用程序框架通常用于解决常见问题。例如，企业应用程序通常使用 [Java&trade; EE（和 Jakarta EE）](/docs/java?topic=java-jee-overview#jakarta-ee)或 [Spring](/docs/java?topic=java-spring-overview) 进行构建。[MicroProfile](/docs/java?topic=java-jee-overview#microprofile) 为 Java&trade; EE 提供了更快的开发速度和对云本机概念的支持。

[{{site.data.keyword.runtime_liberty_full}}](/docs/java?topic=java-liberty) 是一种基于开放式源代码 Open Liberty 项目的动态 Java&trade; 应用程序服务器，运行速度快，使用简单。它为 Java&trade; EE 和 Spring 应用程序提供了灵活、量身打造的企业级运行时。
