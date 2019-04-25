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

# 概觀
{: #jee-overview}



## Java EE 及 Jakarta EE
{: #jakarta-ee}

Java EE7 及 EE8 中引進的技術非常適合以 Java 建立微服務，並支援註釋、小型通訊協定及非同步互動型樣。Java EE7 中的並行公用程式提供直接明確的機制，可用容器友善的方式來使用 RxJava 等回應程式庫。

在 2017 年 9 月，IBM 及 Red Hat 所支援的 Oracle 發表 Java EE 即將移至 Eclipse Foundation，作為 Jakarta EE。Oracle 繼續擁有與 Java EE 相關聯的所有內容，包含 Java EE 8 以及以下版本，但在 Java EE 8 之後，這個 Enterprise Java 平台的所有未來發展，在 Jakarta EE 工作群組的指引下，都將位於 Eclipse Foundation。在 2018 年初，Oracle 便已開始大量投入重新授權的作業，並將 Java EE 技術，包括 API、RI、TCK 及相關聯的專案文件，提供給 EE4J 專案。

## MicroProfile
{: #microprofile}

儘管 Java EE 可提供扎實基礎來建立微服務，它仍需要一些技術和程式設計模型才能更適應微服務應用程式。IBM 與其他公司合力推動了 MicroProfile，這是開發人員、社群與供應商之間的開放合作。

MicroProfile 社群著重於微服務與 Enterprise Java 的快速創新。這個社群針對遵循微服務架構型樣的雲端原生應用程式，建置並整合了最適合該應用程式的技術。每個 MicroProfile 版本都包含一組技術。MicroProfile 1.0（2016 年發行）包含 Java EE 7 的規格子集，提供一組用於輕量型微服務的核心功能：CDI 1.2、JAX-RS 2.0 及 JSON-P 1.0。MicroProfile 平台包括可解決獨特雲端問題的技術，例如，配置注入、容錯、性能檢查、度量、開啟追蹤。它還定義了供應商中立的標準，用於使用 OpenAPI 定義及建立 REST 用戶端，以便可以更輕鬆地使用 REST API，以及用於 JWT 傳播以解決應用程式安全問題。

若要開始參與開放程式碼群組，請造訪 [https://microprofile.io/](https://microprofile.io/){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示") 或 [https://projects.eclipse.org/projects/technology.microprofile](https://projects.eclipse.org/projects/technology.microprofile){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")。
