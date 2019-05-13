---

copyright:
  years: 2019
lastupdated: "2019-04-22"

keywords: spring framework, spring reference, spring boot, boot actuator, spring kubernetes

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

# Spring
{: #spring-overview}

Spring 플랫폼은 Java&trade 애플리케이션을 쉽게 작성할 수 있게 하는 프로젝트의 에코시스템입니다. 대개 "Spring"이라는 용어는 Spring Framework를 나타내지만 일반적으로 그 일부인 프로젝트를 나타내거나 Spring Boot를 포함한 Spring 기반 기술을 사용하는 프로젝트를 나타내는 데 사용됩니다.

## Spring Framework
{: #spring-framework}

Spring Framework는 2002년에 출시되었으며 이후 약 3년마다 주 버전을 출시했습니다. 프레임워크는 REST 엔드포인트부터 데이터베이스 추상화까지 모든 것을 포함하는 대형 컴포넌트 세트(Spring 모듈이라고 함)로 구성됩니다. 2017년 최신 릴리스를 통해 Spring Framework 5에서는 업데이트된 JDK 지원 및 WebFlux를 소개했습니다. WebFlux는 [Project Reactor](https://projectreactor.io/){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")을 기반으로 하는 반응형 프로그래밍을 위한 새로운 모듈입니다.

Spring Framework 4.3은 Spring Framework 4.x의 최신 기능 분기이며 2020년까지 지원됩니다. 이전 버전의 Spring을 사용하는 애플리케이션은 Spring Framework 5로 마이그레이션해야 합니다.

Spring Framework에 대한 종합적인 문서는 다음과 같습니다.

* [5.1.x의 Spring Framework 참조 안내서](https://docs.spring.io/spring/docs/5.1.x/spring-framework-reference/){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")
* [4.3.x의 Spring Framework 참조 안내서](https://docs.spring.io/spring/docs/4.3.x/spring-framework-reference/){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")
* [Spring Framework 5.x로 업그레이드](https://github.com/spring-projects/spring-framework/wiki/Upgrading-to-Spring-Framework-5.x){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")

## Spring Boot
{: #spring-boot}

Spring Boot는 "컨테이너 없는 웹 애플리케이션 아키텍처를 개선"하기 위해 2014년에 출시되었습니다. XML의 구성을 코드로 옮기는 것이 목적이었습니다. Spring Boot는 사용할 기술에 대한 확고한 관점을 기반으로 애플리케이션을 작성하기 위한 메커니즘을 제공합니다. Spring Boot 애플리케이션은 Spring 애플리케이션이며 이 에코시스템에 고유한 프로그래밍 모델을 사용합니다.

Spring Boot는 구성에 대한 규약을 강조하고 어노테이션과 클래스 경로 검색의 조합을 사용하여 추가 기능을 사용으로 설정합니다. 기본적으로 Spring Boot 애플리케이션은 모든 필수 종속성(임베디드 Tomcat 서버 포함)이 포함된 독립적인 실행 가능 JAR 파일에 빌드됩니다. 애플리케이션은 애플리케이션 서버에 배치된 WAR 파일로 패키지될 수도 있습니다.

Spring Boot의 버전은 이제 2.0이며 2018년에 출시되었습니다. 1.5.x 분기의 유지보수는 2019년 8월에 중지됩니다.

Spring Boot에 대한 종합적인 문서는 다음과 같습니다.

* [2.1.x의 Spring Boot 참조 안내서](https://docs.spring.io/spring-boot/docs/2.1.x/reference/){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")
* [1.5.x의 Spring Boot 참조 안내서](https://docs.spring.io/spring-boot/docs/1.5.x/reference/){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")

## Spring Framework 또는 Spring Boot 선택
{: #spring-framework-or-boot}

새 클라우드 고유 애플리케이션에서 Spring Framework를 사용하도록 선택한 경우 Spring Boot도 사용해야 합니다. Spring Boot는 Spring 기반 애플리케이션의 구성과 어셈블리를 단순화하고 [클라우드 고유 애플리케이션](/docs/java?topic=cloud-native-overview#overview) 작성을 단순화하는 Spring Actuator와 같은 기능을 제공합니다.

### Spring Boot Actuator
{: #spring-boot-actuator}

Spring Boot Actuator는 실행 중인 Spring 앱을 검사하는 데 유용한 엔드포인트의 콜렉션을 정의합니다. 이를 사용으로 설정하려면 애플리케이션에 단일 종속성을 추가하면 됩니다.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```
{: codeblock}

Spring Boot Actuator의 구조 및 동작이 Spring Boot 1과 Spring Boot 2 사이에 크게 변경되었습니다. Boot 1 Actuator는 애플리케이션과 함께 제공되었으며 개별 구성 및 보안 메커니즘을 사용했습니다. Boot 2 버전은 향상되어 나머지 Spring 애플리케이션과 통합된 보안 모델을 사용합니다.

동작 변경은 특히 Boot 1 애플리케이션에서 Boot 2로 마이그레이션하는 경우와 관련이 있습니다. 세부사항은 [Boot 1에서 Boot 2로 마이그레이션 안내서](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.0-Migration-Guide#spring-boot-actuator){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")의 Actuator 절을 참조하십시오.
{: tip}

Boot 2에서 Actuator는 미니 REST 프레임워크로 작동합니다. 모든 엔드포인트는 `/actuator` 컨텍스트에 있습니다. 상태 검사 수행 또는 메트릭, 애플리케이션, 기타 환경 정보 수집을 위한 엔드포인트가 제공됩니다. 전체 목록은 [Actuator 문서](https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/html/production-ready-features.html#production-ready){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")를 참조하십시오.

Spring `@Component`를 사용하여 고유 Actuator를 쉽게 작성할 수 있습니다.

```java
@Endpoint(id="example")
@Component
public class Example {
    @ReadOperation
    public String example() {
        return "{\"example\":\"value\"}";
    }
}
```
{: codeblock}

Spring Boot에는 런타임 시 활성인 Actuator를 구성하는 두 개의 제어가 있으며, Actuator는 "사용 가능"하고 "노출"될 수 있습니다. 접속 가능하려면 Actuator가 사용 가능하고 노출되어야 합니다. 기본적으로 많은 엔드포인트가 사용 가능하지만 상태 및 정보만 노출됩니다. 또는 Spring 보안이 애플리케이션에 사용 가능한 경우 Actuator 엔드포인트에 대한 액세스가 명시적으로 허용되어야 합니다.

Spring Boot 1의 Actuator에 자체 보안 구성이 있으며 대개 application.properties에 구성되어 있습니다.  "사용 가능"과 "노출됨" 대신 "사용 가능"과 "민감함"이 있습니다. 민감한 엔드포인트에서는 HTTP를 통해 제공되는 경우 인증이 필요합니다. 기본적으로, Spring Boot Actuator의 /metrics 엔드포인트는 Spring Boot 1에서 사용 가능하지만, 이는 민감하다고 간주되므로 권한을 필요로 하거나 민감하지 않은 것으로 설정되어야 합니다. 일부 환경에서는 Spring 보안을 구성하는 것이 올바른 답일 수 있지만, Kubernetes에서는 메트릭 엔드포인트가 클러스터 내부에 남아 있습니다. 자세한 정보는 [Spring Boot 1 Actuator 문서](https://docs.spring.io/spring-boot/docs/1.5.2.RELEASE/reference/htmlsingle/#production-ready){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")를 참조하십시오.
{: note}

## Spring Cloud
{: #spring-cloud}

Spring Cloud는 서드파티 클라우드 기술과 Spring 프로그래밍 모델 간 통합의 콜렉션입니다. 이는 클라우드에 배치할 Spring 애플리케이션을 빌드하는 개발자를 지원하기 위해 제공됩니다. Spring Cloud 내 각 프로젝트는 특정 기술 또는 기술 세트에 중점을 두고 있으므로, Spring Cloud에서 얻을 수 있는 적용 범위는 모듈식 네이처를 통해 가능하게 됩니다.

Spring Cloud 프로젝트는 구성에 대해 기존에 사용된 일반적인 Spring 방법을 따릅니다. 대부분의 기능은 빌드 시 올바른 종속성을 추가하여 사용할 수 있습니다.

Spring Cloud 프로젝트 및 지원되는 기술에 대한 자세한 정보는 공식 [Spring Cloud 프로젝트 사이트](https://spring.io/projects/spring-cloud){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")를 참조하십시오.

### Kubernetes의 Spring Cloud
{: #spring-cloud-kubernetes}

Spring Cloud Kubernetes는 Kubernetes에서 실행되는 Spring 애플리케이션에 대해 다양한 태스크를 쉽게 수행할 수 있도록 하는 Spring Cloud 프로젝트입니다. Spring Cloud Kubernetes는 Kubernetes 개념과 서비스에 맵핑되는 일반 Spring 인터페이스의 구현을 제공합니다.

그동안 Spring은 Eureka와 같은 Netflix 라이브러리를 서비스 레지스트리로, Ribbon을 클라이언트 측 로드 밸런서로 사용했습니다. Kubernetes 환경에서는 이러한 역할이 모두 Kubernetes 자체에서 수행되어 애플리케이션 레벨 기능이 중복됩니다. Spring Cloud Kubernetes는 `DiscoveryClient`와 같은 Spring 추상화를 기본 Kubernetes 메커니즘에 맞게 조정합니다.

Istio가 사용되어 서비스 라우팅에 영향을 주는 경우 Istio가 설치된 클러스터에서 Spring Cloud Kubernetes를 통해 Ribbon 검색을 사용하게 되면 주의해야 합니다. 클라이언트 측 로드 밸런싱의 경우 클라이언트는 대상 서비스 자체를 선택하려고 하는 반면, Istio는 다른 위치에서 호출 라우팅을 시도할 수 있습니다. 이 중 하나만 가능하기 때문에 호출을 라우팅한 방법에 대한 의견 차이가 발생합니다. 가능한 한 이 조합은 피해야 합니다.
{: note}

또한 Spring Cloud Kubernetes는 Kubernetes 구성 맵과 시크릿 간 통합을 Spring `@Autowired` 구성 Bean에 제공합니다. 여기에는 애플리케이션 실행 중에 작성된 `ConfigMaps` 및 `Secrets`에 대한 동적 변경을 처리하기 위한 전략이 포함됩니다.

마지막으로 Spring Cloud Kubernetes는 배치와 관련된 추가 정보를 포함하도록 기본 Spring Boot Actuator 상태 엔드포인트를 보강합니다.

자세한 정보는 [Spring Cloud Kubernetes 문서](https://cloud.spring.io/spring-cloud-static/spring-cloud-kubernetes/2.1.0.RC1/single/spring-cloud-kubernetes.html){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")를 참조하십시오.


<!--
### Spring Cloud Streams
{: #spring-cloud-streams}


:FIXME:
-->
