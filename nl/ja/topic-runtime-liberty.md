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

# Open Liberty と WebSphere Liberty
{: #liberty}

[Open Liberty](https://openliberty.io/){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン") は、モジュラー*機能*を使用して構築された、軽量のオープン・ソース Java&trade; ランタイムです。 [WebSphere Liberty](https://developer.ibm.com/wasdev/){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン") は、Open Liberty の商用バージョンです。 

Liberty の機能は以下のアプリケーション・フレームワークのためのサポートを提供します。

* [MicroProfile](https://microprofile.io/){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン")。マイクロサービスの作成を加速して簡単にするための新しい標準と API を定義するオープン・ソース・プロジェクトです。
* [Jakarta EE](https://jakarta.ee){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン") および [Java&trade; EE](https://www.oracle.com/technetwork/java/javaee/overview/index.html){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン")。JNDI や JAX-RS など、個別の仕様の機能を含みます。
* [Spring Framework と Spring Boot](https://www.ibm.com/support/knowledgecenter/en/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/twlp_dep_springboot.html){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン")。Spring Boot の大きな .jar からコンパクトなコンテナーを作成するためのメカニズムを含みます。

Liberty を使用してマイクロサービスおよびクラウド・ネイティブ・アプリを作成する方法に関する、多数の実用的なデベロッパー・ガイドが [https://openliberty.io/guides/](https://openliberty.io/guides/){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン") にあります。

## Docker のための最適化
{: #liberty-optimized}

Kubernetes などの自動化システムがコンテナー・イメージを各所にプッシュすると、イメージのサイズが影響を与え始めます。 この問題に対応するために、Docker イメージのレイヤーがキャッシュに入れられます。 Liberty は、そのモジュラー・アーキテクチャーに基づき、Docker コンテナー用の効果的なパッケージ・パイプラインを提供するので、クラウド環境での運用に最適なものとなっています。 1 つの基本イメージを使用して多数のワークロードをサポートし、サービスの要件に基づいて、実行中のコンテナーのリソース・フットプリントを変化させることが可能です。

Liberty には、Spring Boot の大きな .jar を、サイクルとパブリッシュ回数を改善するためにキャッシュに入れた Docker イメージ・レイヤーを利用する、コンパクトで最適化された Docker コンテナーに変換するためのツールと最適化サポートが用意されています。 変更される頻度の低いライブラリー依存関係を別個のレイヤーに移動し、最上部のレイヤーにはアプリ・クラスだけを保管することにより、再ビルドと再デプロイをより速く繰り返すことができます。 詳しくは、[Creating Dual Layer Docker images for Spring Boot apps](https://openliberty.io/blog/2018/07/02/creating-dual-layer-docker-images-for-spring-boot-apps.html){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン") を参照してください。

## Liberty Docker のイメージ
{: #liberty-images}

Open Liberty と WebSphere Liberty の両方には、Java ベースのマイクロサービスの開始点として使用できる、さまざまな基本イメージが用意されています。 さまざまな機能が事前選択された、小さなフットプリントまたは OpenJ9 VM を使用するイメージもあります。 例えば次のようなものです。

1. `open-liberty:microProfile2` または `websphere-liberty:microProfile2`
2. `open-liberty:javaee8` または `websphere-liberty:javaee8`
3. `open-liberty:springBoot2` または `websphere-liberty:springBoot2`

基本イメージの最新のリストは、Docker Hub の [`websphere-liberty`](https://hub.docker.com/_/websphere-liberty/){: external} または [`open-liberty`](https://hub.docker.com/_/open-liberty/){: external} を参照してください。

### Open Liberty と Docker
{: #openliberty-docker}

[Docker Hub のイメージのリスト](https://hub.docker.com/_/open-liberty/){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン") からアプリに最も該当するイメージ (例えば `open-liberty:microProfile` など) を選択し、それを拡張元 (`FROM`) とするサービス用の Dockerfile を作成します。 アプリとサーバー構成を新しいイメージにコピーするコマンドを追加します。 以下に例を示します。

```docker
FROM open-liberty:microProfile\
COPY server.xml /config/\
ADD Sample1.war /config/dropins/
```
{: codeblock}

以下の Open Liberty のガイドには、さらに多くの例と実用コードが記載されています。

* [Using Docker containers to develop microservices](https://openliberty.io/guides/docker.html){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン")
* [Running an application in a Docker container](https://openliberty.io/guides/getting-started.html#running-the-application-in-a-docker-container){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン")

### WebSphere Liberty と Docker
{: #wlp-docker}

Open Liberty とその商用バージョンである WebSphere Liberty の間のいくつかの相違点を以下に示します。 Docker イメージの作成に関連した最重要な相違点の 1 つとして、`installUtility` コマンドは Open Liberty では使用できません。

WebSphere Liberty は、Open Liberty が Docker イメージ用にサポートするものと同じ基本カスタマイズ・パターンをサポートします。 ただし、Liberty は本質的にモジュラー設計であるため、アプリケーションとそれに必要な特定の機能セットを含むカスタム・イメージの作成が簡単 (および標準的) になります。 WebSphere Liberty には、機能の少ないカーネル [`websphere-liberty:kernel`](https://github.com/WASdev/ci.docker/blob/9d28dfba4d20596f89b393bc9e3ae8295feec469/ga/developer/kernel/Dockerfile){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン") のためのイメージがあり、これは本当のカスタマイズ・イメージのための確かな基礎となります。

[`websphere-liberty` イメージの資料](https://hub.docker.com/_/websphere-liberty/){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン") には、カスタム・イメージの作成に必要な、以下の簡単な 3 行の Dockerfile が記載されています。

```docker
FROM websphere-liberty:kernel
COPY server.xml /config/
RUN installUtility install --acceptLicense defaultServer
```
{: codeblock}

`installUtility` ツールは、Liberty イメージではまだ使用できない、`server.xml` ファイルで必要な機能を検索します。 その後、それらの機能をダウンロードして Docker イメージにインストールします。 この方式により作成されるイメージは最小になりますが、`server.xml` の暗黙機能リストは、キャッシュに入れられた Docker レイヤーでは正常に動作しません。 `server.xml` に対する変更により、機能のインストールされたレイヤーは無効になり、次にイメージをビルドするときに欠落している機能を再びインストールすることが必要になります。

カスタム・イメージを作成するためのより良い方法は、`installUtility` に機能の明示的なリストを指定して起動することです。 この方法により作成されるイメージも最小になりますが、サーバー構成の変更とは関係なくイメージのレイヤーをキャッシュに入れて再使用することができるため、ビルド時の動作は一段と優れたものになります。 例えば、以下の例は、JAX-RS、JNDI、WebSockets を使用するアプリをサポートするために、機能のサブセットを使用してカスタム・イメージを作成します。

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

いくつかの要素に応じて Docker イメージを構成する方法:

1. 基本イメージをどの程度再使用可能にしたりカスタマイズしたりしますか?
    これらの手順により、可能な限り最小のイメージが生成されますが、その犠牲として、各アプリの機能が異なっている場合には再使用できない可能性があります。しかし、カスタム・イメージが小さいと、アプリは複数の Docker ホスト上に素早くデプロイされます。 確信のない場合には、より多く再使用するために、Docker Hub にあるいずれかの事前生成されたイメージを使用して開始してください。
2. このイメージはどの程度の頻度で更新しますか?
    アプリが頻繁に更新される場合 (例えば CI/CD パイプラインにある場合など)、最も頻繁に変更されるレイヤー (通常はアプリ・クラスまたはアプリ・バイナリー) が最上部のレイヤーになるようにイメージを構成すると便利です。 この構成により、アプリが変更されてイメージが再ビルドされる際に再ビルドする必要のあるレイヤーの数は少なくなり、ビルドとデプロイメントの時間が短縮されます。

判別が難しい場合には、[Open Liberty Docker guide](https://openliberty.io/guides/docker.html){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン") に従って開始してください。
