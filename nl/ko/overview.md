---

copyright:
  years: 2019
lastupdated: "2019-04-01"

keywords: openjdk, open j9, download openjdk, java frameworks, adoptopenjdk, eclipse openj9, openj9 binaries, openjdk binaries, microprofile framework, jakarta

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

# {{site.data.keyword.cloud_notm}}의 Java
{: #overview}

Java&trade; 애플리케이션은 독립형 애플리케이션에서 Java EE/Jakarta EE, MicroProfile 또는 Spring 애플리케이션까지 모든 모양과 크기로 나타납니다.

## OpenJ9을 포함한 OpenJDK
{: #openjdk}

OpenJDK&trade; 프로젝트는 Java 언어의 오픈 소스 참조 구현이며 사용자 프로그램에 대한 API 안정성과 호환성을 제공합니다. [Java SE 플랫폼](https://docs.oracle.com/javase/8/docs/){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")은 API와 JVM(Java Virtual Machine)으로 구성됩니다. JVM은 Java 애플리케이션을 실행하는 실행 엔진입니다.

[Eclipse OpenJ9 프로젝트](https://www.eclipse.org/openj9/index.html){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")는 OpenJDK에서 핫스팟 JVM을 대체하는 JVM을 제공합니다. OpenJ9 JVM은 2017년 9월에 {{site.data.keyword.IBM}}에서 오픈 소스로 공개되었지만 해당 계보는 훨씬 더 거슬러 올라갑니다. OpenJ9은 10년 넘게 {{site.data.keyword.appserver_short}}를 포함한 3000개가 넘는 {{site.data.keyword.IBM}} 제품에 사용되어 왔습니다.

OpenJDK와 Eclipse OpenJ9은 기존 핫스팟 기반 Java 런타임 외에 클라우드 고유 Java 애플리케이션에서 성능과 서비스 가용성을 향상시킵니다.

### Open9을 포함한 OpenJDK 다운로드
{: #downloads}

OpenJ9을 포함한 OpenJDK 바이너리는 [AdoptOpenJDK](https://adoptopenjdk.net/releases.html?variant=openjdk8&jvmVariant=openj9){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")의 Java 커뮤니티 허브에서 무료로 사용 가능합니다. AdoptOpenJDK는 빌드 스크립트와 인프라의 완전한 오픈 소스 세트에서 사전 빌드된 OpenJDK 바이너리를 제공합니다. 또한 Docker 이미지는 [Dockerhub](https://hub.docker.com/u/adoptopenjdk){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")에서 사용 가능합니다.

## Java 프레임워크
{: #frameworks}

OpenJDK와 OpenJ9은 모든 Java 애플리케이션의 견고한 기반을 제공하지만 애플리케이션 프레임워크는 일반적인 문제를 해결하는 데 자주 사용됩니다. 예를 들어, 엔터프라이즈 애플리케이션은 일반적으로 [Java EE(및 Jakarta EE)](/docs/java?topic=java-jee-overview#jakarta-ee) 또는 [Spring](/docs/java?topic=java-spring-overview)을 사용하여 빌드됩니다. [MicroProfile](/docs/java?topic=java-jee-overview#microprofile)은 개발 속도를 높이고 Java EE에 대한 클라우드 고유 개념을 지원합니다.

[{{site.data.keyword.runtime_liberty_full}}](/docs/java?topic=java-liberty)는 오픈 소스 Open Liberty 프로젝트를 기반으로 하는 빠르고 동적이며 사용하기 쉬운 Java 애플리케이션 서버입니다. Java EE 및 Spring 애플리케이션의 유연하고 적합한 엔터프라이즈급 런타임을 제공합니다.
