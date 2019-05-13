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

# JSON-P 및 JSON-B를 사용한 JSON 처리
{: #mp-json}

[JSON-B(JSON-Binding, JSR 367)](http://json-b.net/){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘") 및 [JSON-P(JSON-Processing, JSR 374)](https://javaee.github.io/jsonp/){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")는 Java&trade 클래스와 JSON 오브젝트가 상호작용하는 방법을 정의하는 두 개의 Java&trade API 스펙입니다. JSON-P는 JSON 형식 데이터를 처리하기 위해 Java&trade API를 제공합니다. JSON-B는 JSON-P의 맨 위에 바인딩 계층을 제공하며, 이를 통해 오브젝트를 JSON으로 그리고 그 반대로 쉽게 변환할 수 있습니다. 대부분의 경우 JSON-B는 하위 레벨 JSON-P에 우선하여 사용되어야 합니다.

## JSON-B
{: #java-json-b}

**JSON-B**를 사용하면 각 필드에 대해 getter 및 setter 메소드와 함께 POJO(Plain Old Java Object)를 사용하여 JSON에서의 변환이 이루어집니다.

예를 들어, 다음과 같습니다.
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

Liberty에서 JSON-B를 사용하려면 `server.xml` 파일에서 `jsonb-1.0` 기능이 사용 가능해야 합니다. 또한 `microProfile-2.0`을 사용할 수 있으며, 이는 JSON-B 및 JSON-P를 포함한 모든 MP 기능을 제공합니다.

JAX-RS 클래스에서 다음을 사용하여 이전에 표시된 필드에서 `Person` 오브젝트를 작성합니다.

```java
Address myAddress = new Address("501-B101", "4205 S Miami Blvd", "Durham", "NC", "27703");
Employee me = new Employee("John Alcorn", 228264, "Senior Software Engineer", myAddress);
```
{: codeblock}

JSON-B는 자동으로 JSON을 정의된 오브젝트로 변환합니다. 예를 들어, MicroProfile REST 클라이언트를 사용하여 리턴 유형으로서 POJO를 통해 서비스에 대한 아웃바운드 호출을 작성하는 경우, JSON-B가 JSON 응답을 POJO 오브젝트로 자동 변환하며, 사용자는 이를 사용하여 속성을 검색할 수 있습니다.

```java
// mpRestClient: outbound client request returning a POJO
// JSON-B converts the JSON response into a Person object
Employee employee = myClient.get("John Alcorn");
int serial = employee.getSerial();
Address address = employee.getAddress();
String city = address.getCity();
```
{: codeblock}

또한 JSON-B는 JSON 오브젝트에 대한 문자열 표시를 사용하고 해당 JSON-B 오브젝트를 다시 제공하는 구문 분석기를 제공합니다. 구문 분석기를 사용하려면 다음 import 문을 마이크로서비스 클래스에 추가해야 합니다.

```java
import javax.json.bind.Jsonb;
import javax.json.bind.JsonbBuilder;
```
{: codeblock}

다음과 같은 내용을 사용하여 `myJSON` 변수에 저장된 JSON 문자열을 구문 분석하십시오.

```java
Jsonb jsonb = JsonbBuilder.create();
Employee employee = jsonb.fromJson(myJSON, Employee.class);
```
{: codeblock}

모든 내용은 철저하게 JSON-B로 입력됩니다. 예를 들어, 필드 이름이 잘못되면 이러한 메소드가 없기 때문에 컴파일 오류가 표시된다. POJO 작성이 지루할 수 있지만, 대부분의 IDE는 필드가 정의되면 Getter와 Setter 생성을 지원하며, 이는 도움이 됩니다.

내부 세부사항의 과다 노출은 피하십시오. 여러 어노테이션을 사용하면 JSON에서 오브젝트로(그 반대로) 변환하는 방법을 제어할 수 있습니다. `@JsonbTransient`는 변환하면 안 되는 필드를 식별하고 `@JsonbNillable`은 널값을 처리하는 방법을 지정합니다. `@JsonbAdapter` 구현은 연결 형식과 내부 구현 간 추가 격리를 위해 데이터를 직렬화하고 직렬화 해제하는 방법을 사용자 정의할 수 있습니다.
{: tip}

## JSON-P
{: #java-json-p}

JSON-B가 존재하기 전에는(Java EE 8의 일부로서 제공됨), **JSON-P**(JSON-Processing)가 Java 코드 내 JSON과 상호작용하는 표준화된 방법이었습니다. 이에 앞서 IBM의 JSON4J와 같은 다양한 전용 JSON 구문 분석 라이브러리가 있었습니다. JSON-P를 사용하여 REST 호출에서 수신한 JSON을 구문 분석하거나 고유 JAX-RS 메소드에서 리턴할 수 있는 JSON을 구성할 수 있습니다.

JSON-P를 사용하려면 `jsonp-1.0` 기능이 `server.xml`에서 사용 가능해야 합니다(또는 모든 MP 기술을 사용하는 `microProfile-2.0` 편의 기능).

다음은 JSON-P의 JSON 오브젝트 및 배열 관련 작업을 위한 import 문입니다.

```java
import javax.json.Json;
import javax.json.JsonArray;
import javax.json.JsonArrayBuilder;
import javax.json.JsonObject;
import javax.json.JsonObjectBuilder;
```
{: codeblock}

REST API 호출에서 수신된 JSON 관련 작업을 수행하려면 원하는 필드의 키를 전달하여 `JsonObject`에서 `get` 메소드를 호출합니다. `Employee` 예제를 계속하기 위해서는 `employee`를 호출한 `JsonObject`를 수신한 경우 `employee.get("name")`을 호출하여 사용자의 이름을 찾거나 `employee.get("title")`을 호출하여 직위를 찾습니다. JSON-P는 예를 들어 Java&trade의 `Map` 처리와 매우 유사합니다.

이제 Java&trade 코드로 이와 같은 JSON 오브젝트를 빌드하려고 합니다. JSON-P는 빌더 패턴을 사용합니다. 다음과 같이 `JsonObjectBuilder`를 사용하여 값을 각각 추가한 다음 `build()`를 호출하여 `JsonObject`를 생성하십시오.

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

JSON-B와 달리 JSON-P의 `JsonObject`는 불변입니다. 필드를 업데이트하려면 `JsonObject`를 새로 작성하고 필드를 복제한 다음 변경해야 합니다. 복사 생성자를 사용하여 고유 POJO를 작성하면 도움이 될 수 있습니다.

JSON-B 및 JSON-P 둘 다 올바른 직렬화 및 직렬화 해제를 위해 MicroProfile Rest Client에서 지원되지만, 형식 변환이 없는 컴파일 시간 피드백을 통해 JSON-B가 선호하는 방식이 됩니다.
