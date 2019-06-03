---

copyright:
  years: 2019
lastupdated: "2019-05-20"

keywords: open liberty java, websphere liberty java, jakarta, webpshere docker, liberty docker, liberty docker images, installutility, microprofile java, dual layer docker, develop microservices

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

# Open Liberty 及 WebSphere Liberty
{: #liberty}

[Open Liberty](https://openliberty.io/){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示") 是一個利用模組*特性* 建置的輕量型開放程式碼 Java&trade; 運行環境。[WebSphere Liberty](https://developer.ibm.com/wasdev/){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示") 是商業版本的 Open Liberty。 

Liberty 特性支援下列應用程式架構：

* [MicroProfile](https://microprofile.io/){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示") 是一個開放程式碼專案，定義新標準及 API 以加速及簡化微服務的建立作業。
* [Jakarta EE](https://jakarta.ee){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示") 及 [Java EE](https://www.oracle.com/technetwork/java/javaee/overview/index.html){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")，包括個別規格的特性，例如，JNDI 或 JAX-RS。
* [SpringFramework 及 Spring Boot](https://www.ibm.com/support/knowledgecenter/en/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/twlp_dep_springboot.html){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")，包括從 Spring Boot 的 fat Jar 製作壓縮容器的機制。

有許多實用的開發人員手冊，可用於使用 Liberty 來建立微服務及雲端原生應用程式，其位於 [https://openliberty.io/guides/](https://openliberty.io/guides/){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")。

## 針對 Docker 進行最佳化
{: #liberty-optimized}

當 Kubernetes 等自動化系統推動容器映像檔時，映像檔大小開始變得重要。會快取 Docker 映像檔中的層，以協助處理此問題。如果提供其模組架構，Liberty 會為 Docker 容器提供有效的包裝管線，使其成為適合雲端環境的絕佳作業。一個基礎映像檔可用來支援許多工作負載，這些工作負載可容許執行容器的資源覆蓋區根據服務需求而改變。

Liberty 提供工具及最佳化支援，以將 Spring Boot fat Jar 轉換成已最佳化的壓縮 Docker 容器，利用快取的 Docker 映像檔層來改善循環和發佈時間。透過將不常變更的程式庫相依關係往下推至個別層，並僅保留最上層中的應用程式類別，可使反覆重建及重新部署的作業快很多。如需相關資訊，請參閱[為 Spring Boot 應用程式建立雙層 Docker 映像檔](https://openliberty.io/blog/2018/07/02/creating-dual-layer-docker-images-for-spring-boot-apps.html){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")。

## Liberty Docker 映像檔
{: #liberty-images}

Open Liberty 及 WebSphere Liberty 都提供各種基礎映像檔，可用來當作以 Java 為基礎之微服務的起點。有些映像檔使用小型覆蓋區，或使用已預先選取不同特性的 OpenJ9 VM。例如：

1. `open-liberty:microProfile2` 或 `websphere-liberty:microProfile2`
2. `open-liberty:javaee8` 或 `websphere-liberty:javaee8`
3. `open-liberty:springBoot2` 或 `websphere-liberty:springBoot2`

請在 Docker Hub 上檢查 [`websphere-liberty`](https://hub.docker.com/_/websphere-liberty/){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示") 或 [`open-liberty`](https://hub.docker.com/_/open-liberty/){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")，以取得最新的基礎映像檔清單。

### Open Liberty 及 Docker
{: #openliberty-docker}

從 [Docker Hub 上的映像檔清單](https://hub.docker.com/_/open-liberty/){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示") 中，為您的應用程式選擇最適當的映像檔（例如，`open-liberty:microProfile`），並為您的服務建立一個 Dockerfile，其 `FROM` 該映像檔延伸而來。新增指令，以將應用程式及伺服器配置複製到新的映像檔。範例如下：

```docker
FROM open-liberty:microProfile\
COPY server.xml /config/\
ADD Sample1.war /config/dropins/
```
{: codeblock}

如需其他範例及作用中程式碼，請參閱下列 Open Liberty 手冊：

* [Using Docker containers to develop microservices](https://openliberty.io/guides/docker.html){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")
* [Running an application in a Docker container](https://openliberty.io/guides/getting-started.html#running-the-application-in-a-docker-container){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")

### WebSphere Liberty 及 Docker
{: #wlp-docker}

Open Liberty 與商業版本 WebSphere Liberty 之間存在一些差異。對於建立 Docker 映像檔而言最重要的差異之一，就是 `installUtility` 指令不適用於 Open Liberty。

對於 Docker 映像檔，WebSphere Liberty 所支援的基本自訂作業模式與 Open Liberty 相同，但是 Liberty 原有的模組設計讓建立自訂映像檔的作業變得簡單（且一般），自訂映像檔包含應用程式及其所需的一組特定特性。WebSphere Liberty 有一個適用於無特性核心的映像檔 [`websphere-liberty:kernel`](https://github.com/WASdev/ci.docker/blob/9d28dfba4d20596f89b393bc9e3ae8295feec469/ga/developer/kernel/Dockerfile){: new_window}![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")，其為真正的自訂映像檔奠定了札實的基礎。

[`websphere-liberty` 映像檔文件](https://hub.docker.com/_/websphere-liberty/){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示") 說明建立自訂映像檔時所需的下列簡單 3 行 Dockerfile：

```docker
FROM websphere-liberty:kernel
COPY server.xml /config/
RUN installUtility install --acceptLicense defaultServer
```
{: codeblock}

`installUtility` 工具會在您的 `server.xml` 檔中，尋找您需要但 Liberty 映像檔中尚未提供的任何特性。接著會下載那些特性，並將它們安裝到您的 Docker 映像檔中。此方法將建立最小映像檔，但 `server.xml` 中的隱含特性清單不適用於快取的 Docker 層。`server.xml` 的任何變更都會使具備已安裝特性的層失效，需要在下次建置映像檔時，重新安裝遺漏的特性。

建立自訂映像檔的更好方式是利用特性的明確清單來呼叫 `installUtility`。這個方法仍會建立最小映像檔，但在建置時期會變得更好，因為映像檔的層可以快取並重複使用，而不管伺服器配置的變更。例如，下列範例會使用部分特性來建立自訂映像檔，以支援使用 JAX-RS、JNDI 及 WebSocket 的應用程式：

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

Docker 映像檔的結構取決於一些因素：

1. 您要如何重複使用或自訂您的基礎映像檔？
    這些步驟會產生可能最小的映像檔，如果每個應用程式的特性不同，則可能會重複使用。但是，具有小型自訂映像檔表示可以透過 Docker 主機快速部署應用程式。如果您不確定，請從 Docker Hub 開始使用其中一個預先產生的映像檔，讓重複使用性達到最高。
2. 您多久一次更新此映像檔？
    如果應用程式經常更新，例如在 CI/CD 管線中，則建構映像檔是有益的，以便最常變更的層（通常是應用程式類別或應用程式二進位檔）位於最頂層。此結構可減少在應用程式變更並重建映像檔時需要重建的層數目，從而加快建置和部署時間。

如果不確定，請遵循 [Open Liberty Docker 手冊](https://openliberty.io/guides/docker.html){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")，以開始進行。
