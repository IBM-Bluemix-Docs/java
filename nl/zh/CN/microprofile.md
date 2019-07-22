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

# 概述
{: #jee-overview}

## Java EE 和 Jakarta EE
{: #jakarta-ee}

Java&trade; EE7 和 EE8 中引入的技术非常适合在 Java&trade; 中创建微服务，并支持注释、轻量级协议和异步交互模式。Java&trade; EE7 中的并行实用程序提供了一种简单的机制，能以容器友好方式使用 RxJava 等反应式库。

2017 年 9 月，在 {{site.data.keyword.IBM}} 和 Red Hat 支持下，Oracle 宣布 Java&trade; EE 将移至 Eclipse Foundation，并重命名为 Jakarta EE。Oracle 将继续拥有与 Java&trade; EE（截至并包括 Java&trade; EE 8）关联的所有内容，但在 Java&trade; EE 8 之后此 Enterprise Java&trade; 平台的所有未来开发工作都将由 Eclipse Foundation 在 Jakarta EE 工作组的指导下处理。2018 年初，Oracle 开始投入大量工作，对 Java&trade; EE 技术（包括 API、RI、TCK 和关联的项目文档）进行重新许可并提供到 EE4J 项目中。

## MicroProfile
{: #microprofile}

虽然 Java&trade; EE 为创建微服务提供了坚实的基础，但它仍需要各种技术和编程模型来更好地适应微服务应用程序。IBM 和其他公司联合推出了 MicroProfile，这是开发者、社区与供应商之间的一个开放式协作项目。

MicroProfile 社区专注于围绕微服务和 Enterprise Java&trade; 进行快速创新。此社区将构建并集成最适用于遵循微服务体系结构模式的云本机应用程序的技术。每个 MicroProfile 发行版都包含一组技术。2016 年发布的 MicroProfile 1.0 包含来自 Java&trade; EE 7 的规范子集，为轻量级微服务提供了一组核心功能：CDI 1.2、JAX-RS 2.0 和 JSON-P 1.0。MicroProfile 平台包含解决独特云问题的各种技术，如配置注入、容错、运行状况检查、度量值和开放式跟踪。此外，该平台还定义了针对使用 OpenAPI 定义和创建 REST 客户机的与供应商无关的标准，能更轻松地使用 REST API 以及进行 JWT 传播来解决应用程序安全问题。

要开始参与开放式源代码组，请访问 [https://microprofile.io/](https://microprofile.io/){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标") 或 [https://projects.eclipse.org/projects/technology.microprofile](https://projects.eclipse.org/projects/technology.microprofile){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")。
