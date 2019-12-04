---

copyright:
  years: 2018, 2019
lastupdated: "2019-12-04"

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

# Configuring the Spring environment
{: #spring-configuration}

Management of service configuration and credentials (service bindings) varies between platforms. Cloud Foundry stores service binding details in a stringified JSON object that is passed to the application as a `VCAP_SERVICES` environment variable. Kubernetes stores service bindings as stringified JSON or flat attributes in `ConfigMaps` or `Secrets`, which can either be passed to the containerized app as environment variables or mounted as temporary volumes. Local development can vary. Working across these variations in a portable way without having environment-specific code paths can be challenging.

## Adding {{site.data.keyword.cloud_notm}} configuration to existing Spring apps
{: #spring-cloud-env}

The [{{site.data.keyword.cloud_notm}} Spring Service Bindings](https://github.com/ibm-developer/ibm-cloud-spring-bind){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon") library provides a consistent way for your Spring app to access configuration by abstracting information from different cloud environments. Which includes local, Cloud Foundry, Cloud Foundry Enterprise Environment, Kubernetes, and OpenShift into an array of search patterns in a `mappings.json` file. When the Spring app starts, the library reads the file and applies the patterns to discover and assign configuration values.

For more information about the `mappings.json` file, see [Working with injected service credentials](/docs/java?topic=cloud-native-configuration#portable-credentials).

### Adding the library to your Spring app
{: #spring-add-service-library}

To use the {{site.data.keyword.cloud_notm}} Spring Service Bindings library, include the `ibm-cloud-spring-boot-service-bind` dependency in your `pom.xml` or `build.gradle` file:

**Maven example**:

```xml
<dependency>
   <groupId>com.ibm.cloud</groupId>
   <artifactId>ibm-cloud-spring-boot-service-bind</artifactId>
   <version>1.1.1</version>
</dependency>
```
{: codeblock}

**Gradle example**:

```json
dependencies {
    compile group: 'com.ibm.cloud', name: 'ibm-cloud-spring-boot-service-bind', version: '1.1.1'
}
```
{: codeblock}

### Accessing credentials
{: #spring-access-credentials}

To access service configuration in your code you can use the `@Value` annotation, or use the Spring framework Environment class `getProperty()` method.

Example: Accessing configuration data for an IBM Cloudant NoSQL DB service:

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

## Next Steps
{: #spring-config-next-steps notoc}

For more information on Spring Boot:

* [Spring Boot 1.5.x](https://docs.spring.io/spring-boot/docs/1.5.x/reference/html/){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon")
* [Spring Boot 2.1.x](https://docs.spring.io/spring-boot/docs/2.1.x/reference/html/){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon")

To learn more about {{site.data.keyword.IBM}} and Spring:

* [IBM Developer: Spring](https://developer.ibm.com/technologies/spring/){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon")
* [IBM Spring Service Bindings](https://github.com/ibm-developer/ibm-cloud-spring-bind){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon")
