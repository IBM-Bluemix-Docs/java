---

copyright:
  years: 2019
lastupdated: "2019-06-10"

keywords: open liberty java, websphere liberty java, jakarta, webpshere docker, liberty docker, liberty docker images, installutility, microprofile java, dual layer docker, develop microservices

subcollection: java

---

{:new_window: target="_blank"}
{:external: target="_blank" .external}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}

# Open Liberty 및 WebSphere Liberty
{: #liberty}

[Open Liberty](https://openliberty.io/){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")는 모듈식 *기능*을 사용하여 빌드된 경량 오픈 소스 Java&trade; 런타임입니다. [WebSphere Liberty](https://developer.ibm.com/wasdev/){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")는 Open Liberty의 상용 버전입니다. 

Liberty 기능은 다음 애플리케이션 프레임워크를 지원합니다.

* [MicroProfile](https://microprofile.io/){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘"), 즉 마이크로서비스 작성을 가속화하고 단순화하기 위해 새 표준과 API를 정의하는 오픈 소스 프로젝트.
* [Jakarta EE](https://jakarta.ee){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘") 및 [Java&trade; EE](https://www.oracle.com/technetwork/java/javaee/overview/index.html){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")(JNDI 또는 JAX-RS와 같은 개별 스펙의 기능 포함).
* [Spring Framework 및 Spring Boot](https://www.ibm.com/support/knowledgecenter/en/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/twlp_dep_springboot.html){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")(Spring Boot의 fat .jar에서 압축 컨테이너를 작성하는 메커니즘 포함).

[https://openliberty.io/guides/](https://openliberty.io/guides/){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")에 Liberty로 마이크로서비스와 클라우드 고유 앱을 작성하기 위한 광범위한 일련의 실용적 개발자 안내서가 있습니다.

## Docker를 위해 최적화됨
{: #liberty-optimized}

Kubernetes와 같은 자동화된 시스템이 컨테이너 이미지를 푸시하는 경우 이미지 크기가 중요합니다. 이 문제점에 도움이 되도록 Docker 이미지의 계층이 캐시됩니다. 모듈식 아키텍처를 고려할 때, Liberty는 Docker 컨테이너에 효율적 패키징 파이프라인을 제공하므로 클라우드 환경에 적합하게 작동할 수 있습니다. 하나의 기본 이미지를 사용하여 실행 중인 컨테이너의 리소스 풋프린트가 서비스의 요구사항에 따라 달라지도록 하는 많은 워크로드를 지원할 수 있습니다.

Liberty는 Spring Boot fat .jar를 압축으로 변환하기 위한 최적화된 지원 및 도구, 캐시된 Docker 이미지 계층을 이용하여 주기 및 공개 시간을 향상시키는 최적화된 Docker 컨테이너를 제공합니다. 드물게 변경되는 라이브러리 종속성을 개별 계층으로 푸시하고 앱 클래스만 상위 계층에 보관하면 반복적 재빌드와 재배치가 훨씬 더 빨라집니다. [Creating Dual Layer Docker images for Spring Boot apps](https://openliberty.io/blog/2018/07/02/creating-dual-layer-docker-images-for-spring-boot-apps.html){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")에서 자세히 알아보십시오.

## Liberty Docker 이미지
{: #liberty-images}

Open Liberty와 WebSphere Liberty는 둘 다 Java 기반 마이크로서비스의 시작점으로 사용할 수 있는 다양한 기본 이미지를 제공합니다. 여러 기능이 사전 선택된 소형 풋프린트 또는 OpenJ9 VM을 사용하는 이미지가 있습니다. 예를 들어, 다음과 같습니다.

1. `open-liberty:microProfile2` 또는 `websphere-liberty:microProfile2`
2. `open-liberty:javaee8` 또는 `websphere-liberty:javaee8`
3. `open-liberty:springBoot2` 또는 `websphere-liberty:springBoot2`

기본 이미지의 최신 목록은 Docker 허브의 [`websphere-liberty`](https://hub.docker.com/_/websphere-liberty/){: external} 또는 [`open-liberty`](https://hub.docker.com/_/open-liberty/){: external}를 참조하십시오.

### Open Liberty 및 Docker
{: #openliberty-docker}

[Docker 허브의 이미지 목록](https://hub.docker.com/_/open-liberty/){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")(예: `open-liberty:microProfile`)에서 앱에 가장 적합한 이미지를 선택하고 이 이미지에서(`FROM`) 확장되는 서비스의 Docker 파일을 작성하십시오. 명령을 추가하여 새 이미지에 앱 및 서버 구성을 추가하십시오. 예를 들어, 다음과 같습니다.

```docker
FROM open-liberty:microProfile\
COPY server.xml /config/\
ADD Sample1.war /config/dropins/
```
{: codeblock}

자세한 예와 작업 코드는 다음 Open Liberty 안내서를 참조하십시오.

* [Using Docker containers to develop microservices](https://openliberty.io/guides/docker.html){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")
* [Running an application in a Docker container](https://openliberty.io/guides/getting-started.html#running-the-application-in-a-docker-container){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")

### WebSphere Liberty 및 Docker
{: #wlp-docker}

Open Liberty와 상용 버전인 WebSphere Liberty 사이에는 몇 가지 차이점이 있습니다. Docker 이미지 작성에 있어 가장 중요한 차이점 중 하나는 Open Liberty에서 `installUtility` 명령을 사용할 수 없다는 점입니다.

WebSphere Liberty는 Open Liberty가 Docker 이미지에 사용하는 것과 동일한 기본 사용자 정의 패턴을 지원합니다. Liberty의 본질적인 모듈식 디자인을 통해 앱과 이 애플리케이션에 필요한 특정 기능 세트를 포함하는 사용자 정의 이미지를 쉽게 작성할 수 있습니다. WebSphere Liberty에는 특징 없는 커널([`websphere-liberty:kernel`](https://github.com/WASdev/ci.docker/blob/9d28dfba4d20596f89b393bc9e3ae8295feec469/ga/developer/kernel/Dockerfile){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘"))의 이미지가 있으며 이 이미지는 실제 사용자 정의되는 이미지의 기초가 됩니다.

[`websphere-liberty` 이미지 문서](https://hub.docker.com/_/websphere-liberty/){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")에서는 사용자 정의 이미지 작성에 필요한 다음과 같이 3행으로 된 간단한 Docker 파일에 대해 설명합니다.

```docker
FROM websphere-liberty:kernel
COPY server.xml /config/
RUN installUtility install --acceptLicense defaultServer
```
{: codeblock}

`installUtility` 도구는 Liberty 이미지에서 아직 사용할 수 없는 `server.xml` 파일에 필요한 기능을 찾습니다. 그런 다음 해당 기능을 다운로드하여 Docker 이미지에 설치합니다. 이 접근 방식은 최소 이미지를 작성하지만 `server.xml`의 내재적 기능 목록은 캐시된 Docker 계층에서 제대로 작동하지 않습니다. `server.xml`의 모든 변경사항은 설치된 기능이 있는 계층을 무효화하며, 다음 번에 이미지를 빌드할 때 누락된 기능을 다시 설치해야 합니다.

사용자 정의 이미지를 작성하는 더 좋은 방법은 명시적 기능 목록을 사용하여 `installUtility`를 호출하는 것입니다. 이 방법을 사용하면 여전히 최소 이미지가 작성되지만, 서버 구성 변경에 관계없이 이미지의 계층을 캐시하여 재사용할 수 있으므로 빌드 시 훨씬 더 효율적으로 작동합니다. 예를 들어, 다음 예에서는 기능의 서브세트로 사용자 정의 이미지를 작성하여 JAX-RS, JNDI 및 WebSocket을 사용하는 앱을 지원합니다.

```docker
FROM websphere-liberty:kernel
RUN /opt/ibm/wlp/bin/installUtility install --acceptLicense \
  cdi-1.2 \
  concurrent-1.0 \
  jaxrs-2.0 \
  jndi-1.0 \
  ssl-1.0 \
  websocket-1.1
COPY server.xml /config/
```
{: codeblock}

Docker 이미지를 구성하는 방법은 몇 가지 요인에 따라 다릅니다.

1. 기본 이미지를 재사용하거나 사용자 정의하는 방법은 무엇입니까?
    각 앱 기능이 서로 다른 경우 이러한 단계에서는 잠재적으로 재사용을 통해 가장 작은 가능한 이미지를 생성합니다. 하지만 작은 사용자 정의 이미지 사용은 Docker 호스트에 앱을 빠르게 배치할 수 있음을 의미합니다. 확실하지 않은 경우 재사용 가능성을 최대화하기 위해 Docker 허브의 미리 생성된 이미지 중 하나로 시작하십시오.
2. 이 이미지를 얼마나 자주 업데이트합니까?
    예를 들어, CI/CD 파이프라인에서 앱이 자주 업데이트되는 경우, 가장 빈번하게 변경된 계층(종종 앱 클래스 또는 앱 바이너리)이 맨 위 계층에 오도록 이미지를 구조화하는 것이 좋습니다. 이 구조를 사용하면 빌드 및 배치 시간이 가속화되어 앱이 변경되고 이미지가 다시 빌드될 때 다시 빌드해야 하는 계층 수가 줄어듭니다.

잘 모를 경우 [Open Liberty Docker 안내서](https://openliberty.io/guides/docker.html){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")에 따라 시작하십시오.
