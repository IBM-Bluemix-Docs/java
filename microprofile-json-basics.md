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

# JSON Handling with JSON-P and JSON-B
{: #mp-json}

[JSON-B (JSON-Binding, JSR 367)](http://json-b.net/){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon") and [JSON-P (JSON-Processing, JSR 374)](https://javaee.github.io/jsonp/){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon") are two Java API specifications that define how Java classes and JSON objects can interact. JSON-P provides a Java API for processing JSON-formatted data. JSON-B provides a binding layer on top of JSON-P, making it easier to convert objects to and from JSON. In most cases, JSON-B should be used in preference to the lower-level JSON-P.

## JSON-B
{: #java-json-b}

With **JSON-B** , conversion to and from JSON is achieved using a Plain Old Java Object (POJO) with a getter/setter method for each field. For example:

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

To use JSON-B with Liberty, you need the `jsonb-1.0` feature enabled in your `server.xml`. You can also use `microProfile-2.0`, which gives you all of the MP features, including JSON-B and JSON-P.

In your JAX-RS class, you'd use the following to create a Person object with the fields shown earlier:

```java
Address myAddress = new Address("501-B101", "4205 S Miami Blvd", "Durham", "NC", "27703");
Employee me = new Employee("John Alcorn", 228264, "Senior Software Engineer", myAddress);
```
{: codeblock}

JSON-B will automatically convert JSON into a defined object. For example, if you use the MicroProfile REST client to make an outbound call to a service with a POJO as a return type, JSON-B will automatically convert the JSON response into a POJO object, which you can then use to retrieve attributes:

```java
// mpRestClient: outbound client request returning a POJO
// JSON-B converts the JSON response into a Person object
Employee employee = myClient.get("John Alcorn");
int serial = employee.getSerial();
Address address = employee.getAddress();
String city = address.getCity();
```
{: codeblock}

JSON-B also provides a parser that will take a String representation of a JSON object and will give you back the corresponding JSON-B object. To use this, you need to add the following import statements to your microservice class:

```java
import javax.json.bind.Jsonb;
import javax.json.bind.JsonbBuilder;
```
{: codeblock}

Use something like the following to parse a JSON string stored in a `myJSON` variable:

```java
Jsonb jsonb = JsonbBuilder.create();
Employee employee = jsonb.fromJson(myJSON, Employee.class);
```
{: codeblock}

Everything is strongly typed with JSON-B. If you get a field name wrong, for example, you would see a compile failure since there wouldn't be such a method. Creating POJOs can be tedious, but most IDEs support generating getters and setters once the fields are defined, which helps.

Avoid over-exposure of internal details. Several annotations can help you control how your objects are converted to and from JSON: `@JsonbTransient` identifies fields that should not be converted at all, and `@JsonbNillable` specifies how null values should be handled. `@JsonbAdapter` implementations can further customize how data is serialised and deserialized for additional isolation between the wire format and the internal implementation.
{: tip}

## JSON-P
{: #java-json-p}

Before JSON-B existed (which arrived as part of Java EE 8), **JSON-P** (JSON-Processing) was a standardized way to interact with JSON within Java code. Prior to this, there were various proprietary JSON parsing libraries, such as IBM's JSON4J. JSON-P can be used to parse JSON you've received from a REST call, or to construct JSON you can return from your own JAX-RS methods.

Using JSON-P requires the `jsonp-1.0` feature be enabled in your `server.xml` (or the `microProfile-2.0` convenience feature, which enables all of the MP technologies).

Here are the import statements for working with JSON objects and arrays in JSON-P:

```java
import javax.json.Json;
import javax.json.JsonArray;
import javax.json.JsonArrayBuilder;
import javax.json.JsonObject;
import javax.json.JsonObjectBuilder;
```
{: codeblock}

To work with JSON received from a REST API call, you call the `get` method on a `JsonObject`, passing in the key for the field you want. To continue our `Employee` example above, if you received a `JsonObject` that you called `employee`, you would call `employee.get("name")` to find out the person's name, or `employee.get("title")` to get their job title. JSON-P is a lot like dealing with a `Map` in Java, as a comparison.

Now, say you want to build such a JSON object in your Java code. JSON-P uses a builder pattern: use a `JsonObjectBuilder` to add each value, then call `build()` to produce the `JsonObject`, like such:

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

Note that a `JsonObject` in JSON-P is immutable, unlike with JSON-B. If you want to update fields, you have to create a new `JsonObject`, duplicate the fields, and then make changes. Making your own POJOs with copy constructors can help here.

JSON-B and JSON-P are both supported by the MicroProfile Rest Client for proper serialization and deserialization, but type-safety and compile-time feedback make JSON-B the preferred choice.
