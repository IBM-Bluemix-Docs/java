---

copyright:
  years: 2018, 2019
lastupdated: "2019-04-09"

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

# Spring 환경 구성
{: #spring-configuration}

서비스 구성 및 인증 정보(서비스 바인딩)의 관리는 플랫폼마다 다릅니다. Cloud Foundry는 `VCAP_SERVICES` 환경 변수로 애플리케이션에 전달되는 문자열로 변환된 JSON 오브젝트에 서비스 바인딩 세부사항을 저장합니다. Kubernetes는 `ConfigMaps` 또는 `Secrets`에 문자열로 변환된 JSON 또는 일반 속성으로 서비스 바인딩을 저장하며, 이는 컨테이너화된 애플리케이션에 환경 변수로 전달되거나 임시 볼륨으로 마운트될 수 있습니다. 로컬 개발은 다를 수 있습니다. 환경에 특정한 코드 경로 없이 이식 가능한 방식으로 이러한 변형에 대해 작업하는 것은 어려울 수 있습니다.

## 기존 Spring 애플리케이션에 {{site.data.keyword.cloud_notm}} 구성 추가
{: #spring-cloud-env}

[{{site.data.keyword.cloud_notm}} Spring Service Bindings](https://github.com/ibm-developer/ibm-cloud-spring-bind){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘") 라이브러리는 로컬, Cloud Foundry, Cloud Foundry Enterprise Environment, Kubernetes, 가상 머신 및 {{site.data.keyword.openwhisk}}를 포함한 여러 클라우드 환경의 정보를 `mappings.json` 파일의 검색 패턴 배열로 추상화하여 Spring 애플리케이션이 구성에 액세스하는 일관된 방식을 제공합니다. Spring 애플리케이션이 시작되면 라이브러리가 파일을 읽고 패턴을 적용하여 구성 값을 검색하고 지정합니다.

`mappings.json` 파일에 대한 자세한 정보는 [삽입된 서비스 인증 정보 관련 작업](/docs/java?topic=cloud-native-configuration#portable-credentials)을 참조하십시오.

### Spring 애플리케이션에 라이브러리 추가
{: #spring-add-service-library}

{{site.data.keyword.cloud_notm}} Spring Service Bindings 라이브러리를 사용하려면 `pom.xml` 또는 `build.gradle` 파일에 `ibm-cloud-spring-boot-service-bind` 종속성을 포함시키십시오.

**Maven 예**:

```xml
<dependency>
   <groupId>com.ibm.cloud</groupId>
   <artifactId>ibm-cloud-spring-boot-service-bind</artifactId>
   <version>1.1.1</version>
</dependency>
```
{: codeblock}

**Gradle 예**:

```json
dependencies {
    compile group: 'com.ibm.cloud', name: 'ibm-cloud-spring-boot-service-bind', version: '1.1.1'
}
```
{: codeblock}

### 인증 정보에 액세스
{: #spring-access-credentials}

코드의 서비스 구성에 액세스하려면 `@Value` 어노테이션을 사용하거나 Spring 프레임워크 Environment 클래스 `getProperty()` 메소드를 사용할 수 있습니다.

예: IBM Cloudant NoSQL DB 서비스에 대한 구성 데이터 액세스:

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

## 다음 단계
{: #spring-config-next-steps notoc}

Spring Boot에 대한 자세한 정보는 다음을 참조하십시오.

* [Spring Boot 1.5.x](https://docs.spring.io/spring-boot/docs/1.5.x/reference/html/){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")
* [Spring Boot 2.1.x](https://docs.spring.io/spring-boot/docs/2.1.x/reference/html/){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")

IBM 및 Spring에 대해 자세히 보려면 다음을 참조하십시오.

* [IBM Developer: Spring](https://developer.ibm.com/technologies/spring/){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")
* [IBM Spring Service Bindings](https://github.com/ibm-developer/ibm-cloud-spring-bind){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")
