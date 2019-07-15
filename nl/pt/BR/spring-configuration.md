---

copyright:
  years: 2018, 2019
lastupdated: "2019-06-10"

keywords: spring environment, spring credentials, ibm-cloud-spring-boot-service-bind, service bindings spring, vcap_services spring, access credential spring

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

# Configurando o ambiente Spring
{: #spring-configuration}

O gerenciamento de configuração de serviço e de credenciais (ligações de serviço) varia entre as plataformas. O Cloud Foundry armazena detalhes de ligação de serviço em um objeto JSON em sequência que é passado para o aplicativo como uma variável de ambiente `VCAP_SERVICES`. O Kubernetes armazena ligações de serviço como atributos de JSON em sequência ou atributos simples em `ConfigMaps` ou `Secrets`, o que pode ser passado para o app conteinerizado como variáveis de ambiente ou montado como volumes temporários. O desenvolvimento local pode variar. Trabalhar por essas variações de uma maneira móvel sem ter caminhos de código específicos do ambiente pode ser desafiador.

## Incluindo a configuração do {{site.data.keyword.cloud_notm}} em apps do Spring existentes
{: #spring-cloud-env}

A biblioteca do [{{site.data.keyword.cloud_notm}} Spring Service Bindings](https://github.com/ibm-developer/ibm-cloud-spring-bind){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") fornece uma maneira consistente para que o seu app do Spring acesse a configuração, abstraindo informações de diferentes ambientes de nuvem. O que inclui o Cloud Foundry, o Cloud Foundry Enterprise Environment, o Kubernetes, as máquinas virtuais e o {{site.data.keyword.openwhisk}} locais em uma matriz de padrões de procura em um arquivo `mappings.json`. Quando o app do Spring iniciar, a biblioteca lerá o arquivo e aplicará os padrões a fim de descobrir e designar valores de configuração.

Para obter mais informações sobre o arquivo `mappings.json`, consulte [Trabalhando com credenciais de serviço injetadas](/docs/java?topic=cloud-native-configuration#portable-credentials).

### Incluindo a biblioteca em seu app do Spring
{: #spring-add-service-library}

Para usar a biblioteca de Ligações de Serviços do {{site.data.keyword.cloud_notm}} Spring, inclua a dependência `ibm-cloud-spring-boot-service-bind` em seu arquivo `pom.xml` ou `build.gradle`:

**Exemplo de Maven**:

```xml
<dependency>
   <groupId>com.ibm.cloud</groupId>
   <artifactId>ibm-cloud-spring-boot-service-bind</artifactId>
   <version>1.1.1</version>
</dependency>
```
{: codeblock}

** Exemplo de Gradle **:

```json
dependencies {
    compile group: 'com.ibm.cloud', name: 'ibm-cloud-spring-boot-service-bind', version: '1.1.1'
}
```
{: codeblock}

### Acessando as credenciais
{: #spring-access-credentials}

Para acessar a configuração de serviço em seu código, é possível usar a anotação `@Value` ou usar o método `getProperty()` da classe de ambiente Spring Framework.

Exemplo: acessando dados de configuração para um serviço IBM Cloudant NoSQL DB:

```java
   // Use the @Value annotation 
   @Value("cloudant.url")
   String cloudantUrl;

   // Use the Spring Environment object 
   @Autowired
   Environment env;

   String cloudantUsername = 
        env.getProperty("cloudant.username");
```
{: codeblock}

## Próximas etapas
{: #spring-config-next-steps notoc}

Para obter mais informações sobre o Spring Boot:

* [Spring Boot 1.5.x](https://docs.spring.io/spring-boot/docs/1.5.x/reference/html/){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo")
* [Spring Boot 2.1.x](https://docs.spring.io/spring-boot/docs/2.1.x/reference/html/){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo")

Para saber mais sobre o {{site.data.keyword.IBM}} e o Spring:

* [IBM Developer: Spring](https://developer.ibm.com/technologies/spring/){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo")
* [Ligações do serviço IBM Spring](https://github.com/ibm-developer/ibm-cloud-spring-bind){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo")
