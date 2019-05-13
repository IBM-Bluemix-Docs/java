---

copyright:
  years: 2019
lastupdated: "2019-04-22"

keywords: dependency injection, cdi jakarta, cdi beans, cdi scopes, bean lifecycle, context injection microprofile, microprofile cdi

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

# Contexts and Dependency Injection (CDI、JSR-365)
{: #mp-cdi}

CDI は Jakarta-EE と MicroProfile の両方において中心的な要素です。これは、さまざまなコンポーネントを共につなぐために使用されます。 CDI は、**コンテキスト**を定義し、Bean のライフサイクルを指定および管理します。また、**依存関係の注入**を使用し、宣言された依存関係を動的かつタイプ・セーフな方法で満たします。 さらに CDI は、疎結合のイベントおよびインターセプター・モデルを使用します。他のフレームワークで独自の CDI Bean を定義したり、既存のコンポーネントを更新したりするための強力かつ柔軟な移植可能拡張メカニズムも提供します。

MicroProfile 仕様書 (例えば MicroProfile Config、MicroProfile JWT) は、新しい機能で既存の規格 (JAX-RS など) を強化するために CDI 拡張メカニズムに依存しており、CDI 優先のアプローチが取られています。 CDI 1.2 は MicroProfile 1.0 リリース以降の一部を構成しています (MicroProfile 2.0 は CDI 2.0 に移行しました)。

## CDI Bean
{: #cdi-bean}

CDI は _Bean_ で動作します。 CDI Bean は [Bean 定義アノテーション](https://docs.jboss.org/cdi/spec/2.0/cdi-spec.html){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg " 外部リンク・アイコン") を使用して作成されます。 引数のないコンストラクター、または `@Inject` アノテーションがあるコンストラクターを持つ、ほとんどすべての Plain Old Java&trade Object (POJO) が Bean になります。

CDI Bean を定義するアノテーションを発見する必要があります。CDI アノテーション・スキャンは、`META-INF` フォルダー (.jar アーカイブの場合) または `WEB-INF` フォルダー (.war アーカイブの場合) 内の `beans.xml` ファイルを使用して有効にすることができます。このファイルは空にすることも、あるいは次のような内容を含めることもできます。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://xmlns.jcp.org/xml/ns/javaee"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
            http://xmlns.jcp.org/xml/ns/javaee/beans_1_1.xsd"
      version="1.1" bean-discovery-mode="all">
  // enable interceptors; decorators; alternatives
</beans>
```
{: codeblock}

空の `beans.xml` ファイル、または `bean-discovery-mode="all"` を指定した `beans.xml` が存在する場合、すべての潜在的なオブジェクトが CDI Bean になります。存在しない場合は、CDI Bean 定義アノテーションを持つオブジェクトだけが CDI Bean になります。

## CDI スコープ
{: #cdi-scopes}

CDI スコープ・タイプのアノテーションは、Bean のライフサイクルを制御します。

* アプリケーションがアクティブである限りインスタンスが存続する場合は、`@ApplicationScoped` クラス・レベルのアノテーションを使用します。
* 要求 (例えば、サーブレットの要求) ごとに Bean の新しいインスタンスを作成する場合は、`@RequestScoped` クラス・レベルのアノテーションを使用します。
* 新しいインスタンスが別のオブジェクトに属する場合は、`@Dependent` クラス・レベルのアノテーションを使用します。従属 Bean のインスタンスは、異なるクライアント間または異なる注入ポイント間で共有されることはありません。 属するオブジェクトが作成されるとインスタンス化され、属するオブジェクトが破棄されると破棄されます。
* クラス・レベルのアノテーションが定義されていない場合は、`@Dependent` がデフォルトになります。

CDI コンテナーは、定義されたスコープに従って Bean インスタンスを作成および破棄し、それらを適切なコンテキストに関連付けてから、それらを他のオブジェクトの依存関係として注入します。

## CDI Bean の注入
{: #cdi-inject}

CDI は、`@Inject` を介して、定義済みの Bean を他のコンポーネントに注入します。 例えば、以下の POJO `MyBean` は CDI Bean です。

```java
@ApplicationScoped
public class MyBean {
    int i=0;
    public String sayHello() {
        return "MyBean hello " + i++;
    }
}
```
{: codeblock}

これは `@ApplicationScoped` なので、インスタンスは 1 つだけ作成されます。 以下の `MyRestEndPoint` は、`@RequestScoped` です。つまり、要求ごとにインスタンスが作成されます。 CDI はそれぞれに同じ `MyBean` インスタンスを挿入します。

```java
@RequestScoped
@Path("/hello")
public class MyRestEndPoint {
    @Inject MyBean bean;

    @GET
    @Produces (MediaType.TEXT_PLAIN)
    public String sayHello() {
        return bean.sayHello();
    }
}
```
{: codeblock}

CDI は、次の Bean のライフサイクルの管理を行っています。

* Bean を初期化するには、コンストラクターの代わりに `@PostConstruct` のアノテーションを付けたメソッドを使用します。 これは、CDI コンテナーで依存関係および関連付けられている Bean を適切なコンテキストに注入した後に呼び出されます。
* Bean をクリーンアップするには、`@PreDestroy` というアノテーションを付けたメソッドを使用します。 これは、依存関係が削除される前に呼び出されます。
