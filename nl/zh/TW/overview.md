---

copyright:
  years: 2019
lastupdated: "2019-04-01"

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

# Java on {{site.data.keyword.cloud_notm}}
{: #overview}

Java&trade; 應用程式有各種形態和大小，從獨立式應用程式到 Java EE /Jakarta EE、MicroProfile 或 Spring 應用程式。

## 具有 OpenJ9 的 OpenJDK
{: #openjdk}

OpenJDK&trade; 專案是 Java 語言的開放程式碼參照實作，提供 API 穩定性及使用者程式的相容性。[Java SE 平台](https://docs.oracle.com/javase/8/docs/){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示") 由 API 及 Java 虛擬機器 (JVM) 所組成。JVM 是負責執行 Java 應用程式的執行引擎。

[Eclipse OpenJ9 專案](https://www.eclipse.org/openj9/index.html){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示") 提供替代的 JVM，取代 OpenJDK 中的 Hotspot JVM。OpenJ9 JVM 由 {{site.data.keyword.IBM}} 於 2017 年 9 月開放程式碼，雖然其世系遠遠超過該日期。十多年來，OpenJ9 已為超過 3000 個 {{site.data.keyword.IBM}} 產品供電，包括 {{site.data.keyword.appserver_short}}。

OpenJDK 和 Eclipse OpenJ9 一起為雲端原生 Java 應用程式（超越傳統的 Hotspot 型 Java 運行環境），提供改良的效能和服務功能。

### 下載具有 OpenJ9 的 OpenJDK
{: #downloads}

可從 Java 社群中心（位於 [AdoptOpenJDK](https://adoptopenjdk.net/releases.html?variant=openjdk8&jvmVariant=openj9){: new_window}![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")）自由取得具有 OpenJ9 二進位檔的 OpenJDK。AdoptOpenJDK 從一組完全開放程式碼的建置 Script 及基礎架構中，提供預先建置的 OpenJDK 二進位檔。Docker 映像檔也可在 [Dockerhub](https://hub.docker.com/u/adoptopenjdk){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示") 上使用。

## Java 架構
{: #frameworks}

OpenJDK 及 OpenJ9 為任何 Java 應用程式提供了扎實的基礎，但應用程式架構通常用來解決一般問題。例如，通常利用 [Java EE（及 Jakarta EE）](/docs/java?topic=java-jee-overview#jakarta-ee)或 [Spring](/docs/java?topic=java-spring-overview) 來建置企業應用程式。[MicroProfile](/docs/java?topic=java-jee-overview#microprofile) 提供更快速的開發速度，並支援將雲端原生概念提供給 Java EE。

[{{site.data.keyword.runtime_liberty_full}}](/docs/java?topic=java-liberty) 是一個快速、動態且容易使用，並以開放程式碼 Open Liberty 專案為基礎的 Java 應用程式伺服器。它為 Java EE 和 Spring 應用程式提供彈性且符合用途的企業級運行環境。
