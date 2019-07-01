---

copyright:
  years: 2019
lastupdated: "2019-06-10"

keywords: spring framework, spring reference, spring boot, boot actuator, spring kubernetes

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

# Spring
{: #spring-overview}

Spring プラットフォームは Java&trade; アプリケーション作成の単純化を目指したプロジェクトのエコシステムです。 通常、「Spring」という言葉は Spring Framework を指しますが、Spring ベースのテクノロジーの一環である、あるいは Spring Boot を含むそうしたテクノロジーを使用する、あらゆるプロジェクトの総称でもあります。

## Spring Framework
{: #spring-framework}

Spring Framework は 2002 年に導入され、約 3 年ごとにメジャー・バージョンがリリースされています。このフレームワークは、REST エンドポイントからデータベース抽象化までのあらゆるものを対象とした、大規模なコンポーネント・セット (Spring モジュールと呼ばれる) で構成されます。 2017 年にリリースされた最新の Spring Framework 5 では、更新された JDK サポート、および WebFlux が導入されました。WebFlux は、[Project Reactor](https://projectreactor.io/){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン") に基づくリアクティブ・プログラミングの新しいモジュールです。

Spring Framework 4.3 は Spring Framework 4.x 用の最新のフィーチャー・ブランチで、サポートは 2020 年まで続きます。 古いバージョンの Spring を使用するアプリケーションは、Spring Framework 5 にマイグレーションする必要があります。

Spring Framework の総合的な資料については、こちらをご覧ください。

* [Spring Framework Reference Guide for 5.1.x](https://docs.spring.io/spring/docs/5.1.x/spring-framework-reference/){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン")
* [Spring Framework Reference Guide for 4.3.x](https://docs.spring.io/spring/docs/4.3.x/spring-framework-reference/){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン")
* [Upgrading to Spring Framework 5.x](https://github.com/spring-projects/spring-framework/wiki/Upgrading-to-Spring-Framework-5.x){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン")

## Spring Boot
{: #spring-boot}

Spring Boot は、「コンテナーなしの Web アプリケーション・アーキテクチャーを改善する」ために 2014 年に導入されました。 その目標は、構成を XML からコードに移行することでした。 Spring Boot は、どんなテクノロジーを使用するかに関する自説に基づいて、アプリを作成するためのメカニズムを提供します。 Spring Boot アプリは Spring アプリであり、そのエコシステムに固有のプログラミング・モデルを使用します。

Spring Boot は構成よりも規則を重視し、アノテーションとクラスパス検出の組み合わせを使用して機能を可能にします。 デフォルトで、Spring Boot アプリケーションは、必要なすべての依存関係を含む (組み込みの Tomcat サーバーも含む) 自己完結型で実行可能な JAR ファイルに組み込まれます。 または、アプリ・サーバーにデプロイされる war ファイルとしてアプリをパッケージ化することができます。

Spring Boot の現在のバージョンは 2018 年にリリースされたバージョン 2.0 です。 1.5.x ブランチの保守は 2019 年 8 月に終了します。

Spring Boot の総合的な資料については、こちらをご覧ください。

* [Spring Boot Reference Guide for 2.1.x](https://docs.spring.io/spring-boot/docs/2.1.x/reference/){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン")
* [Spring Boot Reference Guide for 1.5.x](https://docs.spring.io/spring-boot/docs/1.5.x/reference/){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン")

## Spring Framework と Spring Boot のどちらを選択するか
{: #spring-framework-or-boot}

新しいクラウド・ネイティブ・アプリでは、Spring Framework を使用することを選択した場合、Spring Boot も使用できます。Spring Boot は Spring ベースのアプリの構成とアセンブリーを簡素化し、さらに、[クラウド・ネイティブ・アプリ](/docs/java?topic=cloud-native-overview#overview)の作成を簡素化する Spring Actuator などの各種フィーチャーを提供します。

### Spring Boot Actuator
{: #spring-boot-actuator}

Spring Boot Actuator は、実行中の Spring アプリを検査するのに役立つエンドポイントのコレクションを定義します。 これらを有効にすることは、1 つの依存関係をアプリに追加するのと同じくらい単純です。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```
{: codeblock}

Spring Boot Actuator の構造と振る舞いは、Spring Boot 1 と Spring Boot 2 との間で大幅に変更されました。Boot 1 Actuator はアプリと並んで配置され、別個の構成とセキュリティー・メカニズムを備えていました。 Boot 2 バージョンはより機能的になり、残りの Spring アプリと統合されたセキュリティー・モデルを使用します。

Spring Boot 1.x アプリから Spring Boot 2.0 にマイグレーションする場合は、振る舞いが変わることが関連します。詳しくは、「[Boot 1 to Boot 2 migration guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.0-Migration-Guide#spring-boot-actuator){: new_window}」![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン") の Actuator のセクションを参照してください。
{: tip}

Boot 2 では、Actuator がミニ REST フレームワークとして機能します。 すべてのエンドポイントは `/actuator` コンテンツの下にあります。 ヘルス・チェックを実行する、またはメトリック、アプリ、および他の環境に関する情報を収集するエンドポイントが提供されます。 完全なリストについては、[Actuatorの資料](https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/html/production-ready-features.html#production-ready){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン") を参照してください。

Spring の `@Component` を使用して、独自の Actuator エンドポイントを簡単に作成できます。

```java
@Endpoint(id="example")
@Component
public class Example {
    @ReadOperation
    public String example() {
        return "{\"example\":\"value\"}";
    }
}
```
{: codeblock}

Spring Boot には、実行時にどの Actuator をアクティブにするかを構成するための 2 つのコントロールがあります。Actuator には「有効化」と「公開」を設定できます。到達可能であるためには、Actuator は両方でなければなりません。デフォルトでは多くのエンドポイントが有効になっていますが、ヘルスと情報だけが公開されます。 さらに、アプリに対して Spring セキュリティーが有効にされる場合、Actuator エンドポイントへのアクセスが明示的に許可される必要があります。

Spring Boot 1 の Actuator には、通常はアプリケーション・プロパティーで構成される独自のセキュリティー構成があります。 この場合、「有効化」と「公開」の代わりに「有効化」と「機密」になります。 機密性の高いエンドポイントでは、HTTP を介して動作する場合に認証が必要です。 デフォルトで、Spring Boot Actuator の /metrics エンドポイントは Spring Boot 1 で有効になっていますが、これは機密性が高いと見なされ、そのため権限が必要になるか、機密ではないと設定される必要があります。 一部の環境では、Spring Security を構成することが適切な答えとなりますが、Kubernetes ではメトリック・エンドポイントはクラスターに対して内部に留まります。 詳しくは、[Spring Boot 1 Actuator の資料](https://docs.spring.io/spring-boot/docs/1.5.2.RELEASE/reference/htmlsingle/#production-ready){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン") を参照してください。
{: note}

## Spring Cloud
{: #spring-cloud}

Spring Cloud はサード・パーティーのクラウド・テクノロジーと Spring プログラミング・モデルとの間の統合のコレクションです。 クラウドにデプロイする Spring アプリを構築する開発者を支援することを目的としています。 Spring Cloud によって達成される範囲は、そのモジュラー性が可能にしています。Spring Cloud 内の各プロジェクトは、特定のテクノロジーまたは一連のテクノロジーが中核になっているためです。

Spring Cloud プロジェクトは、構成よりも規則を優先するという一般的な Spring 方式に従います。 ほとんどの機能は、ビルド時に適切な依存関係を追加することによって使用可能になります。

Spring Cloud プロジェクトについて詳しくは、公式の [Spring Cloud プロジェクト・サイト](https://spring.io/projects/spring-cloud){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン") を参照してください。

### Spring Cloud と Kubernetes
{: #spring-cloud-kubernetes}

Spring Cloud Kubernetes は、Kubernetes で Spring アプリが実行するさまざまなタスクを、より容易に実行することを意図した Spring Cloud プロジェクトです。 Spring Cloud Kubernetes は、Kubernetes のコンセプトとサービスにマップされる一般的な Spring インターフェースの実装を提供します。

従来、Spring は Eureka などの Netflix ライブラリーをサービス・レジストリーとして使用し、Ribbon をクライアント・サイドのロード・バランサーとして使用してきました。 Kubernetes 環境では、これら両方の役割が Kubernetes そのものによって実現されるので、アプリケーション・レベルの機能が冗長化されます。 Spring Cloud Kubernetes は `DiscoveryClient` など、基礎となる Kubernetes メカニズムへの Spring 抽象化に適応します。

Istio がインストールされているクラスターで Spring Cloud Kubernetes を介して Ribbon ディスカバリーを使用する場合は、Istio が使用されることにより、サービスのルーティングに影響するので気を付ける必要があります。 クライアント・サイドのロード・バランシングは、クライアントが自身で宛先サービスを選択することを暗黙的に示しますが、Istio は呼び出しをどこか別の場所にルーティングしようとする可能性があります。 勝つことができるのは 1 つの方法だけであるため、呼び出しをどのようにルーティングできるかに関して不一致が生じることになります。この組み合わせは可能な限り避けてください。
{: note}

さらに、Spring Cloud Kubernetes は Kubernetes `ConfigMaps` と `Secrets` 間から Spring `@Autowired` 構成 Bean への統合を提供します。 アプリ実行中の `ConfigMaps` および `Secrets` への動的変更を処理するための戦略も含まれます。

最後に、Spring Cloud Kubernetes は、デフォルトの Spring Boot Actuator ヘルス・エンドポイントを増強して、デプロイメントに関連する追加情報を含めます。

詳しくは、[Spring Cloud Kubernetes の資料](https://cloud.spring.io/spring-cloud-static/spring-cloud-kubernetes/2.1.0.RC1/single/spring-cloud-kubernetes.html){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン ") を参照してください。


<!--
### Spring Cloud Streams
{: #spring-cloud-streams}


:FIXME:
-->
