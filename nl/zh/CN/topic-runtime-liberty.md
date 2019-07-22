---

copyright:
  years: 2019
lastupdated: "2019-06-10"

keywords: open liberty java, websphere liberty java, jakarta, webpshere docker, liberty docker, liberty docker images, installutility, microprofile java, dual layer docker, develop microservices

subcollection: java

---

{:new_window: target="_blank"}
{:external: target="_blank" .external}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}

# Open Liberty 和 WebSphere Liberty
{: #liberty}

[Open Liberty](https://openliberty.io/){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标") 是一种使用模块化*功能*构建的轻量级开放式源代码 Java&trade; 运行时。[WebSphere Liberty](https://developer.ibm.com/wasdev/){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标") 是 Open Liberty 的商业版本。 

Liberty 功能支持以下应用程序框架：

* [MicroProfile](https://microprofile.io/){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")，是一个开放式源代码项目，用于定义新的标准和 API，以加速和简化微服务的创建。
* [Jakarta EE](https://jakarta.ee){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标") 和 [Java&trade; EE](https://www.oracle.com/technetwork/java/javaee/overview/index.html){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")，包含用于不同的规范（如 JNDI 或 JAX-RS）的功能。
* [Spring Framework 和 Spring Boot](https://www.ibm.com/support/knowledgecenter/en/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/twlp_dep_springboot.html){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")，包含用于通过 Spring Boot 的胖 .jar 创建紧凑容器的机制。

在 [https://openliberty.io/guides/](https://openliberty.io/guides/){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标") 上，有很多实用的开发者指南，用于指导如何使用 Liberty 创建微服务和云本机应用程序。

## 已针对 Docker 进行优化
{: #liberty-optimized}

Kubernetes 等自动化系统在各处推送容器映像时，映像大小会开始变得重要。Docker 映像中的层会进行高速缓存，以帮助解决此问题。Liberty 在给定其模块化体系结构的情况下，为 Docker 容器提供了高效的打包管道，使其非常适合在云环境中运行。一个基本映像可用于支持许多工作负载，这些工作负载允许运行容器的资源占用量根据服务需求而变化。

Liberty 提供了多种工具，并优化了对 Spring Boot 胖 .jar 转换为紧凑的优化 Docker 容器的支持，这些 Docker 容器利用高速缓存的 Docker 映像层来改进循环和发布时间。通过将很少更改的库依赖项向下推送到单独的层中，并仅保留顶层中的应用程序类，迭代重建和重新部署的速度会快得多。请参阅 [Creating Dual Layer Docker images for Spring Boot apps](https://openliberty.io/blog/2018/07/02/creating-dual-layer-docker-images-for-spring-boot-apps.html){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")。

## Liberty Docker 映像
{: #liberty-images}

Open Liberty 和 WebSphere Liberty 都提供了各种基本映像，可以将其用作基于 Java 的微服务的起点。有一些映像使用的占用量小，或使用预先选择了不同功能 OpenJ9 VM。例如：

1. `open-liberty:microProfile2` 或 `websphere-liberty:microProfile2`
2. `open-liberty:javaee8` 或 `websphere-liberty:javaee8`
3. `open-liberty:springBoot2` 或 `websphere-liberty:springBoot2`

请查看 Docker Hub 上的 [`websphere-liberty`](https://hub.docker.com/_/websphere-liberty/){: external} 或 [`open-liberty`](https://hub.docker.com/_/open-liberty/){: external}，以获取基本映像的最新列表。

### Open Liberty 和 Docker
{: #openliberty-docker}

从 [Docker Hub 上的映像列表](https://hub.docker.com/_/open-liberty/){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标") 中选择最适合应用程序的映像（例如，`open-liberty:microProfile`），并为服务创建通过 `FROM` 操作从该映像扩展的 Dockerfile。添加用于将应用程序和服务器配置复制到新映像中的命令。例如：

```docker
FROM open-liberty:microProfile\
COPY server.xml /config/\
ADD Sample1.war /config/dropins/
```
{: codeblock}

有关更多示例和有效代码，请参阅以下 Open Liberty 指南：

* [Using Docker containers to develop microservices](https://openliberty.io/guides/docker.html){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")
* [Running an application in a Docker container](https://openliberty.io/guides/getting-started.html#running-the-application-in-a-docker-container){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")

### WebSphere Liberty 和 Docker
{: #wlp-docker}

Open Liberty 与其商业版本 WebSphere Liberty 之间存在一些差异。在创建 Docker 映像方面，最重要的一个差异是 `installUtility` 命令在 Open Liberty 中不可用。

对于 Docker 映像，WebSphere Liberty 支持的基本定制模式与 Open Liberty 支持的相同。但是 Liberty 的固有模块化设计使得创建包含应用程序及其所需特定功能集的定制映像十分简单（且典型）。WebSphere Liberty 具有用于无功能内核的映像 [`websphere-liberty:kernel`](https://github.com/WASdev/ci.docker/blob/9d28dfba4d20596f89b393bc9e3ae8295feec469/ga/developer/kernel/Dockerfile){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")，这为真正定制的映像提供了坚实的基础。

[`websphere-liberty` 映像文档](https://hub.docker.com/_/websphere-liberty/){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标") 描述了以下简单的 Dockerfile，此文件包含创建定制映像所需的 3 行内容：

```docker
FROM websphere-liberty:kernel
COPY server.xml /config/
RUN installUtility install --acceptLicense defaultServer
```
{: codeblock}

`installUtility` 工具会在 `server.xml` 文件中查找您需要但在 Liberty 映像中尚不可用的任何功能。然后，此工具会下载这些功能，并将其安装到 Docker 映像中。此方法会创建极简映像，但 `server.xml` 中的隐式功能列表用于高速缓存的 Docker 层不太理想。对 `server.xml` 的任何更改都会使安装了这些功能的层失效，导致下次构建映像时，会要求再次安装缺少的功能。

创建定制映像的更好方法是使用显式功能列表来调用 `installUtility`。此方法仍会创建极简映像，但在构建时的行为要好得多，因为映像层可以进行高速缓存和复用，而与服务器配置更改无关。例如，以下示例会创建使用功能子集的定制映像，以支持使用 JAX-RS、JNDI 和 WebSocket 的应用程序：

```docker
FROM websphere-liberty:kernel
RUN /opt/ibm/wlp/bin/installUtility install --acceptLicense \
  cdi-1.2 \
  concurrent-1.0 \
  jaxrs-2.0 \
  jndi-1.0 \
  ssl-1.0 \
  websocket-1.1
COPY server.xml /config/
```
{: codeblock}

如何设计 Docker 映像结构取决于多个因素：

1. 您希望基本映像的可复用或定制程度如何？
    这些步骤会生成尽可能最小的映像，如果每个应用程序的功能不同，那么有可能会牺牲复用能力。但是，具有小的定制映像意味着应用程序可以在不同 Docker 主机上快速进行部署。如果您不确定，请首先使用 Docker Hub 中的其中一个预生成的映像，以获得最大的复用能力。
2. 此映像多久更新一次？
    如果应用程序频繁更新（例如，在 CI/CD 管道中），那么将映像结构调整为使更改最频繁的层（通常是应用程序类或应用程序二进制文件）位于最顶层，会非常有用。此结构可减少在更改应用程序和重建映像时需要重建的层数，从而缩短构建和部署时间。

如有疑问，请首先遵循 [Open Liberty Docker 指南](https://openliberty.io/guides/docker.html){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")。
