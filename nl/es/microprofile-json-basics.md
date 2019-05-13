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

# Manejo de JSON con JSON-P y JSON-B
{: #mp-json}

[JSON-B (JSON-Binding, JSR 367)](http://json-b.net/){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo") y [JSON-P (JSON-Processing, JSR 374)](https://javaee.github.io/jsonp/){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo") son dos especificaciones de API de Java que definen cómo pueden interactuar las clases Java y los objetos JSON. JSON-P proporciona una API de Java para procesar los datos con formato JSON. JSON-B proporciona una capa de enlace sobre JSON-P y facilita la conversión de objetos a JSON y desde JSON. En la mayoría de los casos, se debe utilizar JSON-B como preferencia ante JSON-P, de menor nivel.

## JSON-B
{: #java-json-b}

Con **JSON-B**, la conversión a y desde JSON se consigue utilizando un POJO (Plain Old Java Object) con un método get y set para cada campo.

Por ejemplo:
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

Para utilizar JSON-B con Liberty, debe tener la característica `jsonb-1.0` habilitada en el archivo `server.xml`. También puede utilizar `microProfile-2.0`, que le proporciona todas las características de MP, incluidos JSON-B y JSON-P.

En la clase JAX-RS, debe utilizar lo siguiente para crear un objeto `Person` con los campos mostrados anteriormente:

```java
Address myAddress = new Address("501-B101", "4205 S Miami Blvd", "Durham", "NC", "27703");
Employee me = new Employee("John Alcorn", 228264, "Senior Software Engineer", myAddress);
```
{: codeblock}

JSON-B convierte automáticamente JSON en un objeto definido. Por ejemplo, si utiliza el cliente REST de MicroProfile REST para realizar una llamada saliente a un servicio con un POJO como tipo de retorno, JSON-B convertirá automáticamente la respuesta JSON en un objeto POJO, que puede utilizar entonces para recuperar atributos:

```java
// mpRestClient: solicitud de cliente saliente que devuelve un POJO
// JSON-B convierte la respuesta JSON en un objeto de persona
Employee employee = myClient.get("John Alcorn");
int serial = employee.getSerial();
Address address = employee.getAddress();
String city = address.getCity();
```
{: codeblock}

JSON-B también proporciona un analizador que acepta una representación de serie de un objeto JSON y le devuelve el objeto JSON-B correspondiente. Para utilizar el analizador, debe añadir las siguientes sentencias de importación a la clase de microservicio:

```java
import javax.json.bind.Jsonb;
import javax.json.bind.JsonbBuilder;
```
{: codeblock}

Utilice algo parecido a lo siguiente para analizar una serie JSON almacenada en una variable `myJSON`:

```java
Jsonb jsonb = JsonbBuilder.create();
Employee employee = jsonb.fromJson(myJSON, Employee.class);
```
{: codeblock}

JSON-B cuenta con una definición estricta de tipos. Si se equivoca en un nombre de campo, por ejemplo, verá un error de compilación, ya que no existirá ese método. La creación de POJO puede ser tediosa, pero la mayoría de los IDE soportan la generación de métodos get y set una vez que se definen los campos, lo que sirve de ayuda.

Evite la sobreexposición de detalles internos. Hay varias anotaciones que le pueden ayudar a controlar cómo se convierten los objetos a y desde JSON: `@JsonbTransient` identifica los campos que no se deben convertir y `@JsonbNillable` especifica cómo gestionar los valores nulos. Con las implementaciones de `@JsonbAdapter` se puede personalizar cómo serializar y deserializar los datos para obtener un aislamiento añadido entre el formato de cable y la implementación interna.
{: tip}

## JSON-P
{: #java-json-p}

Antes de que existiera JSON-B (que llegó como parte de Java EE 8), **JSON-P** (JSON-Processing) era una forma estandarizada de interactuar con JSON dentro del código Java. Antes de eso, había varias bibliotecas de análisis JSON propietarias, como por ejemplo JSON4J de IBM. JSON-P se puede utilizar para analizar JSON que ha recibido de una llamada REST o para construir JSON que puede devolver desde sus propios métodos JAX-RS.

La utilización de JSON-P requiere que la característica `jsonp-1.0` esté habilitada en el `server.xml` (o en la característica `microProfile-2.0`, que habilita todas las tecnologías de MP).

A continuación, se muestran las sentencias de importación para trabajar con objetos JSON y matrices en JSON-P:

```java
import javax.json.Json;
import javax.json.JsonArray;
import javax.json.JsonArrayBuilder;
import javax.json.JsonObject;
import javax.json.JsonObjectBuilder;
```
{: codeblock}

Para trabajar con JSON recibido de una llamada de API REST, llame al método `get` en un `JsonObject` y pase la clave para el campo que desee. Para continuar con el ejemplo de `Employee`, si recibiese un `JsonObject` al que denominase `employee`, tendría que llamar a `employee.get("name")` para averiguar el nombre de la persona o a `employee.get("title")` para obtener su puesto de trabajo. JSON-P es muy parecido a tratar con un `Map` en Java, como una comparación.

Ahora, pongamos que desea crear dicho objeto JSON en el código Java. JSON-P utiliza un patrón de compilador: utilice un `JsonObjectBuilder` para añadir cada valor y, a continuación, llame a `build()` para producir el `JsonObject`, como por ejemplo:

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

Un `JsonObject` en JSON-P es inmutable, a diferencia de JSON-B. Si desea actualizar campos, debe crear un nuevo `JsonObject`, duplicar los campos y, a continuación, realizar cambios. En este caso, puede resultar útil hacer sus propios POJO con constructores de copia.

Tanto JSON-B como JSON-P son compatibles con el cliente Rest de MicroProfile para una serialización y deserialización adecuadas, pero la seguridad de tipo y la retroalimentación de tiempo de compilación hacen de JSON-B la opción preferida.
