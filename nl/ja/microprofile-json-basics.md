---

copyright:
  years: 2019
lastupdated: "2019-03-27"

keywords: json-b, json-p, json-binding, json response, pojo object, pojo, jsonobject, jsonobjectbuilder, java api json

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

# JSON-P および JSON-B による JSON 処理
{: #mp-json}

[JSON-B (JSON-Binding、JSR 367)](http://json-b.net/){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン") および [JSON-P (JSON-Processing、JSR 374)](https://javaee.github.io/jsonp/){: new_window} ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン") は、Java クラスと JSON オブジェクトが対話する方法を定義する、2 つの Java API 仕様です。JSON-P は JSON 形式のデータを処理する Java API を提供します。JSON-B は JSON-P の上のバインディング層を提供し、オブジェクトから JSON へ、JSON からオブジェクトへの変換を容易にします。ほとんどの場合、下位レベルの JSON-P よりも JSON-B を優先して使用する必要があります。

## JSON-B
{: #java-json-b}

**JSON-B** を使用する場合、JSON への変換または JSON からの変換は、各フィールドに getter/setter メソッドを指定した Plain Old Java Object (POJO) を使用して行われます。例えば次のようにします。

```java
public class Employee {
  private String fName;
  private int fSerial;
  private String fTitle;
  private Address fAddress;

  public Employee() {}
  public Employee(String name, int serial, String title, Address address) {
    fName = name;
    fSerial = serial;
    fTitle = title;
    fAddress = address;
  }

  public String getName() { return fName; }
  public void setName(String name) { fName = name; }
  public int getSerial() { return fSerial; }
  public void setSerial(int serial) { fSerial = serial; }
  public String getTitle() { return fTitle; }
  public void setTitle(String title) { fTitle = title; }
  public Address getAddress() { return fAddress; }
  public void setAddress(Address address) { fAddress = address; }
}

public class Address {
  private String fOffice;
  private String fStreet;
  private String fCity;
  private String fState;
  private String fZip;

  public Address() {}
  public Address(String office, String street, String city, String state, String zip) {
    fOffice = office;
    fStreet = street;
    fCity = city;
    fState = state;
    fZip = zip;
  }

  public String getOffice() { return fOffice; }
  public void setOffice(String office) { fOffice = office; }
  public String getStreet() { return fStreet; }
  public void setStreet(String street) { fStreet = street; }
  public String getCity() { return fCity; }
  public void setCity(String city) { fCity = city; }
  public String getState() { return fState; }
  public void setState(String state) { fState = state; }
  public String getZip() { return fZip; }
  public void setZip(String zip) { fZip = zip; }
}
```
{: codeblock}

Liberty で JSON-B を使用するには、`server.xml` で `jsonb-1.0` フィーチャーを有効にする必要があります。また、`microProfile-2.0` も使用しなければなりません。こうすると、JSON-Bと JSON-P を含めたすべての MP 機能が提供されます。

JAX-RS クラスで前述のフィールドを指定し、以下を使用して Person オブジェクトを作成できます。

```java
Address myAddress = new Address("501-B101", "4205 S Miami Blvd", "Durham", "NC", "27703");
Employee me = new Employee("John Alcorn", 228264, "Senior Software Engineer", myAddress);
```
{: codeblock}

JSON から定義されたオブジェクトへの変換は、JSON-B によって自動的に行われます。例えば、MicroProfile REST クライアントを使用して、POJO を戻り型とするアウトバウンド・コールを作成する場合、JSON-B によって JSON 応答が POJO オブジェクトに自動的に変換され、それを使ってユーザーは属性を取り出すことができます。

```java
// mpRestClient: outbound client request returning a POJO
// JSON-B converts the JSON response into a Person object
Employee employee = myClient.get("John Alcorn");
int serial = employee.getSerial();
Address address = employee.getAddress();
String city = address.getCity();
```
{: codeblock}

また、JSON-B には JSON オブジェクトの String 表現を取るコマンド解析機能も用意されており、対応する JSON-B オブジェクトをユーザーに返します。この機能を使用するには、マイクロサービス・クラスに次の IMPORT ステートメントを追加する必要があります。

```java
import javax.json.bind.Jsonb;
import javax.json.bind.JsonbBuilder;
```
{: codeblock}

以下のようなコードを使用して、`myJSON` 変数に保管されている JSON 文字列を解析します。

```java
Jsonb jsonb = JsonbBuilder.create();
Employee employee = jsonb.fromJson(myJSON, Employee.class);
```
{: codeblock}

JSON-B では、すべてが強い型付きです。例えば、フィールド名を間違えた場合、そのようなメソッドがないためにコンパイルが失敗することになります。POJO の作成は退屈かもしれませんが、ほとんどの IDE はいったんフィールドが定義されると getter や setter の生成をサポートしているので便利です。

内部詳細の過剰な公開は避けてください。いくつかアノテーションを付けることで、オブジェクトと JSON 間の変換方法を管理するのに役立ちます。`@JsonbTransient` は、全く変換されないフィールドを識別し、`@JsonbNillable` は、NULL 値の処理方法を指定します。`@JsonbAdapter` を実装することによって、データのシリアライズおよびデシリアライズ方法をさらにカスタマイズでき、ワイヤー・フォーマットと内部実装間をさらに分離できます。
{: tip}

## JSON-P
{: #java-json-p}

JSON-B が登場する前は (JSON-B は Java EE 8 の一部として発表されました)、**JSON-P** (JSON-Processing) が、Java コード内で JSON とやり取りするための標準的な方法でした。そのさらに以前は、IBM の JSON4J など、さまざまな専有の JSON 構文解析ライブラリーがありました。JSON-P は、REST 呼び出しから受け取った JSON を解析したり、ユーザー独自の JAX-RS メソッドから返される JSON を構成するために使用できます。

JSON-P を使用するには、`jsonp-1.0` フィーチャーを `server.xml` で有効にする必要があります。(または `microProfile-2.0` という便利なフィーチャーを有効にします。この場合、すべての MP テクノロジーが使用可能になります。)

JSON-P で JSON オブジェクトおよび配列を扱うための IMPORT ステートメントを以下に示します。

```java
import javax.json.Json;
import javax.json.JsonArray;
import javax.json.JsonArrayBuilder;
import javax.json.JsonObject;
import javax.json.JsonObjectBuilder;
```
{: codeblock}

REST API 呼び出しから受け取った JSON を操作する場合は、`JsonObject` オブジェクトで `get` メソッドを呼び出し、希望するフィールドのキーを渡します。上記の `Employee` サンプル・コードを続行するには、`employee` を呼び出した `JsonObject` を受け取った場合に、`employee.get("name")` を呼び出してその人の名前を、または `employee.get("title")` を呼び出してその役職名を見つけます。例えてみるなら、JSON-P は Java で `Map` を操作するようなものです。

さて、例えば、Java コードで次のような JSON オブジェクトを作成するとします。JSON-P ではビルダー・パターンを使用します。`JsonObjectBuilder` を使用して各値を追加してから `build()` を呼び出して `JsonObject` を作成します。以下に例を示します。

```java
JsonObjectBuilder addressBuilder = Json.createObjectBuilder();

addressBuilder.add("office", "501-B101");
addressBuilder.add("street", "4205 S Miami Blvd");
addressBuilder.add("city", "Durham");
addressBuilder.add("state", "NC");
addressBuilder.add("zip", "27703");

JsonObjectBuilder employeeBuilder = Json.createObjectBuilder();

employeeBuilder.add("name", "John Alcorn");
employeeBuilder.add("serial", 228264);
employeeBuilder.add("title", "Senior Software Engineer");

employeeBuilder.add("address", addressBuilder);

JsonObject employee = employeeBuilder.build();
```
{: codeblock}

JSON-B とは異なり、JSON-P では `JsonObject` が変更不可能であることに気を付けてください。フィールドを更新する場合は、新しい `JsonObject` を作成し、フィールドを複製してから変更する必要があります。この際にコピー・コンストラクターで独自の POJO を作成すると役立ちます。

JSON-B と JSON-P はいずれも、MicroProfile Rest Client での正しいシリアライゼーションおよびデシリアライゼーションがサポートされていますが、型安全性とコンパイル時間のフィードバックの面で、JSON-B が優先される選択肢となります。
