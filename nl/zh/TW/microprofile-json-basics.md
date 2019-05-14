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

# 使用 JSON-P 及 JSON-B 進行 JSON 處理
{: #mp-json}

[JSON-B（JSON-Binding、JSR 367）](http://json-b.net/){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示") 以及 [JSON-P（JSON-Processing、JSR 374）](https://javaee.github.io/jsonp/){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示") 是兩個 Java&trade API 規格，這些規格定義 Java&trade 類別和 JSON 物件如何互動。JSON-P 提供 Java&trade API 來處理 JSON 格式的資料。JSON-B 在 JSON-P 上提供連結層，可讓您更容易將物件轉換為 JSON 以及從 JSON 轉換回物件。在大部分情況下，應先使用 JSON-B，再使用較低層次的 JSON-P。

## JSON-B
{: #java-json-b}

使用 **JSON-B**，即可使用「一般舊 Java 物件 (POJO)」與每個欄位的 getter 及 setter 方法搭配，來達成與 JSON 互相轉換。

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

若要搭配使用 JSON-B 與 Liberty，您需要在 `server.xml` 檔中啟用 `jsonb-1.0` 特性。您也可以使用 `microProfile-2.0`，其提供您所有 MP 特性，包括 JSON-B 及 JSON-P。

在您的 JAX-RS 類別中，您可以使用下列程式碼來建立 `Person` 物件，並包含稍早顯示的欄位：

```java
Address myAddress = new Address("501-B101", "4205 S Miami Blvd", "Durham", "NC", "27703");
Employee me = new Employee("John Alcorn", 228264, "Senior Software Engineer", myAddress);
```
{: codeblock}

JSON-B 會自動將 JSON 轉換成已定義的物件。例如，若您使用 MicroProfile REST 用戶端，對傳回類型為 POJO 的服務發出出埠呼叫，則 JSON-B 會自動將 JSON 回應轉換成 POJO 物件，然後您可以使用該物件來擷取屬性：

```java
// mpRestClient: outbound client request returning a POJO
// JSON-B converts the JSON response into a Person object
Employee employee = myClient.get("John Alcorn");
int serial = employee.getSerial();
Address address = employee.getAddress();
String city = address.getCity();
```
{: codeblock}

JSON-B 還提供一個剖析器，其採用 JSON 物件的「字串」表示法，並會傳回給您對應的 JSON-B 物件。若要使用此剖析器，您需要將下列 import 陳述式新增至微服務類別：

```java
import javax.json.bind.Jsonb;
import javax.json.bind.JsonbBuilder;
```
{: codeblock}

使用類似下列的內容來剖析儲存在 `myJSON` 變數中的 JSON 字串：

```java
Jsonb jsonb = JsonbBuilder.create();
Employee employee = jsonb.fromJson(myJSON, Employee.class);
```
{: codeblock}

一切都是使用 JSON-B 強力輸入的。例如，若您看到某個欄位名稱錯誤，您會看到編譯失敗，因為沒有這樣的方法。建立 POJO 的作業很冗長乏味，但是一旦定義欄位之後，大部分 IDE 都會支援產生 getter 及 setter，這很有幫助。

避免過度曝光內部詳細資料。有數個註釋可協助您控制如何將您的物件轉換成 JSON 以及從 JSON 轉換回物件：`@JsonbTransient` 識別完全不應該轉換的欄位，而 `@JsonbNillable` 指定應該如何處理空值。`@JsonbAdapter` 實作可進一步自訂如何將資料序列化和解除序列化，以在發訊格式與內部實作之間新增隔離。
{: tip}

## JSON-P
{: #java-json-p}

有 JSON-B 之前（在 Java EE 8 才隨附），**JSON-P** (JSON-Processing) 是 Java 程式碼內與 JSON 互動的標準化方式。在此之前，有各種專有的 JSON 剖析程式庫，例如，IBM 的 JSON4J。JSON-P 可用來剖析從 REST 呼叫中收到的 JSON，或者用來建構可從您自己的 JAX-RS 方法傳回的 JSON。

使用 JSON-P 時，必須在 `server.xml`（或可啟用所有 MP 技術的 `microProfile-2.0` 便利特性）中啟用 `jsonp-1.0` 特性。

以下是在 JSON-P 中使用 JSON 物件和陣列的 import 陳述式：

```java
import javax.json.Json;
import javax.json.JsonArray;
import javax.json.JsonArrayBuilder;
import javax.json.JsonObject;
import javax.json.JsonObjectBuilder;
```
{: codeblock}

若要使用從 REST API 呼叫接收到的 JSON，請在 `JsonObject` 上呼叫 `get` 方法，針對您要的欄位，傳入索引鍵。若要繼續 `Employee` 範例，如果您收到一個稱為 `employee` 的 `JsonObject`，請呼叫 `employee.get("name")` 來找出該人員的姓名，或呼叫 `employee.get("title")` 來取得其工作職稱。相較之下，JSON-P 很像在 Java&trade 中處理 `Map`。

現在，您想要在 Java&trade 程式碼中建置此類 JSON 物件。JSON-P 使用建置器型樣：使用 `JsonObjectBuilder` 來新增每個值，然後呼叫 `build()` 來產生 `JsonObject`，如下所示：

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

不同於 JSON-B，JSON-P 中的 `JsonObject` 是不可變的。如果您要更新欄位，則必須建立新的 `JsonObject`，複製欄位，然後進行變更。使用複製建構子來製作自己的 POJO 在這裡很有幫助。

MicroProfile Rest Client 同時支援 JSON-B 和 JSON-P 來進行適當的序列化和解除序列化，但是，類型安全和編譯時期意見讓 JSON-B 成為首選。
