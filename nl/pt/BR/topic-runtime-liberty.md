---

copyright:
  years: 2019
lastupdated: "2019-03-15"

keywords: open liberty java, websphere liberty java, jakarta, webpshere docker, liberty docker, liberty docker images, installutility, microprofile java, dual layer docker, develop microservices

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

# Open Liberty e WebSphere Liberty
{: #liberty}

O [Open Liberty](https://openliberty.io/){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") é um tempo de execução Java de software livre leve construído usando *recursos* modulares. O [WebSphere Liberty](https://developer.ibm.com/wasdev/){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") é uma versão comercial do Open Liberty. Os recursos do Liberty suportam as estruturas de aplicativos a seguir:

* [MicroProfile](https://microprofile.io/){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo"), um projeto de software livre que define novos padrões e APIs para acelerar e simplificar a criação de microsserviços.
* [Jakarta EE](https://jakarta.ee){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") e [Java EE](https://www.oracle.com/technetwork/java/javaee/overview/index.html){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo"), incluindo recursos para especificações individuais, como JNDI ou JAX-RS.
* [Spring Framework e Spring Boot](https://www.ibm.com/support/knowledgecenter/en/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/twlp_dep_springboot.html){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo"), incluindo mecanismos para fazer contêineres compactos dos fat jars do Spring Boot.

Há um amplo conjunto de guias de desenvolvedor práticos para criar microsserviços e aplicativos nativos de nuvem com Liberty em [https://openliberty.io/guides/](https://openliberty.io/guides/){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo").

## Otimizado para o Docker
{: #liberty-optimized}

Quando sistemas automatizados como o Kubernetes ficam impondo imagens de contêiner, o tamanho da imagem começa a importar. As camadas em imagens do Docker são armazenadas em cache para ajudar com isso. Dada a sua arquitetura modular, o Liberty fornece um pipeline de pacote eficiente para contêineres do Docker, tornando-o um ótimo ajuste operacional para ambientes de nuvem. Uma imagem base pode ser usada para suportar muitas cargas de trabalho, enquanto permite a área de cobertura de recurso de contêineres em execução varie com base nos requisitos do serviço.

O Liberty fornece ferramentas e suporte otimizado para converter fat jars do Spring Boot em contêineres de Docker compactos e otimizados que aproveitam as camadas de imagem do Docker em cache para melhorar o ciclo e a publicação de horários. Ao enviar por push as dependências de biblioteca que não mudam com frequência para uma camada separada e manter apenas as classes de aplicativos na camada superior, reconstruções iterativas e reimplementações serão muito mais rápidas. Consulte mais em [Criando imagens do Docker de camada dupla para os apps Spring Boot](https://openliberty.io/blog/2018/07/02/creating-dual-layer-docker-images-for-spring-boot-apps.html){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo").

## Imagens do Docker Liberty
{: #liberty-images}

O Open Liberty e o WebSphere Liberty fornecem uma variedade de imagens base que podem ser usadas como pontos de início para os microsserviços baseados em Java. Há imagens que usam pequenas áreas de cobertura ou VMs OpenJ9 com recursos diferentes pré-selecionados. Por exemplo:

1. open-liberty:microProfile2 ou websphere-liberty:microProfile2
2. open-liberty:javaee8 ou websphere-liberty:javaee8
3. open-liberty:springBoot2 ou websphere-liberty:springBoot2

Verifique [websphere-liberty](https://hub.docker.com/_/websphere-liberty/){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") ou [open-liberty](https://hub.docker.com/_/open-liberty/){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") no Docker Hub para o mais atualizado da lista de imagens base.

### Open Liberty e Docker
{: #openliberty-docker}

Escolha a imagem mais apropriada para seu aplicativo na [lista de imagens no Docker Hub](https://hub.docker.com/_/open-liberty/){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") (por exemplo, `open-liberty:microProfile`) e crie um Dockerfile para seu serviço que estenda `FROM`. Inclua os comandos para copiar sua configuração de app e servidor para a nova imagem. Como um exemplo:

```docker
FROM open-liberty:microProfile\
COPY server.xml /config/\
ADD Sample1.war /config/dropins/
```
{: codeblock}

Para obter mais exemplos e código de trabalho, consulte os guias Open Liberty a seguir:

* [Usando contêineres do Docker para desenvolver microsserviços](https://openliberty.io/guides/docker.html){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo")
* [Executando um aplicativo em um contêiner do Docker](https://openliberty.io/guides/getting-started.html#running-the-application-in-a-docker-container){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo")

### WebSphere Liberty e Docker
{: #wlp-docker}

Há algumas diferenças entre o Open Liberty e a versão comercial, o WebSphere Liberty. Uma das mais significativas para criar imagens do Docker é que o comando `installUtility` não está disponível no Open Liberty.

O WebSphere Liberty suporta os mesmos padrões de customização básicos que o OpenLiberty para as imagens do Docker, mas o design inerentemente modular do Liberty torna simples (e típico) a criação de uma imagem customizada que contém um aplicativo e o conjunto específico de recursos que ele requer. O WebSphere Liberty possui uma imagem para um kernel sem recurso, [websphere-liberty:kernel](https://github.com/WASdev/ci.docker/blob/9d28dfba4d20596f89b393bc9e3ae8295feec469/ga/developer/kernel/Dockerfile){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo"), que faz uma base sólida para uma imagem realmente customizada.

A [documentação da imagem websphere-liberty](https://hub.docker.com/_/websphere-liberty/){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") descreve o Dockerfile de 3 linhas simples a seguir necessário para criar uma imagem customizada:

```docker
FROM websphere-liberty:kernel
COPY server.xml /config/
RUN installUtility install --acceptLicense defaultServer
```
{: codeblock}

A ferramenta `installUtility` procura quaisquer recursos que você precisa em seu arquivo `server.xml` que ainda não estão disponíveis em sua imagem do Liberty. Em seguida, ela fará download desses recursos e os instalará em sua imagem do Docker. Essa abordagem criará uma imagem mínima, mas a lista de recursos implícitos em `server.xml` não funciona bem com as camadas do Docker em cache. Qualquer mudança para `server.xml` invalidará a camada com os recursos instalados, requerendo que os recursos ausentes sejam instalados novamente na próxima vez em que a imagem for construída.

Uma maneira melhor de criar imagens customizadas é chamar `installUtility` com uma lista explícita de recursos. Isso ainda criará uma imagem mínima, mas se comportará muito melhor no momento da criação, pois a camada da imagem pode ser armazenada em cache e reutilizada, independentemente das mudanças na configuração do servidor. Por exemplo, o seguinte criará uma imagem customizada usando o subconjunto de recursos para suportar para um aplicativo que usa JAX-RS, JNDI e WebSockets:

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

Como você estrutura sua imagem do Docker depende de alguns fatores:

1. quão reutilizável ou customizada você deseja que sua imagem base seja?
    Essas etapas produzem a menor imagem possível, potencialmente em detrimento da reutilização, se os recursos de cada aplicativo forem diferentes. No entanto, ter imagens customizadas muito pequenas significa que o aplicativo pode ser implementado rapidamente em hosts do Docker. Se você não tiver certeza, inicie com uma das imagens pré-geradas do Docker Hub para maior reutilização.
2. Com que frequência você atualiza essa imagem?
    Se o aplicativo for atualizado com muita frequência, por exemplo, em um pipeline CI/CD, será benéfico estruturar a imagem para que a camada mais frequentemente mudada (geralmente, suas classes de aplicativo ou binário do app) esteja no topo. Isso reduz o número de camadas que precisam ser reconstruídas quando o aplicativo muda e a imagem é reconstruída, acelerando os tempos de construção e implementação.

Quando em dúvida, siga o [Guia do Open Liberty Docker](https://openliberty.io/guides/docker.html){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") para iniciar.
