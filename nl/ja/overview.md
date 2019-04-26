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

# {{site.data.keyword.cloud_notm}} 上の Java
{: #overview}

Java&trade; アプリケーションにはさまざまな形やサイズがあり、スタンドアロン・アプリケーションから Java EE / Jakarta EE、MicroProfile、Spring アプリケーションに至るまで、あらゆるものがあります。

## OpenJDK + Open J9
{: #openjdk}

OpenJDK&trade; プロジェクトは Java 言語のオープン・ソースの参照実装で、ユーザー・プログラムに API 安定性と互換性を提供します。[Java SE プラットフォーム](https://docs.oracle.com/javase/8/docs/){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン") は、API と Java 仮想マシン (JVM) で構成されます。JVM は Java アプリケーションの実行を担当する実行エンジンです。

[Eclipse OpenJ9 プロジェクト](https://www.eclipse.org/openj9/index.html){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン") は、OpenJDK 内の Hotspot JVM に代わる代替 JVM を提供します。OpenJ9 JVM は 2017 年 9 月、{{site.data.keyword.IBM}} によってオープン・ソース化されましたが、その系統はかなり以前に遡ります。OpenJ9 はこの 10 年間にわたって {{site.data.keyword.appserver_short}} を含む 3000 以上の{{site.data.keyword.IBM}} 製品に実装されてきました。

OpenJDK と Eclipse OpenJ9 を併用することで、従来の Hotspot ベースの Java ランタイムを超えるクラウド・ネイティブ Java アプリケーションのパフォーマンスと保守容易性の向上が実現します。

### OpenJ9 付属の OpenJDK のダウンロード
{: #downloads}

OpenJ9 バイナリーが付いた OpenJDK は、Java コミュニティー・ハブ ([AdoptOpenJDK](https://adoptopenjdk.net/releases.html?variant=openjdk8&jvmVariant=openj9){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン")) から無料で入手できます。AdoptOpenJDK は、ビルド・スクリプトおよびインフラストラクチャーの完全なオープン・ソース・セットの事前構築された OpenJDK バイナリーを提供しています。Docker イメージも [Dockerhub](https://hub.docker.com/u/adoptopenjdk){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン") で入手できます。

## Java フレームワーク
{: #frameworks}

OpenJDK と OpenJ9 はすべての Java アプリケーションに対して強固な基盤を提供しますが、一般的な問題に対応する場合には、しばしばアプリケーション・フレームワークが使用されます。例えばエンタープライズ・アプリケーションは通常、[Java EE (および Jakarta EE)](/docs/java?topic=java-jee-overview#jakarta-ee) または [Spring](/docs/java?topic=java-spring-overview) のいずれかを使用して構築されます。[MicroProfile](/docs/java?topic=java-jee-overview#microprofile) は、クラウド・ネイティブ・コンセプトのより速い開発テンポとサポートを Java EE にもたらします。

[{{site.data.keyword.runtime_liberty_full}}](/docs/java?topic=java-liberty)は、使いやすく、高速かつ動的な Java アプリケーション・サーバーであり、オープン・ソースの Open Liberty プロジェクトを土台にしています。Java EE および Spring アプリケーションに、柔軟で目的に合った、エンタープライズ・レベルのランタイムを提供します。
