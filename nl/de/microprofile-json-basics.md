---

copyright:
  years: 2019
lastupdated: "2019-06-10"

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

# JSON-Handhabung mit JSON-P und JSON-B
{: #mp-json}

[JSON-B (JSON-Binding, JSR 367)](http://json-b.net/){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") und [JSON-P (JSON-Processing, JSR 374)](https://javaee.github.io/jsonp/){: new_window} ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link") sind zwei Java-API-Spezifikationen, die definieren, wie Java-Klassen und JSON-Objekte interagieren können. JSON-P bietet eine Java&-API für die Verarbeitung von mit JSON formatierten Daten. JSON-B stellt zusätzlich zu JSON-P einen Binding-Layer bereit und vereinfacht somit die Konvertierung von Objekten in oder aus JSON. In den meisten Fällen sollte JSON-B JSON-P vorgezogen werden, das sich auf einer niedrigeren Stufe befindet. 

## JSON-B
{: #java-json-b}

Mit **JSON-B** wird die Konvertierung in und aus JSON unter Verwendung eines Plain Old Java&trade; Object (POJO) mit einer Getter/Setter-Methode für jedes Feld erreicht. 

Beispiel:
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

Wenn Sie JSON-B mit Liberty verwenden möchten, müssen Sie das Feature `jsonb-1.0` in der Datei `server.xml` aktivieren. Sie können auch `microProfile-2.0` verwenden, wodurch Sie alle MP-Features, einschließlich JSON-B und JSON-P, nutzen können.

Verwenden Sie in der JAX-RS-Klasse den folgenden Code, um ein Objekt `Person` mit den zuvor angegebenen Feldern zu erstellen:

```java
Address myAddress = new Address("501-B101", "4205 S Miami Blvd", "Durham", "NC", "27703");
Employee me = new Employee("John Alcorn", 228264, "Senior Software Engineer", myAddress);
```
{: codeblock}

JSON-B konvertiert JSON automatisch in ein definiertes Objekt. Wenn Sie beispielsweise den MicroProfile-REST-Client zum Durchführen eines ausgehenden Aufrufs an einen Service mit einem POJO als Rückgabetyp verwenden, konvertiert JSON-B die JSON-Antwort automatisch in ein POJO-Objekt, das Sie dann zum Abrufen von Attributen verwenden können:

```java
// mpRestClient: abgehende Clientanforderung, die ein POJO zurückgibt
// JSON-B konvertiert die JSON-Antwort in ein Personenobjekt
Employee employee = myClient.get("John Alcorn");
int serial = employee.getSerial();
Address address = employee.getAddress();
String city = address.getCity();
```
{: codeblock}

JSON-B bietet zudem einen Parser, der eine Zeichenfolgedarstellung eines JSON-Objekts verwendet und Ihnen das entsprechende JSON-B-Objekt zurückgibt. Um den Parser zu verwenden, müssen Sie die folgenden Importanweisungen zu Ihrer Mikroserviceklasse hinzufügen:

```java
import javax.json.bind.Jsonb;
import javax.json.bind.JsonbBuilder;
```
{: codeblock}

Verwenden Sie zum Parsen einer JSON-Zeichenfolge, die in einer `myJSON`-Variablen gespeichert ist, in etwa den folgenden Code:

```java
Jsonb jsonb = JsonbBuilder.create();
Employee employee = jsonb.fromJson(myJSON, Employee.class);
```
{: codeblock}

Mit JSON-B ist alles stark typisiert. Wenn Sie beispielsweise einen falschen Feldnamen verwenden, würde ein Kompilierungsfehler angezeigt werden, da es eine solche Methode nicht geben würde. Die Erstellung von POJOs kann mühsam sein, aber die meisten IDEs unterstützen die Generierung von Gettern und Settern, sobald die Felder definiert sind, was sich in vielen Fällen als hilfreich erweist. 

Vermeiden Sie die Darstellung zu vieler interner Details. Mit mehreren Annotationen können Sie steuern, wie Ihre Objekte in und aus JSON konvertiert werden: `@JsonbTransient` gibt Felder an, die nicht konvertiert werden sollen, und `@JsonbNillable` gibt an, wie Nullwerte gehandhabt werden sollen. Mit Implementierungen von `@JsonbAdapter` können Sie die Serialisierung und Deserialisierung von Daten weiter anpassen, um eine zusätzliche Isolation zwischen dem Verbindungsformat und der internen Implementierung zu erhalten.
{: tip}

## JSON-P
{: #java-json-p}

Vor der Einführung von JSON-B (als Teil von Java&trade; EE 8) war **JSON-P** (JSON-Processing) eine standardisierte Methode für die Interaktion mit JSON in Java&trade;-Code. Davor gab es verschiedene proprietäre JSON-Parsing-Bibliotheken, wie z. B. JSON4J von IBM. JSON-P kann verwendet werden, um JSON syntaktisch zu analysieren, die Sie von einem REST-Aufruf erhalten haben, oder um JSON zu erstellen, die Sie von Ihren eigenen JAX-RS-Methoden zurückgeben können.

Die Verwendung von JSON-P erfordert die Aktivierung des Features `jsonp-1.0` in der Datei `server.xml` (oder des optionalen Features `microProfile-2.0`, das alle MP-Technologien aktiviert).

Im Folgenden finden Sie die Importanweisungen für das Arbeiten mit JSON-Objekten und -Arrays in JSON-P:

```java
import javax.json.Json;
import javax.json.JsonArray;
import javax.json.JsonArrayBuilder;
import javax.json.JsonObject;
import javax.json.JsonObjectBuilder;
```
{: codeblock}

Um mit von einem REST-API-Aufruf erhaltenen JSON zu arbeiten, können Sie die `GET`-Methode für ein `JsonObject` aufrufen und somit den Schlüssel für das gewünschte Feld übergeben. Wenn Sie ausgehend vom `Employee`-Beispiel ein `JsonObject` mit der Bezeichnung `employee` erhalten haben, würden Sie `employee.get("name")` aufrufen, um den Namen der Person zu ermitteln, oder `employee.get("title")`, um die Jobbezeichnung zu erhalten. JSON-P ähnelt als Vergleich in der Handhabung dem Umgang mit einer `Map` in Java&.

Angenommen, Sie möchten ein solches JSON-Objekt in Ihrem Java&trade;-Code erstellen. JSON-P verwendet ein Builder-Muster: Verwenden Sie einen `JsonObjectBuilder`, um jeden Wert hinzuzufügen, und rufen Sie dann `build ()` auf, um das `JsonObject` zu erzeugen. Beispiel:

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

Ein `JsonObject` in JSON-P ist nicht veränderbar (im Gegensatz zu JSON-B). Wenn Sie Felder aktualisieren möchten, müssen Sie ein neues `JsonObject` erstellen, die Felder duplizieren und dann Änderungen vornehmen. Die Erstellung eigener POJOs mit Kopierkonstruktoren kann hier hilfreich sein.

JSON-B und JSON-P werden vom MicroProfile-REST-Client für die ordnungsgemäße Serialisierung und Deserialisierung unterstützt; das Feedback zur Typsicherheit und Kompilierzeit machen JSON-B jedoch zur bevorzugten Wahl.
