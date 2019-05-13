---

copyright:
  years: 2019
lastupdated: "2019-04-22"

keywords: microprofile api, microprofile cdi, microprofile config api, config api, store properties multiple sources

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

# MicroProfile을 사용한 구성
{: #mp-configuration}

MicroProfile Config API를 사용하면 애플리케이션 구성 특성을 여러 소스에 저장할 수 있으며 이를 통해 사용자는 환경 특정 코드 경로 없이 코드를 작성할 수 있습니다. API는 [CDI](/docs/java?topic=java-mp-cdi#mp-cdi)를 사용하여 값 지정 특성을 애플리케이션에 삽입합니다.

## 선행 조건
{: #mp-config-prereq}

애플리케이션에서 MicroProfile Config API를 사용하려면 다음 종속성을 `pom.xml` 파일에 추가해야 합니다.

```xml
<dependency>
  <groupId>org.eclipse.microprofile</groupId>
  <artifactId>microprofile</artifactId>
  <version>2.1</version>
  <type>pom</type>
  <scope>provided</scope>
</dependency>
```
{: codeblock}

## 구성 인젝션
{: #mp-config-inject}

CDI Bean 클래스 내에서 다음 가져오기를 사용하십시오.

```java
//CDI 1.2
import javax.inject.Inject;

//mpConfig 1.2
import org.eclipse.microprofile.config.inject.ConfigProperty;
```
{: codeblock}

`WATSON_URL`이라는 특성의 값을 `watsonService` 변수에 삽입하려면 다음 예를 참조하십시오.

```java
@Inject 
@ConfigProperty(name = "WATSON_URL") 
String watsonService;
```
{: codeblock}

또한 이름 지정된 특성을 찾을 수 없는 경우에 사용할 기본값을 지정할 수 있습니다.

```java
@Inject 
@ConfigProperty(name = "WATSON_URL", defaultValue = "https://gateway.watsonplatform.net/tone-analyzer/api/v3/tone?version=2017-09-21") 
String watsonService;
```
{: codeblock}

설정이 완료되었습니다.

MicroProfile Config에 대한 자세한 정보는 다음을 참조하십시오.

* [Eclipse MicroProfile Config – what is it?](https://www.eclipse.org/community/eclipse_newsletter/2017/september/article3.php){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")
* OpenLiberty 안내서: [Configuring microservices](https://openliberty.io/guides/microprofile-config.html){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")
