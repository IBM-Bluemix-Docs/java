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

# 概説
{: #jee-overview}

## Java EE と Jakarta EE
{: #jakarta-ee}

Java&trade; EE7 および Java&trade; EE8 で導入されたテクノロジーではアノテーション、軽量のプロトコル、および非同期対話パターンがサポートされているため、Java でマイクロサービスを作成するのに最適です。 Java&trade; EE7 の並行性ユーティリティーには、RxJava などのリアクティブ・ライブラリーを、コンテナーで扱いやすい方法で使用するための直接的なメカニズムが備えられています。

2017 年 9 月、{{site.data.keyword.IBM}} と Red Hat のサポートを受けている Oracle は、Java&trade; EE が Jakarta EE として Eclipse Foundation に移行されることを発表しました。 Oracle は、Java&trade; EE 8 までの Java&trade; EE に関連するすべてのものを引き続き所有しますが、Java&trade; EE 8 の後の Enterprise Java&trade; プラットフォームの将来の開発はすべて、Jakarta EE ワーキング・グループの指導の下、Eclipse Foundation で行われます。2018 年初頭、Oracle は、API、RI、TCK、およびそれに関連するプロジェクト文書を含む Java&trade; EE テクノロジーのライセンス再交付と EE4J プロジェクトへの寄与という大きな取り組みを開始しました。

## MicroProfile
{: #microprofile}

Java&trade; EE は、マイクロサービスを作成するための強固な基盤を提供しますが、マイクロサービス・アプリケーションにより適したテクノロジーおよびプログラミング・モデルが必要でした。 IBM と他の企業が協力して、開発者、コミュニティー、およびベンダー間のオープンなコラボレーションである MicroProfile を立ち上げました。

MicroProfile コミュニティーは、マイクロサービスおよび Enterprise Java&trade; に関する迅速なイノベーションに重点を置いています。 このコミュニティーは、マイクロサービスのアーキテクチャー・パターンに従うクラウド・ネイティブ・アプリに最適なテクノロジーを構築および統合します。 MicroProfile の各リリースには、テクノロジーのセットが含まれています。 2016 年にリリースされた MicroProfile 1.0 には、軽量のマイクロサービス用の一連のコア機能を提供する Java&trade; EE 7 の仕様のサブセット (CDI 1.2、JAX-RS 2.0、および JSON-P 1.0) が含まれていました。 MicroProfile プラットフォームには、構成の注入、フォールト・トレランス、ヘルス・チェック、メトリック、オープン・トレースなど、クラウド特有の懸念事項に対処するためのテクノロジーが導入されています。 また、OpenAPI 定義を使用して REST API を簡単に使用できるようにする REST クライアントを作成したり、アプリのセキュリティー問題に対処するために JWT を伝搬したりするための、ベンダーに依存しない規格も定義しています。

このオープン・ソース・グループに参加し始めるには、[https://microprofile.io/](https://microprofile.io/){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン") または [https://projects.eclipse.org/projects/technology.microprofile](https://projects.eclipse.org/projects/technology.microprofile){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン") にアクセスしてください。
