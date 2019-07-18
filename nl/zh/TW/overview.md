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

# 入門指導教學
{: #getting-started}

Java&trade; 應用程式有各種形態和大小，從獨立式應用程式到 Java&trade; EE/Jakarta EE、MicroProfile 或 Spring 應用程式。

## 具有 OpenJ9 的 OpenJDK
{: #openjdk}

OpenJDK&trade; 專案是 Java&trade; 語言的開放程式碼參考實作，提供了 API 穩定性以及與使用者程式的相容性。[Java&trade; SE 平台](https://docs.oracle.com/javase/8/docs/){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示") 由 API 和 Java&trade; 虛擬機器 (JVM) 組成。JVM 是負責執行 Java&trade; 應用程式的執行引擎。

[Eclipse OpenJ9 專案](https://www.eclipse.org/openj9/index.html){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示") 提供替代的 JVM，取代 OpenJDK 中的 Hotspot JVM。2017 年 9 月，{{site.data.keyword.IBM}} 開放了 OpenJ9 JVM 的開放程式碼，不過其發展譜系可追溯到比這早得多的日期。10 多年來，OpenJ9 為 3000 多種 {{site.data.keyword.IBM}} 產品提供支援，包括 {{site.data.keyword.appserver_short}}。

OpenJDK 與 Eclipse OpenJ9 一起使用，為雲端原生 Java&trade;應用程式提供的效能和服務功能遠超過熱點型的傳統 Java&trade; 運行環境。

### 下載具有 OpenJ9 的 OpenJDK
{: #downloads}

OpenJDK（包含 OpenJ9）二進位檔可從 [AdoptOpenJDK](https://adoptopenjdk.net/releases.html?variant=openjdk8&jvmVariant=openj9){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示") 上的 Java&trade; 社群中心免費獲取。AdoptOpenJDK 從一組完全開放程式碼的建置 Script 及基礎架構中，提供預先建置的 OpenJDK 二進位檔。Docker 映像檔還可在 [Docker Hub](https://hub.docker.com/u/adoptopenjdk){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示") 上取得。

## Java 架構
{: #frameworks}

OpenJDK 和 OpenJ9 可為任何 Java&trade; 應用程式提供堅固的基礎，但應用程式架構通常用於解決常見問題。例如，企業應用程式通常使用 [Java&trade; EE（和 Jakarta EE）](/docs/java?topic=java-jee-overview#jakarta-ee)或 [Spring](/docs/java?topic=java-spring-overview) 進行建置。[MicroProfile](/docs/java?topic=java-jee-overview#microprofile) 為 Java&trade; EE 提供了更快的開發速度和對雲端原生概念的支援。

[{{site.data.keyword.runtime_liberty_full}}](/docs/java?topic=java-liberty) 是一種基於開放程式碼 Open Liberty 專案的動態 Java&trade; 應用程式伺服器，執行速度快，使用簡單。它為 Java&trade; EE 和 Spring應用程式提供了靈活、量身打造的企業級運行環境。
