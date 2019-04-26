---

copyright:
  years: 2019
lastupdated: "2019-04-04"

keywords: java ee, jakarta ee, microprofile overview, cloud native java, cloud native microprofile

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

# 개요
{: #jee-overview}



## Java EE 및 Jakarta EE
{: #jakarta-ee}

Java EE7 및 EE8에 도입된 기술은 어노테이션, 경량 프로토콜 및 비동기 상호작용 패턴에 대한 지원을 통해 Java에서 마이크로서비스를 작성하는 데 적합합니다. Java EE7의 동시성 유틸리티는 RxJava와 같은 반응형 라이브러리를 컨테이너 친화적인 방식으로 사용하기 위한 간단한 메커니즘을 제공합니다.

2017년 9월에 IBM 및 Red Hat에서 지원하는 Oracle은 Java EE가 Jakarta EE로서 Eclipse Foundation으로 옮겨진다고 발표했습니다. Oracle은 계속 Java EE(Java EE 8 이하)와 연관된 모든 것을 소유하며, 이 엔터프라이즈 Java 플랫폼(Java EE 8 다음)의 모든 향후 개발은 Jakarta EE 작업 그룹의 안내에 따라 Eclipse Foundation에서 이루어집니다. 2018년 초에 Oracle은 EE4J 프로젝트에 API, RI, TCK 및 연관된 프로젝트 문서를 포함한 Java EE 기술을 제공하고 다시 라이센싱하는 데 큰 노력을 들이기 시작했습니다.

## MicroProfile
{: #microprofile}

Java EE는 마이크로서비스를 작성하기 위한 견고한 기반을 제공하지만, 마이크로서비스 애플리케이션에 적합한 기술과 프로그래밍 모델이 필요했습니다. IBM은 타사와 협력하여 개발자, 커뮤니티 및 벤더 간의 열린 협업인 MicroProfile을 출시하였습니다.

MicroProfile 커뮤니티는 마이크로서비스 및 엔터프라이즈 Java에 대한 신속한 혁신에 초점이 맞춰져 있습니다. 이 커뮤니티는 마이크로서비스 아키텍처 패턴을 따르는 클라우드 고유 애플리케이션에 가장 적합한 기술을 빌드하고 통합합니다. 각 MicroProfile 릴리스에는 일련의 기술이 포함되어 있습니다. 2016년에 출시된 MicroProfile 1.0은 경량 마이크로서비스를 위한 핵심 기능 세트인 CDI 1.2, JAX-RS 2.0 및 JSON-P 1.0을 제공하는 Java EE 7의 스펙 서브세트를 포함하고 있습니다. MicroProfile 플랫폼에는 구성 인젝션, 결함 허용, 상태 검사, 메트릭, 열린 추적과 같은 고유 클라우드 관련 문제를 처리하는 기술이 포함되어 있습니다. 또한 OpenAPI 정의 관련 작업을 수행하고 REST API를 보다 쉽게 이용할 수 있도록 하는 REST 클라이언트를 작성하며 JWT 전파를 사용하여 애플리케이션 보안 문제를 해결하는 데 필요한 벤더 애그노스틱 표준을 정의합니다.

오픈 소스 그룹에 참여하려면 [https://microprofile.io/](https://microprofile.io/){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘") 또는 [https://projects.eclipse.org/projects/technology.microprofile](https://projects.eclipse.org/projects/technology.microprofile){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")을 방문하십시오.
