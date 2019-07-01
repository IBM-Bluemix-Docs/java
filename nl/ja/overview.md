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

# 入門チュートリアル
{: #getting-started}

Java&trade; アプリにはさまざまな形やサイズがあり、スタンドアロン・アプリから Java&trade; EE / Jakarta EE、MicroProfile、Spring アプリに至るまで、あらゆるものがあります。

## OpenJDK + Open J9
{: #openjdk}

OpenJDK&trade; プロジェクトは Java&trade; 言語のオープン・ソースの参照実装で、ユーザー・プログラムに API 安定性と互換性を提供します。[Java&trade; SE プラットフォーム](https://docs.oracle.com/javase/8/docs/){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン") は、API と Java&trade; 仮想マシン (JVM) で構成されます。JVM は Java&trade; アプリの実行を担当する実行エンジンです。

[Eclipse OpenJ9 プロジェクト](https://www.eclipse.org/openj9/index.html){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン") は、OpenJDK 内の Hotspot JVM に代わる代替 JVM を提供します。 OpenJ9 JVM は 2017 年 9 月に {{site.data.keyword.IBM}} によってオープン・ソース化されましたが、その系統はさらに遡ります。OpenJ9 は 10 年以上にわたって、{{site.data.keyword.appserver_short}} を含む 3000 以上の {{site.data.keyword.IBM}} 製品に実装されています。

OpenJDK と Eclipse OpenJ9 を併用することで、従来の Hotspot ベースの Java&trade; ランタイムを超えるクラウド・ネイティブ Java&trade; アプリのパフォーマンスと保守容易性の向上が実現します。

### OpenJ9 付属の OpenJDK のダウンロード
{: #downloads}

OpenJ9 バイナリーが付いた OpenJDK は、Java&trade; コミュニティー・ハブ ([AdoptOpenJDK](https://adoptopenjdk.net/releases.html?variant=openjdk8&jvmVariant=openj9){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン")) から無料で入手できます。 AdoptOpenJDK は、ビルド・スクリプトおよびインフラストラクチャーの完全なオープン・ソース・セットの事前構築された OpenJDK バイナリーを提供しています。 Docker イメージも [Docker Hub](https://hub.docker.com/u/adoptopenjdk){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン") で入手できます。

## Java フレームワーク
{: #frameworks}

OpenJDK と OpenJ9 はすべての Java&trade; アプリに対して強固な基盤を提供しますが、一般的な問題に対応する場合には、しばしばアプリ・フレームワークが使用されます。 例えばエンタープライズ・アプリは通常、[Java&trade; EE (および Jakarta EE)](/docs/java?topic=java-jee-overview#jakarta-ee) または [Spring](/docs/java?topic=java-spring-overview) のいずれかを使用して構築されます。 [MicroProfile](/docs/java?topic=java-jee-overview#microprofile) は、クラウド・ネイティブ・コンセプトのより速い開発テンポとサポートを Java&trade; EE にもたらします。

[{{site.data.keyword.runtime_liberty_full}}](/docs/java?topic=java-liberty)は、使いやすく、高速かつ動的な Java&trade; アプリケーション・サーバーであり、オープン・ソースの Open Liberty プロジェクトを土台にしています。 Java&trade; EE および Spring アプリに、柔軟で目的に合った、エンタープライズ・レベルのランタイムを提供します。
