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

# Manipulação JSON com JSON-P e JSON-B
{: #mp-json}

[JSON-B (JSON-Binding, JSR 367)](http://json-b.net/){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") e [JSON-P (JSON-Processing, JSR 374)](https://javaee.github.io/jsonp/){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") são duas especificações de API Java que definem como as classes Java e os objetos JSON podem interagir. O JSON-P fornece uma API Java para processamento de dados formatados por JSON. O JSON-B fornece uma camada de ligação sobre o JSON-P, tornando mais fácil converter objetos para e por meio do JSON. Na maioria dos casos, o JSON-B deve ser usado em preferência ao JSON-P de nível inferior.

## JSON-B
{: #java-json-b}

Com o **JSON-B**, a conversão para e por meio de JSON é alcançada usando um Plain Old Java Object (POJO) com um método getter/setter para cada campo. Por exemplo:

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

Para usar o JSON-B com o Liberty, é necessário o recurso `jsonb-1.0` ativado em seu `server.xml`. Também é possível usar o `microProfile-2.0`, que fornece todos os recursos do MP, incluindo JSON-B e JSON-P.

Em sua classe JAX-RS, você usaria o seguinte para criar um objeto Pessoal com os campos mostrados anteriormente:

```java
Address myAddress = new Address("501-B101", "4205 S Miami Blvd", "Durham", "NC", "27703");
Employee me = new Employee("John Alcorn", 228264, "Senior Software Engineer", myAddress);
```
{: codeblock}

O JSON-B converterá JSON automaticamente em um objeto definido. Por exemplo, se você usar o cliente REST do MicroProfile para fazer uma chamada de saída para um serviço com um POJO como um tipo de retorno, o JSON-B converterá automaticamente a resposta JSON em um objeto POJO, que você pode usar para recuperar atributos:

```java
// mpRestClient: outbound client request returning a POJO
// JSON-B converts the JSON response into a Person object
Employee employee = myClient.get("John Alcorn");
int serial = employee.getSerial();
Address address = employee.getAddress();
String city = address.getCity();
```
{: codeblock}

O JSON-B também fornece um analisador que obterá uma representação do String de um objeto JSON e fornecerá de volta o objeto JSON-B correspondente. Para usar isso, é necessário incluir as instruções de importação a seguir em sua classe de microsserviço:

```java
import javax.json.bind.Jsonb;
import javax.json.bind.JsonbBuilder;
```
{: codeblock}

Use algo como o seguinte para analisar uma sequência JSON armazenada em uma variável `myJSON`:

```java
Jsonb jsonb = JsonbBuilder.create();
Employee employee = jsonb.fromJson(myJSON, Employee.class);
```
{: codeblock}

Tudo é fortemente digitado com JSON-B. Se você obtivesse um nome de campo incorreto, por exemplo, você veria uma falha de compilação, uma vez que não haveria tal método. A criação de POJOs pode ser tediosa, mas a maioria dos IDEs suporta a geração de getters e setters quando os campos são definidos, o que ajuda.

Evite a exposição excessiva dos detalhes internos. Várias anotações podem ajudá-lo a controlar como seus objetos são convertidos para e de JSON: `@JsonbTransient` identifica campos que não devem ser convertidos de forma alguma, e `@JsonbNillable` especifica como os valores nulos devem ser manipulados. As implementações `@JsonbAdapter` podem customizar ainda mais como os dados são serializados e desserializados para isolamento adicional entre o formato de ligação e a implementação interna.
{: tip}

## JSON-P
{: #java-json-p}

Antes da existência do JSON-B (que chegou como parte do Java EE 8), o **JSON-P** (JSON-Processing) era uma maneira padronizada de interagir com o JSON dentro do código Java. Antes disso, havia várias bibliotecas de análise JSON proprietárias, como o JSON4J da IBM. O JSON-P pode ser usado para analisar JSON que você recebeu de uma chamada REST ou para construir o JSON que pode ser retornado por meio de seus próprios métodos JAX-RS.

O uso do JSON-P requer que o recurso `jsonp-1.0` esteja ativado em seu `server.xml` (ou no recurso de conveniência `microProfile-2.0`, que ativa todas as tecnologias MP).

Aqui estão as instruções de importação para trabalhar com objetos JSON e matrizes em JSON-P:

```java
import javax.json.Json;
import javax.json.JsonArray;
import javax.json.JsonArrayBuilder;
import javax.json.JsonObject;
import javax.json.JsonObjectBuilder;
```
{: codeblock}

Para trabalhar com JSON recebido de uma chamada de API de REST, você chama o método `get` em um `JsonObject`, transmitindo a chave para o campo que você deseja. Para continuar nosso exemplo de `Employee` acima, se você recebeu um `JsonObject` que você chamou de `employee`, você chamaria `employee.get("name")` para descobrir o nome da pessoa ou `employee.get("title")` para obter o cargo. O JSON-P é muito semelhante a lidar com um `Map` em Java, como uma comparação.

Agora, digamos que você deseja construir esse objeto JSON em seu código Java. O JSON-P usa um padrão de construtor: use um `JsonObjectBuilder` para incluir cada valor e, em seguida, chame `build()` para produzir o `JsonObject`, como:

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

Observe que um `JsonObject` no JSON-P é imutável, ao contrário do JSON-B. Se você desejar atualizar os campos, terá que criar um novo `JsonObject`, duplicar os campos e, em seguida, fazer mudanças. Fazer seus próprios POJOs com construtores de cópia pode ajudar aqui.

JSON-B e JSON-P são ambos suportados pelo MicroProfile Rest Client para serialização e desserialização apropriadas, mas o feedback de tipo de segurança e de tempo de compilação torna o JSON-B a opção preferencial.
