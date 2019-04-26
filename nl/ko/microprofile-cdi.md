---

copyright:
  years: 2019
lastupdated: "2019-03-15"

keywords: dependency injection, cdi jakarta, cdi beans, cdi scopes, bean lifecycle, context injection microprofile, microprofile cdi

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

# CDI(Contexts and Dependency Injection), JSR-365
{: #mp-cdi}

CDI는 다양한 컴포넌트를 연결하는 데 사용되므로 Jakarta-EE 및 MicroProfile 모두에서 중심 요소입니다. CDI는 **컨텍스트**를 정의하여 Bean 라이프사이클을 지정하고 관리하며 **종속성 인젝션**을 사용하여 동적이며 형식 변환을 허용하지 않는 방식으로 선언된 종속성을 충족시킵니다. 또한 CDI는 느슨하게 결합된 이벤트와 인터셉터 모델을 사용하며, 다른 프레임워크에 대한 강력하고 유연하며 어디서든 실행 가능한 확장 메커니즘을 제공하여 자체 CDI Bean을 정의하거나 기존 컴포넌트를 업데이트합니다.

MicroProfile 스펙(예: MicroProfile Config, MicroProfile JWT 등)은 CDI 확장 메커니즘에 의존하여 새 기능으로 JAX-RS와 같은 기존 표준을 개선하는 CDI 우선 방식을 사용합니다. CDI 1.2는 MicroProfile 1.0 이후 릴리스의 일부입니다(MicroProfile 2.0은 CDI 2.0으로 이동함).

## CDI Bean
{: #cdi-bean}

CDI는 _Bean_에서 작동합니다. CDI Bean은 [Bean 정의 어노테이션](https://docs.jboss.org/cdi/spec/2.0/cdi-spec.html){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")을 사용하여 작성됩니다. 인수가 없는 생성자 또는 `@Inject` 어노테이션이 있는 생성자를 포함한 거의 모든 POJO(Plain Old Java Object)는 Bean이 될 수 있습니다.

CDI Bean 정의 어노테이션을 발견해야 합니다. JAR 아카이브의 `META-INF` 폴더 또는 WAR 아카이브의 `WEB-INF` 폴더에서 `beans.xml` 파일을 사용하여 CDI 어노테이션 스캔을 사용으로 설정할 수 있습니다. 이 파일은 비어 있거나 다음 컨텐츠와 같은 내용을 포함할 수 있습니다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://xmlns.jcp.org/xml/ns/javaee"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
            http://xmlns.jcp.org/xml/ns/javaee/beans_1_1.xsd"
      version="1.1" bean-discovery-mode="all">
  // enable interceptors; decorators; alternatives
</beans>
```
{: codeblock}

빈 `beans.xml` 또는 `bean-discovery-mode="all"`이 포함된 `beans.xml`이 있으면 모두 잠재적 오브젝트 CDI Bean이 됩니다. 그렇지 않으면 CDI Bean 정의 어노테이션이 있는 오브젝트만 CDI Bean이 됩니다.

## CDI 범위
{: #cdi-scopes}

CDI 범위 유형 어노테이션은 다음과 같이 Bean의 라이프사이클을 제어합니다.

* 애플리케이션이 실행되는 동안 인스턴스가 활성 상태여야 하는 경우 `@ApplicationScoped` 클래스 레벨 어노테이션을 사용합니다.
* 모든 요청(예: Servlet 요청)에 대해 Bean의 새 인스턴스를 작성해야 하는 경우 `@RequestScoped` 클래스 레벨 어노테이션을 사용합니다.
* 새 인스턴스가 다른 오브젝트에 속해야 하는 경우 `@Dependent` 클래스 레벨 어노테이션을 사용합니다. 종속 Bean의 인스턴스는 서로 다른 클라이언트 간에 또는 서로 다른 인젝션 지점 간에 공유되지 않습니다. 이 인스턴스는 자신이 속하는 오브젝트가 작성될 때 인스턴스화되고 해당 오브젝트가 영구 삭제될 때 영구 삭제됩니다.
* 클래스 레벨 어노테이션을 정의하지 않은 경우 `@Dependent`가 기본값입니다.

CDI 컨테이너는 정의된 범위에 따라 Bean 인스턴스를 작성하고 영구 삭제하며 해당 인스턴스를 적절한 컨텍스트와 연관시킨 다음 다른 오브젝트의 종속성으로 삽입합니다.

## CDI Bean 인젝션
{: #cdi-inject}

CDI는 `@Inject`를 통해 다른 컴포넌트에 정의된 Bean을 삽입합니다. 예를 들어, 다음 POJO `MyBean`은 CDI Bean입니다.

```java
@ApplicationScoped
public class MyBean {
    int i=0;
    public String sayHello() {
        return "MyBean hello " + i++;
    }
}
```
{: codeblock}

이는 `@ApplicationScoped`이므로 하나의 인스턴스만 작성됩니다. 다음에서 `MyRestEndPoint`는 `@RequestScoped`이며, 이는 각 요청에 대해 인스턴스가 작성됨을 의미합니다. CDI는 각각에 동일한 `MyBean` 인스턴스를 삽입합니다.

```java
@RequestScoped
@Path("/hello")
public class MyRestEndPoint {
    @Inject MyBean bean;

    @GET
    @Produces (MediaType.TEXT_PLAIN)
    public String sayHello() {
        return bean.sayHello();
    }
}
```
{: codeblock}

CDI는 다음과 같이 이러한 Bean의 라이프사이클을 관리합니다.

* 생성자 대신 `@PostConstruct`라는 어노테이션이 있는 메소드를 사용하여 Bean을 초기화합니다. 이는 CDI 컨테이너가 종속성을 삽입하고 적절한 컨텍스트에 Bean을 연관시킨 후에 호출됩니다.
* `@PreDestroy`라는 어노테이션이 있는 메소드를 사용하여 Bean을 정리합니다. 이 메소드는 종속성을 제거하기 전에 호출됩니다.
