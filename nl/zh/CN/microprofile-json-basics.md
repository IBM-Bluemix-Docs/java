---

copyright:
  years: 2019
lastupdated: "2019-04-30"

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

# 使用 JSON-P 和 JSON-B 进行 JSON 处理
{: #mp-json}

[JSON-B (JSON-Binding, JSR 367)](http://json-b.net/){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标") 和 [JSON-P (JSON-Processing, JSR 374)](https://javaee.github.io/jsonp/){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标") 是两个 Java&trade API 规范，用于定义 Java&trade 类与 JSON 对象如何进行交互。JSON-P 提供的 Java&trade API 用于处理 JSON 格式数据。JSON-B 在 JSON-P 的基础上提供了一个绑定层，能更轻松地在对象与 JSON 之间进行相互转换。在大多数情况下，JSON-B 应优先于较低级别的 JSON-P 进行使用。

## JSON-B
{: #java-json-b}

使用 **JSON-B** 将普通旧式 Java 对象 (POJO) 用于每个字段的 getter 方法和 setter 方法，可实现与 JSON 之间的相互转换。

例如：
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

要将 JSON-B 用于 Liberty，需要 `server.xml` 中启用的 `jsonb-1.0` 功能。此外，还可以使用 `microProfile-2.0`，这将为您提供所有 MP 功能，包括 JSON-B 和 JSON-P。

在 JAX-RS 类中，您将使用以下内容来创建具有先前所示字段的`个人`对象：

```java
Address myAddress = new Address("501-B101", "4205 S Miami Blvd", "Durham", "NC", "27703");
Employee me = new Employee("John Alcorn", 228264, "Senior Software Engineer", myAddress);
```
{: codeblock}

JSON-B 会自动将 JSON 转换为定义的对象。例如，如果使用 MicroProfile REST 客户机对服务发起出站调用，并将 POJO 作为返回类型，那么 JSON-B 会自动将 JSON 响应转换为 POJO 对象，然后您可以将其用于检索属性：

```java
// mpRestClient: outbound client request returning a POJO
// JSON-B converts the JSON response into a Person object
Employee employee = myClient.get("John Alcorn");
int serial = employee.getSerial();
Address address = employee.getAddress();
String city = address.getCity();
```
{: codeblock}

JSON-B 还提供了一个解析器，用于接受 JSON 对象的字符串表示法，并返回相应的 JSON-B 对象。要使用此解析器，需要将以下 import 语句添加到微服务类：

```java
import javax.json.bind.Jsonb;
import javax.json.bind.JsonbBuilder;
```
{: codeblock}

使用类似下面的内容来解析存储在 `myJSON` 变量中的 JSON 字符串：

```java
Jsonb jsonb = JsonbBuilder.create();
Employee employee = jsonb.fromJson(myJSON, Employee.class);
```
{: codeblock}

对于 JSON-B，所有内容都是强类型的。例如，如果您错误地收到某个字段名称，那么将看到编译失败，因为不存在此类方法。创建 POJO 可能很繁琐，但大多数 IDE 支持在定义字段后生成 getter 和 setter 方法，这会很有帮助。

避免过度公开内部细节。有若干注释可以帮助您控制对象与 JSON 之间相互转换的方式：`@JsonbTransient` 用于标识完全不应进行转换的字段，`@JsonbNillable` 用于指定应如何处理 null 值。`@JsonbAdapter` 实现可以进一步定制如何对数据进行序列化和反序列化，从而加强有线格式和内部实现之间的隔离。
{: tip}

## JSON-P
{: #java-json-p}

在 JSON-B（作为 Java EE 8 的一部分推出）出现之前，**JSON-P** (JSON-Processing) 是与 Java 代码中的 JSON 进行交互的标准化方式。在此之前，有各种专有 JSON 解析库，例如 IBM 的 JSON4J。JSON-P 可用于解析从 REST 调用收到的 JSON，或者用于构造 JSON，供您通过自己的 JAX-RS 方法返回。

使用 JSON-P 需要在 `server.xml` 中启用 `jsonp-1.0` 功能（或启用方便的 `microProfile-2.0` 功能，这将启用所有 MP 技术）。

下面是用于在 JSON-P 中使用 JSON 对象和数组的 import 语句：

```java
import javax.json.Json;
import javax.json.JsonArray;
import javax.json.JsonArrayBuilder;
import javax.json.JsonObject;
import javax.json.JsonObjectBuilder;
```
{: codeblock}

要使用从 REST API 调用收到的 JSON，可以对 `JsonObject` 调用 `get` 方法，并传入所需字段的键。继续上面的 `Employee` 示例，如果收到名为 `employee` 的 `JsonObject`，那么将调用 `employee.get("name")` 来查找此人的姓名，或者调用 `employee.get("title")` 来获取其职位。作为对照，JSON-P 非常类似于在 Java&trade 中处理 `Map`。

现在，假设要在 Java&trade 代码中构建此类 JSON 对象。JSON-P 使用的构建器模式为：使用 `JsonObjectBuilder` 添加每个值，然后调用 `build()` 以生成 `JsonObject`，如下所示：

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

JSON-P 中的 `JsonObject` 是不可变的，这与 JSON-B 不同。如果要更新字段，必须创建新的 `JsonObject`，复制这些字段，然后进行更改。在此，使用副本构造函数创建您自己的 POJO 会很有帮助。

MicroProfile Rest Client 同时支持 JSON-B 和 JSON-P 用于正确的序列化和反序列化，但 JSON-B 凭借类型安全和编译时反馈功能成为首选。
