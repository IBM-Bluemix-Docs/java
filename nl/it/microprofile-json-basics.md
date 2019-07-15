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

# Gestione di JSON con JSON-P e JSON-B
{: #mp-json}

[JSON-B (JSON-Binding, JSR 367)](http://json-b.net/){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno") e [JSON-P (JSON-Processing, JSR 374)](https://javaee.github.io/jsonp/){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno") sono due specifiche API Java&trade che definiscono in che modo le classi Java&trade e gli oggetti JSON possono interagire. JSON-P fornisce un'API Java&trade per elaborare i dati con formattazione JSON. JSON-B fornisce un livello di binding in aggiunta a JSON-P, rendendo più facile la conversione di oggetti in e da JSON. Nella maggior parte dei casi, JSON-B deve essere utilizzato preferendolo al JSON-P di livello più basso.

## JSON-B
{: #java-json-b}

Con **JSON-B** , la conversione in e da JSON viene ottenuta utilizzando un POJO (Plain Old Java&trade; Object) con un metodo getter e sender per ciascun campo. 

Ad esempio:
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

Per utilizzare JSON-B con Liberty, hai bisogno della funzione `jsonb-1.0` abilitata nel tuo file `server.xml`. Puoi anche utilizzare `microProfile-2.0`, che ti offre tutte le funzioni MP, compresi JSON-B e JSON-P.

Nella tua classe JAX-RS, utilizzi quanto segue per creare un oggetto `Person` con i campi mostrati in precedenza.

```java
Address myAddress = new Address("501-B101", "4205 S Miami Blvd", "Durham", "NC", "27703");
Employee me = new Employee("John Alcorn", 228264, "Senior Software Engineer", myAddress);
```
{: codeblock}

JSON-B converte automaticamente JSON in un oggetto definito. Ad esempio, se utilizzi il client REST MicroProfile per eseguire una chiamata in uscita a un servizio con un POJO, come tipo di restituzione, JSON-B converte automaticamente la risposta JSON in un oggetto POJO, che puoi quindi utilizzate per richiamare gli attributi.

```java
// mpRestClient: outbound client request returning a POJO
// JSON-B converts the JSON response into a Person object
Employee employee = myClient.get("John Alcorn");
int serial = employee.getSerial();
Address address = employee.getAddress();
String city = address.getCity();
```
{: codeblock}

JSON-B fornisce anche un parser che prende una rappresentazione stringa di un oggetto JSON e ti restituisce l'oggetto JSON-B corrispondente. Per utilizzare il parser, devi aggiungere le seguenti istruzioni di importazione alla tua classe di microservizi:

```java
import javax.json.bind.Jsonb;
import javax.json.bind.JsonbBuilder;
```
{: codeblock}

Utilizza qualcosa di simile a quanto di seguito indicato per analizzare una stringa JSON memorizzata in una variabile `myJSON`:

```java
Jsonb jsonb = JsonbBuilder.create();
Employee employee = jsonb.fromJson(myJSON, Employee.class);
```
{: codeblock}

Tutto è fortemente tipizzato con JSON-B. Se ottieni un nome di campo errato, ad esempio, vedrai un errore di compilazione poiché tale metodo non esiste. La creazione di POJO può essere tediosa ma la maggior parte degli IDE supporta la generazione di getter e setter dopo che i campi sono stati definiti, il che aiuta.

Evita una sovraesposizione di dettagli interni. Diverse annotazioni possono aiutarti a controllare in che modo i tuoi oggetti vengono convertiti in e da JSON: `@JsonbTransient` identifica i campi che non devono essere convertiti affatto e `@JsonbNillable` specifica come devono essere gestiti i valori null. Le implementazioni `@JsonbAdapter` possono personalizzare ulteriormente in che modo i dati vengono serializzati e deserializzati per un ulteriore isolamento tra il formato wire e l'implementazione interna.
{: tip}

## JSON-P
{: #java-json-p}

Prima che esistesse JSON-B (che è arrivato come parte di Java&trade; EE 8), **JSON-P** (JSON-Processing) era un modo standardizzato per interagire con JSON all'interno del codice Java&trade;. Prima di questo, c'erano diverse librerie di analisi JSON proprietarie, come ad esempio JSON4J di IBM. JSON-P può essere utilizzato per analizzare il JSON ricevuto da una chiamata REST o per creare il JSON che puoi restituire dai tuoi metodi JAX-RS.

L'utilizzo di JSON-P richiede che la funzione `jsonp-1.0` sia abilitata nel tuo `server.xml` (oppure la comoda funzione `microProfile-2.0`, che abilita tutte le tecnologie MP).

Queste sono le istruzioni di importazione per lavorare con array e oggetti JSON in JSON-P:

```java
import javax.json.Json;
import javax.json.JsonArray;
import javax.json.JsonArrayBuilder;
import javax.json.JsonObject;
import javax.json.JsonObjectBuilder;
```
{: codeblock}

Per lavorare con il JSON ricevuto da una chiamata API REST, richiami il metodo `get` su un `JsonObject`, passando la chiave per il campo che desideri. Per continuare l'esempio `Employee`, se hai ricevuto un `JsonObject` che hai denominato `employee`, richiamerai `employee.get("name")` per rilevare il nome della persona oppure `employee.get("title")` per ottenere la sua posizione professionale. JSON-P è molto simile a un `Map` in Java&trade, come paragone.

Se vuoi creare un tale oggetto JSON nel tuo codice Java&trade;. JSON-P utilizza un modello di builder: utilizza un `JsonObjectBuilder` per aggiungere ciascun valore e richiama quindi `build()` per produrre il `JsonObject`, in questo modo:

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

A differenza di JSON-B, un `JsonObject` in JSON-P è immutabile. Se vuoi aggiornare i campi, devi creare un nuovo `JsonObject`, duplicare i campi e apportare quindi le modifiche. Creare dei tuoi POJO con i constructor di copia può essere di aiuto, qui.

JSON-B e JSON-P sono entrambi supportati dal client REST MicroProfile per una corretta serializzazione e deserializzazione, ma l'indipendenza dai tipi e il feedback in fase di compilazione fanno di JSON-B la scelta preferita.
