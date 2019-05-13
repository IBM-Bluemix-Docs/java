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

O Spring Platform é um ecossistema de projetos que visam tornar mais simples a criação de aplicativos Java&trade. Geralmente, o termo "Spring" refere-se ao Spring Framework, mas também é usado genericamente para se referir a qualquer projeto que faça parte ou que use tecnologias baseadas em Spring, incluindo Spring Boot.

## Spring Framework
{: #spring-framework}

O Spring Framework foi introduzido em 2002 e tem liberado uma versão principal aproximadamente a cada 3 anos. A estrutura inclui um conjunto grande de componentes (referidos como Módulos Spring) que cobrem tudo de terminais REST a abstrações de banco de dados. Com a sua liberação mais recente em 2017, o Spring Framework 5 introduziu o suporte ao JDK atualizado e o WebFlux. O WebFlux é um novo módulo para programação reativa com base no [Reator de projeto](https://projectreactor.io/){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo").

O Spring Framework 4.3 é a última ramificação de recurso para o Spring Framework 4.x, com suporte até 2020. Os aplicativos que usam versões mais antigas do Spring devem ser migrados para o Spring Framework 5.

A documentação abrangente para o Spring Framework está aqui:

* [Guia de Referência do Spring Framework para 5.1.x](https://docs.spring.io/spring/docs/5.1.x/spring-framework-reference/){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo")
* [Guia de Referência do Spring Framework para 4.3.x](https://docs.spring.io/spring/docs/4.3.x/spring-framework-reference/){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo")
* [Fazendo upgrade para o Spring Framework 5.x](https://github.com/spring-projects/spring-framework/wiki/Upgrading-to-Spring-Framework-5.x){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo")

## Spring Boot
{: #spring-boot}

O Spring Boot foi apresentado em 2014, para "melhorar as arquiteturas de aplicativos da web sem contêiner". Seu objetivo era mover a configuração de XML para o código. O Spring Boot fornece mecanismos para criar aplicativos com base em uma visualização opinada sobre quais tecnologias devem ser usadas. Os aplicativos Spring Boot são aplicativos Spring e usam um modelo de programação exclusivo para esse ecossistema.

O Spring Boot enfatiza a convenção sobre a configuração e usa uma combinação de anotações e descoberta de caminho de classe para ativar funções adicionais. Por padrão, os aplicativos Spring Boot são construídos em um arquivo JAR executável autocontido que inclui todas as dependências necessárias (incluindo um servidor Tomcat integrado). Os aplicativos podem, de forma alternativa, serem empacotados como arquivos WAR implementados em um servidor de aplicativos.

O Spring Boot está agora na versão 2.0, que foi liberada em 2018. A manutenção da ramificação 1.5.x termina em agosto de 2019.

A documentação abrangente do Spring Boot está aqui:

* [Guia de Referência do Spring Boot para 2.1.x](https://docs.spring.io/spring-boot/docs/2.1.x/reference/){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo")
* [Guia de Referência do Spring Boot para 1.5.x](https://docs.spring.io/spring-boot/docs/1.5.x/reference/){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo")

## Selecionando o Spring Framework ou o Spring Boot
{: #spring-framework-or-boot}

Para novos aplicativos nativos de nuvem, se você optar por usar o Spring Framework, também deverá usar o Spring Boot. O Spring Boot simplifica a configuração e o conjunto de aplicativos baseados em Spring e fornece recursos, como o Spring Actuator, que simplificam a criação de [aplicativos nativos de nuvem](/docs/java?topic=cloud-native-overview#overview).

### Spring Boot Actuator
{: #spring-boot-actuator}

O Spring Boot Actuator define uma coleção de terminais que são úteis para inspecionar um app Spring em execução. A ativação deles é tão simples quanto a inclusão de uma única dependência no aplicativo.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```
{: codeblock}

A estrutura e o comportamento do Spring Boot Actuator mudaram significativamente entre o Spring Boot 1 e o Spring Boot 2. O Boot 1 Actuator juntou-se ao aplicativo, com mecanismos de configuração e segurança separados. A versão Boot 2 é mais capaz e usa um modelo de segurança que se integra ao restante do aplicativo Spring.

As mudanças de comportamento são particularmente relevantes se você estiver migrando um aplicativo Boot 1 para o Boot 2. Leia a seção Actuator do [Guia de migração do Boot 1 para o Boot 2](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.0-Migration-Guide#spring-boot-actuator){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") para obter mais detalhes.
{: tip}

No Boot 2, o Actuator age como uma estrutura mini-REST. Todos os terminais estão sob um contexto `/actuator`. Terminais para executar verificações de funcionamento ou reunir métricas, aplicativo ou outras informações de ambiente são fornecidos. Para obter uma lista completa, verifique a [Documentação do Actuator](https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/html/production-ready-features.html#production-ready){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo").

É fácil criar seu próprio terminal do Actuator usando um `@Component` Spring:

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

O Spring Boot tem dois controles para configurar quais Actuators estão ativos no tempo de execução; os Actuators podem ser "ativados" e "expostos". Para poder ser acessível, um Actuator deve ser ambos. Por padrão, muitos terminais são ativados, mas apenas o funcionamento e as informações são expostos. Além disso, se o Spring Security estiver ativado para o aplicativo, o acesso deverá ser explicitamente permitido para os terminais do Actuator.

O Actuator no Spring Boot 1 tem sua própria configuração de segurança, geralmente configurada em application.properties. Em vez de "ativados" e "expostos", há "ativados" e "sensíveis". Terminais sensíveis requerem autenticação se entregues por HTTP. Por padrão, o terminal /metrics do Spring Boot Actuator é ativado no Spring Boot 1, mas ele é considerado sensível e, portanto, requer autorização ou ser configurado como não sensível. Para alguns ambientes, a configuração do Spring Security pode ser a resposta certa, mas no Kubernetes, os terminais de métricas permanecem internos ao cluster. Para obter mais informações, verifique os [docs do Spring Boot 1 Actuator](https://docs.spring.io/spring-boot/docs/1.5.2.RELEASE/reference/htmlsingle/#production-ready){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo").
{: note}

## Spring Cloud
{: #spring-cloud}

O Spring Cloud é uma coleção de integrações entre as tecnologias de nuvem de terceiros e o modelo de programação Spring. O objetivo é ajudar os desenvolvedores que constroem aplicativos Spring para implementação na nuvem. A amplitude de cobertura que é obtida pelo Spring Cloud torna-se possível por sua natureza modular, pois cada projeto no Spring Cloud é focado em uma tecnologia específica ou um conjunto de tecnologias.

Os projetos Spring Cloud seguem a abordagem geral do Spring de favorecer a convenção sobre a configuração. A maioria dos recursos é ativada ao incluir a dependência correta no momento da criação.

Consulte o [Site do projeto do Spring Cloud](https://spring.io/projects/spring-cloud){: new_window} oficial![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") para obter mais informações sobre os projetos Spring Cloud e tecnologias suportadas.

### Spring Cloud com Kubernetes
{: #spring-cloud-kubernetes}

O Spring Cloud Kubernetes é um projeto Spring Cloud que é destinado a tornar várias tarefas mais fáceis para os aplicativos Spring que são executados no Kubernetes. O Spring Cloud Kubernetes oferece implementações de interfaces Spring comuns que são mapeadas para os conceitos e serviços do Kubernetes.

Historicamente, o Spring usa as bibliotecas da Netflix como Eureka como um Registro de Serviço e Ribbon como um Load Balancer do lado do cliente. Em ambientes do Kubernetes, ambas as funções são atendidas pelo próprio Kubernetes, fazendo com que os recursos no nível do aplicativo sejam redundantes. O Spring Cloud Kubernetes adapta abstrações do Spring, como `DiscoveryClient`, para os mecanismos do Kubernetes subjacentes.

Tenha cuidado se você estiver usando a descoberta Ribbon por meio do Spring Cloud Kubernetes em um cluster com o Istio instalado, em que Istio está sendo usado para afetar o roteamento de serviço. O balanceamento de carga do lado do cliente implica que o cliente deseja selecionar o serviço de destino em si, enquanto o Istio pode estar tentando rotear a chamada em outro lugar. Somente um deles pode vencer, levando a discordância sobre como uma chamada pode ser roteada. Essa combinação deve ser evitada se for possível.
{: note}

Além disso, o Spring Cloud Kubernetes oferece integração entre os ConfigMaps do Kubernetes e os Secrets para os beans de configuração `@Autowired` do Spring. Isso inclui estratégias para manipular mudanças dinâmicas em `ConfigMaps` e `Secrets` feitas enquanto o aplicativo está em execução.

Finalmente, o Spring Cloud Kubernetes aumenta o terminal de funcionamento padrão do Spring Boot Actuator para incluir informações adicionais que estão relacionadas à implementação.

Para obter mais informações, verifique a [documentação do Spring Cloud Kubernetes](https://cloud.spring.io/spring-cloud-static/spring-cloud-kubernetes/2.1.0.RC1/single/spring-cloud-kubernetes.html){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo")


<!--
### Spring Cloud Streams
{: #spring-cloud-streams}


:FIXME:
-->
