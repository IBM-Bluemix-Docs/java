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

# Traitement de JSON avec JSON-P et JSON-B
{: #mp-json}

[JSON-B (JSON-Binding, JSR 367)](http://json-b.net/){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe") et [JSON-P (JSON-Processing, JSR 374)](https://javaee.github.io/jsonp/){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe") sont deux spécifications de l'API Java qui définissent comment les classes Java et les objets JSON peuvent interagir. JSON-P fournit une API Java pour le traitement des données au format JSON. JSON-B fournit une couche de liaison par dessus JSON-P, ce qui facilite la conversion des objets vers et depuis JSON. Dans la plupart des cas, il est préférable d'utiliser JSON-B plutôt que le niveau inférieur JSON-P.

## JSON-B
{: #java-json-b}

Avec **JSON-B**, la conversion vers et depuis JSON est réalisée à l'aide d'un objet Java simple (POJO) avec une méthode getter/setter pour chaque zone. Exemple :

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

Pour utiliser JSON-B avec Liberty, la fonction `jsonb-1.0` doit être activée dans votre fichier `server.xml`. Vous pouvez également utiliser `microProfile-2.0`, qui offre toutes les fonctions de MP, y compris JSON-B et JSON-P.

Dans votre classe JAX-RS, utilisez ce qui suit pour créer un objet Person avec les zones indiquées précédemment :

```java
Address myAddress = new Address("501-B101", "4205 S Miami Blvd", "Durham", "NC", "27703");
Employee me = new Employee("John Alcorn", 228264, "Senior Software Engineer", myAddress);
```
{: codeblock}

JSON-B convertira automatiquement JSON en un objet défini. Par exemple, si vous utilisez le client REST MicroProfile pour effectuer un appel sortant vers un service avec un objet POJO comme type de retour, JSON-B convertira automatiquement la réponse JSON en un objet POJO, que vous pouvez ensuite utiliser pour extraire des attributs :

```java
// mpRestClient: outbound client request returning a POJO
// JSON-B converts the JSON response into a Person object
Employee employee = myClient.get("John Alcorn");
int serial = employee.getSerial();
Address address = employee.getAddress();
String city = address.getCity();
```
{: codeblock}

JSON-B fournit également un analyseur qui prendra une représentation de chaîne d'un objet JSON et vous rendra l'objet JSON-B correspondant. Pour l'utiliser, vous devez ajouter les instructions d'importation suivantes à votre classe microservice :

```java
import javax.json.bind.Jsonb;
import javax.json.bind.JsonbBuilder;
```
{: codeblock}

Utilisez par exemple ce qui suit pour analyser une chaîne JSON stockée dans une variable `myJSON` :

```java
Jsonb jsonb = JsonbBuilder.create();
Employee employee = jsonb.fromJson(myJSON, Employee.class);
```
{: codeblock}

Tout est fortement typé avec JSON-B. Si vous obtenez un nom de zone erroné, par exemple, vous verrez un échec de compilation dans la mesure où cette méthode n'existe pas. La création d'objets POJO peut être fastidieuse, mais la plupart des IDE prennent en charge la génération de getters et setters une fois que les zones sont définies, ce qui est utile.

Evitez la surexposition des détails internes. Plusieurs annotations peuvent vous aider à contrôler comment vos objets sont convertis vers et depuis JSON : `@JsonbTransient` identifie les zones qui ne doivent pas être converties du tout, et `@JsonbNillable` précise comment les valeurs nulles doivent être traitées. Les implémentations `@JsonbAdapter` peuvent personnaliser davantage la manière dont les données sont sérialisées et désérialisées pour une isolation supplémentaire entre le format Wire et l'implémentation interne.
{: tip}

## JSON-P
{: #java-json-p}

Avant l'existence de JSON-B (qui faisait partie de Java EE 8), **JSON-P** (JSON-Processing) était un moyen standardisé d'interagir avec JSON dans le code Java. Auparavant, il existait plusieurs bibliothèques d'analyse JSON propriétaires, telles que la bibliothèque JSON4J d'IBM. JSON-P peut être utilisé pour analyser le JSON que vous avez reçu d'un appel REST, ou pour construire un JSON, auquel cas vous pouvez vous en servir pour analyser vos propres méthodes JAX-RS.

L'utilisation de JSON-P nécessite que la fonction `jsonp-1.0` soit activée dans votre fichier `server.xml` (ou la fonction `microProfile-2.0`, qui active toutes les technologies MP).

Les instructions d'importation suivantes permettent de travailler avec les tableaux et les objets JSON dans JSON-P :

```java
import javax.json.Json;
import javax.json.JsonArray;
import javax.json.JsonArrayBuilder;
import javax.json.JsonObject;
import javax.json.JsonObjectBuilder;
```
{: codeblock}

Pour travailler avec un JSON reçu d'un appel d'API REST, vous appelez la méthode `get` sur un `JsonObject`, en spécifiant la clé de la zone souhaitée. Pour reprendre notre exemple `Employee` ci-dessus, si vous avez reçu un `JsonObject` que vous avez appelé `employee`, vous appelez `employee.get("name")` pour connaître le nom de la personne ou `employee.get("title")` pour connaître son titre. A titre de comparaison, JSON-P ressemble beaucoup à un `Map` dans Java.

Maintenant, imaginons que vous voulez construire un objet JSON de ce type dans votre code Java. JSON-P utilise un modèle de constructeur : utilisez un `JsonObjectBuilder` pour ajouter chaque valeur, puis appelez `build()` pour produire le `JsonObject`, comme suit :

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

Notez que contrairement à JSON-B, un `JsonObject` n'est pas modifiable dans JSON-P. Si vous souhaitez mettre à jour des zones, vous devez créer un nouveau `JsonObject`, dupliquer les zones et effectuer vos modifications. Il peut être utile ici de créer vos propres objets Java simples (POJO) avec des constructeurs de copie.

JSON-B et JSON-P sont tous les deux pris en charge par le client REST MicroProfile pour une sérialisation et une désérialisation appropriées, mais le feedback obtenu en terme de cohérence des types et de temps de compilation fait de JSON-B la meilleure option.
