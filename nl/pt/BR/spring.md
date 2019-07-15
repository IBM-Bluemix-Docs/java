---

copyright:
  years: 2019
lastupdated: "2019-06-10"

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

O Spring Platform é um ecossistema de projetos que visam tornar mais simples a criação de aplicativos do Java&trade;. Geralmente, o termo "Spring" refere-se ao Spring Framework, mas também é usado genericamente para se referir a qualquer projeto que faça parte ou que use tecnologias baseadas em Spring, incluindo Spring Boot.

## Spring Framework
{: #spring-framework}

O Spring Framework foi introduzido em 2002 e libera uma versão principal aproximadamente a cada 3 anos. A estrutura inclui um conjunto grande de componentes (referidos como Módulos Spring) que cobrem tudo de terminais REST a abstrações de banco de dados. Com a sua liberação mais recente em 2017, o Spring Framework 5 introduziu o suporte ao JDK atualizado e o WebFlux. O WebFlux é um novo módulo para programação reativa com base no [Reator de projeto](https://projectreactor.io/){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo").

O Spring Framework 4.3 é a última ramificação de recurso para o Spring Framework 4.x, com suporte até 2020. Os aplicativos que usam versões mais antigas do Spring devem ser migrados para o Spring Framework 5.

A documentação abrangente para o Spring Framework está aqui:

* [Guia de Referência do Spring Framework para 5.1.x](https://docs.spring.io/spring/docs/5.1.x/spring-framework-reference/){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo")
* [Guia de Referência do Spring Framework para 4.3.x](https://docs.spring.io/spring/docs/4.3.x/spring-framework-reference/){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo")
* [Fazendo upgrade para o Spring Framework 5.x](https://github.com/spring-projects/spring-framework/wiki/Upgrading-to-Spring-Framework-5.x){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo")

## Spring Boot
{: #spring-boot}

O Spring Boot foi apresentado em 2014, para "melhorar as arquiteturas de aplicativos da web sem contêiner". Seu objetivo era mover a configuração de XML para o código. O Spring Boot fornece mecanismos para criar apps com base em uma visão opinativa sobre quais tecnologias devem ser usadas. Os apps do Spring Boot são apps do Spring e usam um modelo de programação exclusivo para esse ecossistema.

O Spring Boot enfatiza a convenção sobre a configuração e usa uma combinação de anotações e descoberta de caminho de classe para ativar funções. Por padrão, os aplicativos Spring Boot são construídos em um arquivo JAR executável autocontido que inclui todas as dependências necessárias (incluindo um servidor Tomcat integrado). Os apps podem, alternativamente, ser empacotados como arquivos war implementados em um servidor de aplicativos.

O Spring Boot está agora na versão 2.0, que foi liberada em 2018. A manutenção da ramificação 1.5.x termina em agosto de 2019.

A documentação abrangente do Spring Boot está aqui:

* [Guia de Referência do Spring Boot para 2.1.x](https://docs.spring.io/spring-boot/docs/2.1.x/reference/){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo")
* [Guia de Referência do Spring Boot para 1.5.x](https://docs.spring.io/spring-boot/docs/1.5.x/reference/){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo")

## Selecionando o Spring Framework ou o Spring Boot
{: #spring-framework-or-boot}

Para novos apps nativos de nuvem, se você optar por usar o Spring Framework, também será possível usar o Spring Boot. O Spring Boot simplifica a configuração e a montagem de apps baseados em Spring e fornece recursos, como o Spring Actuator, que simplificam a criação de [apps nativos de nuvem](/docs/java?topic=cloud-native-overview#overview).

### Spring Boot Actuator
{: #spring-boot-actuator}

O Spring Boot Actuator define uma coleção de terminais que são úteis para inspecionar um app Spring em execução. A ativação deles é tão simples quanto incluir uma única dependência no app.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```
{: codeblock}

A estrutura e o comportamento do Spring Boot Actuator mudaram significativamente entre o Spring Boot 1 e o Spring Boot 2. O Boot 1 Actuator ficou ao lado do app, com mecanismos de configuração e segurança separados. A versão do Boot 2 é mais capaz e usa um modelo de segurança que se integra com o restante do app do Spring.

As mudanças de comportamento serão relevantes se você estiver migrando um app do Spring Boot 1.x para o Spring Boot 2.0. Leia a seção do Atuador do [Guia de migração do Boot 1 para o Boot 2](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.0-Migration-Guide#spring-boot-actuator){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") para obter mais detalhes.
{: tip}

No Boot 2, o Actuator age como uma estrutura mini-REST. Todos os terminais estão sob um contexto `/actuator`. Terminais para executar verificações de funcionamento ou reunir métricas, informações de app ou outras informações de ambiente são fornecidos. Para obter uma lista completa, verifique a [Documentação do Actuator](https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/html/production-ready-features.html#production-ready){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo").

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

O Spring Boot tem dois controles para configurar quais Atuadores estão ativos no tempo de execução. Os atuadores podem ser "ativados" e "expostos". Para ser acessível, um Atuador deve ser ambos. Por padrão, muitos terminais são ativados, mas apenas o funcionamento e as informações são expostos. Além disso, se o Spring Security estiver ativado para o app, o acesso deverá ser explicitamente permitido para os terminais do Atuador.

O Actuator no Spring Boot 1 tem sua própria configuração de segurança, geralmente configurada em application.properties. Em vez de "ativados" e "expostos", há "ativados" e "sensíveis". Terminais sensíveis requerem autenticação se entregues por HTTP. Por padrão, o terminal /metrics do Spring Boot Actuator é ativado no Spring Boot 1, mas ele é considerado sensível e, portanto, requer autorização ou ser configurado como não sensível. Para alguns ambientes, a configuração do Spring Security pode ser a resposta certa, mas no Kubernetes, os terminais de métricas permanecem internos ao cluster. Para obter mais informações, verifique os [docs do Spring Boot 1 Actuator](https://docs.spring.io/spring-boot/docs/1.5.2.RELEASE/reference/htmlsingle/#production-ready){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo").
{: note}

## Spring Cloud
{: #spring-cloud}

O Spring Cloud é uma coleção de integrações entre as tecnologias de nuvem de terceiros e o modelo de programação Spring. Ele visa ajudar os desenvolvedores que constróem apps do Spring para implementação na nuvem. A cobertura que é alcançada pelo Spring Cloud é possibilitada por sua natureza modular, pois cada projeto dentro do Spring Cloud é focado em uma tecnologia específica ou em um conjunto de tecnologias.

Os projetos de nuvem do Spring seguem a abordagem geral do Spring de favorecer a convenção sobre a configuração. A maioria dos recursos é ativada ao incluir a dependência correta no momento da criação.

Para obter mais informações sobre projetos do Spring Cloud, consulte o [Site do projeto do Spring Cloud oficial](https://spring.io/projects/spring-cloud){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo").

### Spring Cloud com Kubernetes
{: #spring-cloud-kubernetes}

O Spring Cloud Kubernetes é um projeto do Spring Cloud que é destinado a tornar várias tarefas mais fáceis para os apps do Spring que são executados no Kubernetes. O Spring Cloud Kubernetes oferece implementações de interfaces Spring comuns que são mapeadas para os conceitos e serviços do Kubernetes.

Historicamente, o Spring usa as bibliotecas da Netflix como Eureka como um Registro de Serviço e Ribbon como um Load Balancer do lado do cliente. Em ambientes do Kubernetes, ambas as funções são atendidas pelo próprio Kubernetes, fazendo com que os recursos no nível do aplicativo sejam redundantes. O Spring Cloud Kubernetes adapta abstrações do Spring, como `DiscoveryClient`, para os mecanismos do Kubernetes subjacentes.

Tenha cuidado se você estiver usando a descoberta Ribbon por meio do Spring Cloud Kubernetes em um cluster com o Istio instalado, em que Istio está sendo usado para afetar o roteamento de serviço. O balanceamento de carga do lado do cliente implica que o cliente deseja selecionar o serviço de destino em si, enquanto o Istio pode estar tentando rotear a chamada em outro lugar. Somente um método pode vencer, levando à discordância sobre como uma chamada pode ser roteada. Essa combinação deve ser evitada se for possível.
{: note}

Além disso, o Spring Cloud Kubernetes oferece a integração entre os `ConfigMaps` e os `Secrets` do Kubernetes com beans de configuração `@Autowired` do Spring. As estratégias serão incluídas para manipular mudanças dinâmicas para `ConfigMaps` e `Secrets` enquanto o app estiver em execução.

Finalmente, o Spring Cloud Kubernetes aumenta o terminal de funcionamento padrão do Spring Boot Actuator para incluir informações adicionais que estão relacionadas à implementação.

Para obter mais informações, verifique a [documentação do Spring Cloud Kubernetes](https://cloud.spring.io/spring-cloud-static/spring-cloud-kubernetes/2.1.0.RC1/single/spring-cloud-kubernetes.html){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo")


<!--
### Spring Cloud Streams
{: #spring-cloud-streams}


:FIXME:
-->
