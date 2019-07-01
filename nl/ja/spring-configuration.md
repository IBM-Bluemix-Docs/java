---

copyright:
  years: 2018, 2019
lastupdated: "2019-06-10"

keywords: spring environment, spring credentials, ibm-cloud-spring-boot-service-bind, service bindings spring, vcap_services spring, access credential spring

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

# Spring 環境の構成
{: #spring-configuration}

サービス構成と資格情報 (サービス・バインディング) の管理は、プラットフォーム間で異なります。 Cloud Foundry では、サービス・バインディングの詳細情報が、ストリング化された JSON オブジェクト内に格納され、環境変数 `VCAP_SERVICES` としてアプリケーションに渡されます。 Kubernetes は、サービス・バインディングを、ストリング化された JSON 属性またはフラットな属性として `ConfigMaps` 内または `Secrets` 内に格納するので、コンテナー化されたアプリに環境変数として渡したり、一時ボリュームとしてマウントしたりできます。 ローカル開発は異なる可能性があります。このような違いがある中で、環境固有のコード・パスを使用せずに移植可能な方法で作業することは容易ではありません。

## 既存の Spring アプリへの {{site.data.keyword.cloud_notm}} 構成の追加
{: #spring-cloud-env}

[{{site.data.keyword.cloud_notm}} Spring サービス・バインディング](https://github.com/ibm-developer/ibm-cloud-spring-bind){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン")・ライブラリーは、さまざまなクラウド環境の情報を抽象化することによって、一貫した方法で Spring アプリが構成にアクセスできるようにします。こうした情報としては、ローカル、Cloud Foundry、Cloud Foundry エンタープライズ環境、Kubernetes、仮想マシン、{{site.data.keyword.openwhisk}} があり、これらが抽象化されて `mappings.json` ファイルの検索パターンの配列に書き込まれます。Spring アプリが起動すると、ライブラリーはこのファイルを読み取り、構成値を検出して割り当てるためにパターンを適用します。

`mappings.json` ファイルについて詳しくは、[挿入されたサービス資格情報の操作](/docs/java?topic=cloud-native-configuration#portable-credentials)を参照してください。

### Spring アプリへのライブラリーの追加
{: #spring-add-service-library}

{{site.data.keyword.cloud_notm}} Spring Service Bindings ライブラリーを使用するには、`pom.xml` ファイルまたは `build.gradle` ファイルに `ibm-cloud-spring-boot-service-bind` 依存関係を指定します。

**Maven の例**:

```xml
<dependency>
   <groupId>com.ibm.cloud</groupId>
   <artifactId>ibm-cloud-spring-boot-service-bind</artifactId>
   <version>1.1.1</version>
</dependency>
```
{: codeblock}

**Gradle の例**:

```json
dependencies {
    compile group: 'com.ibm.cloud', name: 'ibm-cloud-spring-boot-service-bind', version: '1.1.1'
}
```
{: codeblock}

### 資格情報へのアクセス
{: #spring-access-credentials}

コードの中でサービス構成にアクセスするには、`@Value` アノテーションを使用するか、Spring フレームワークの Environment クラスの `getProperty ()` メソッドを使用します。

例: IBM Cloudant NoSQL DB サービスの構成データへのアクセス

```java
   // Use the @Value annotation
   @Value("cloudant.url")
   String cloudantUrl;

   // Use the Spring Environment object
   @Autowired
   Environment env;

   String cloudantUsername =
        env.getProperty("cloudant.username");
```
{: codeblock}

## 次のステップ
{: #spring-config-next-steps notoc}

Spring Boot について詳しくは、以下をご覧ください。

* [Spring Boot 1.5.x](https://docs.spring.io/spring-boot/docs/1.5.x/reference/html/){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン")
* [Spring Boot 2.1.x](https://docs.spring.io/spring-boot/docs/2.1.x/reference/html/){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン")

{{site.data.keyword.IBM}} および Spring についてもっと詳しく知りたい場合は、以下をご覧ください。

* [IBM Developer: Spring](https://developer.ibm.com/technologies/spring/){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン")
* [IBM Spring Service Bindings](https://github.com/ibm-developer/ibm-cloud-spring-bind){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン")
