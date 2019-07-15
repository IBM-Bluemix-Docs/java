---

copyright:
  years: 2019
lastupdated: "2019-06-10"

keywords: spring metrics, configure metrics spring, micrometer spring, micrometer, spring boot 2, spring actuator, prometheus spring

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

# Métricas com Spring
{: #spring-metrics}

A partir do Spring Framework 5, as métricas no Spring são agora manipuladas pelo Micrometer. O Micrometer é uma estrutura que descreve a si mesmo como "SLF4J for Metrics". Exatamente como o SLF4J age como uma API neutra do fornecedor para criação de log que se conecta a vários back-ends de criação de log, o Micrômetro fornece uma API neutra do fornecedor que pode instrumentar e medir o seu código. Em seguida, ele pode fornecer essas métricas em diante para os agregadores de métricas, como Prometheus, DataDog, InfluxDB e Telegraf. 

A estrutura do Micrometer permite que o Spring se integre em uma ampla variedade de arquiteturas nativas de nuvem. Incluir suporte para Prometheus ou Statsd é uma questão simples de modificar dependências e se o coletor de métricas for baseado em push, fornecendo informações de destino em `application.properties`. O Spring e o Micrometer determinam o que fazer no tempo de execução com base em quais dependências eles localizam no caminho de classe do aplicativo.

## Ativando métricas
{: #spring-metrics-enabling}

As métricas básicas são fornecidas pelo Spring Boot Actuator, que requer a dependência `spring-boot-iniciter-actuator` em seu arquivo `pom.xml`:  

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-actuator</artifactId>
</dependency>
```
{: codeblock}

Se você ainda estiver no Spring Boot 1.5.x, as métricas (e atuadores) funcionarão um pouco diferente. O Micrômetro tornou-se o padrão para as métricas do Spring no Spring Boot 2.0, mas as backports foram disponibilizadas para a Boot 1.5.x que permitem práticas consistentes para criar e reunir métricas. Mais importante, o Micrometer suporta vários sistemas de monitoramento, incluindo o Prometheus, que o torna uma opção melhor do que o sistema de métricas Boot 1.5.x para implementações em nuvem.
{: note}

## Métricas do Micrometer
{: #spring-metrics-micrometer}

O Micrometer abstrai o conceito de um destino para métricas por meio de sua interface `MeterRegistry`. O `MeterRegistry` fornece a maneira para o app obter contadores customizados, medidores e cronômetros. Se múltiplos destinos forem necessários, o Micrômetro fornecerá uma implementação `CompositeMeterRegistry`, permitindo que o app seja isolado dos destinos configurados reais para métricas.

Em qualquer versão do Spring Boot, quando um terminal de métricas baseado no Micrômetro estiver ativado, o Spring Boot criará automaticamente um `CompositeMeterRegistry` e o disponibilizará para o app como um `Bean`. O registro é preenchido automaticamente com as implementações suportadas de `MeterRegistry` que estão localizadas no caminho da classe. Suportar vários sistemas de monitoramento ou alternar entre eles é, então, uma questão simples de alterar dependências no pom.xml. 

O Spring permite a customização do bean `MeterRegistry` por meio de um `MeterRegistryCustomizer`, que é em si, um bean. Quando o Spring identifica a presença desse bean, ele pode configurar o `MeterRegistry` global antes de quaisquer métricas serem relatadas. Essa customização é frequentemente usada para incluir tags comuns no `MeterRegistry`, DataCenter ou EnvironmentType.

Quando você nomeia métricas, o Spring e o Micrômetro recomendam seguir uma convenção de nomenclatura que divide as palavras com um `.` em vez de usar o Camel Case ou `_` ou outras abordagens. Nomear é importante porque o Micrômetro está retransmitindo essas métricas para um ou mais destinos configurados e executa quaisquer conversões de nomes, conforme necessário, para atender aos requisitos dos destinos.

## Configurando métricas com o Spring Boot
{: #spring-metrics-configuration}

As seções a seguir descrevem como ativar as métricas do Spring Boot Actuator com o Micrômetro. Aprenda a coletar métricas e forneça um terminal para Prometheus (para cada liberação) começando com o Spring Boot 2 como o mais atual.

### Configurando métricas no Spring Boot 2
{: #spring-metrics-boot2}

O terminal de métricas do Spring Boot 2 Actuator deve ser ativado e exposto para o acesso à web. Por padrão, o terminal está ativado, mas não exposto (verifique a [Documentação do Terminal Spring](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-endpoints.html){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") para obter mais informações).  

Inclua o seguinte em `application.properties` para expor o terminal de métricas:

```properties
management.endpoints.web.exposure.include=health,metrics
```
{: codeblock}

Verifique se o terminal de métricas está ativado verificando o terminal `/actuator`. A saída é semelhante à seguinte quando você executa o app localmente e usa `jq` para ter uma impressão elegante da resposta do JSON:

```
$ curl -s localhost:8080/actuator | jq .
{
  "_links": {
    "self": {
      "href": "http://localhost:8080/actuator",
      "templated": false
    },
    ...
    "metrics": {
      "href": "http://localhost:8080/actuator/metrics",
      "templated": false
    },
    "metrics-requiredMetricName": {
      "href": "http://localhost:8080/actuator/metrics/{requiredMetricName}",
      "templated": true
    },
    ...
  }
}
```
{: screen}

Acesse o terminal `/actuator/metrics` para exibir uma lista formatada de JSON de métricas disponíveis, semelhante à saída a seguir:
```
$ curl -s localhost:8080/actuator/metrics | jq .
{
  "names": [
    "jvm.memory.max",
    "jvm.threads.states",
    "process.files.max",
    "jvm.gc.memory.promoted",
    ...
  ]
}
```
{: screen}

Uma outra solicitação é necessária para ver o valor real de uma métrica individual (como pode ser inferido na lista de URLs atuadoras). Para recuperar os estados de encadeamento jvm, por exemplo:

```
$ curl -s localhost:8080/actuator/metrics/jvm.threads.states | jq .
{
  "name": "jvm.threads.states",
  "description": "The current number of threads having TERMINATED state",
  "baseUnit": "threads",
  "measurements": [
    {
    "statistic": "VALUE",
    "value": 24
    }
  ],
  "availableTags": [
    {
      "tag": "state",
      "values": [
        "timed-waiting",
        "new",
        "runnable",
        "blocked",
        "waiting",
        "terminated"
      ]
    }
  ]
}
```
{: screen}

Para obter mais informações sobre como customizar o comportamento do atuador de métricas, consulte a [documentação oficial](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-metrics.html){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo").

### Ativando o suporte do Prometheus no Spring Boot 2
{: #spring-prometheus-boot2}

O Prometheus requer que o app hospede um terminal no qual o Prometheus Server lê para obter as métricas. A hospedagem desse terminal no Spring conta com o app para entregar os dados. O que significa que o terminal do Prometheus suporta os apps do Spring Boot que usam Spring MVC, Spring WebFlux ou Jersey.

Para ativar o terminal Prometheus, inclua a dependência a seguir em `pom.xml`:
```xml
<dependency>
  <groupId>io.micrometer</groupId>
  <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```
{: codeblock}

O Spring vê a dependência e configura automaticamente o `PrometheusMeterRegistry` como um dos registros configurados no bean global `MeterRegistry`. No entanto, como com o terminal de métricas, o terminal do Prometheus deve ser ativado e exposto para o acesso à web, incluindo o seguinte em `application.properties`:

```properties
management.endpoints.web.exposure.include=health,metrics,prometheus
```
{: codeblock}

Verifique se o terminal Prometheus está ativado verificando o terminal `/actuator`. A saída será semelhante à saída a seguir se você executar o app localmente:

```
$ curl -s localhost:8080/actuator | jq .
{
  "_links": {
    "self": {
      "href": "http://localhost:8080/actuator",
      "templated": false
    },
    ...
    "prometheus": {
      "href": "http://localhost:8080/actuator/prometheus",
      "templated": false
    },
    ...
  }
}
```
{: screen}

O terminal `/actuator/prometheus` emite todas as métricas configuradas no formato de scrape do Prometheus:

```
$ curl -s localhost:8080/actuator/prometheus
# HELP system_cpu_usage The "recent cpu usage" for the whole system
# TYPE system_cpu_usage gauge
system_cpu_usage 0.046231032221805926
...
```
{: screen}

Inclua as anotações a seguir em sua definição de serviço para permitir a extração do Prometheus para o terminal de métricas:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: ...
  annotations:
    prometheus.io/scrape: 'true'
    prometheus.io/scheme: http
    prometheus.io/path: /actuator/prometheus
    prometheus.io/port: "8080"
  ...
```
{: codeblock}

### Configurando métricas no Spring Boot 1
{: #spring-metrics-boot1}

O terminal `/metrics` do Spring Boot Actuator é ativado por padrão no Spring Boot 1, mas é considerado "sensível" e requer autorização. Para alguns ambientes, a proteção do terminal de métricas pode ser a resposta certa, mas a autorização de cada solicitação para o terminal de métricas apresenta muita latência para verificação automatizada com o Kubernetes.

Desative o atributo sensível do terminal de métricas para remover o requisito de autorização, incluindo o seguinte em `application.properties`:
```properties
endpoints.metrics.sensitive=false
```
{: codeblock}

Agora, o suporte de métricas padrão do Spring Boot 1 está ativado, o que produz a saída a seguir:
```
$ curl -s localhost:8080/metrics | jq .
{
  "names": [
    "jvm.memory.max",
    "jvm.threads.states",
    "process.files.max",
    "jvm.gc.memory.promoted",
    ...
  ]
}
```
{: screen}

Para obter mais informações sobre como customizar o comportamento do atuador de métricas do Spring Boot 1, consulte a [documentação oficial](https://docs.spring.io/spring-boot/docs/1.5.x/reference/html/production-ready-metrics.html){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo").

### Ativando o suporte do Prometheus no Spring Boot 1
{: #spring-prometheus-boot1}

O suporte do Prometheus não é construído nas métricas do Spring Boot 1 Actuator, mas pode ser incluído com o Micrômetro.

Conforme mencionado na [visão geral](#spring-metrics), o Micrometer foi relatado ao Spring Boot 1 porque ele tem um conjunto mais rico de primitivas do medidor e um melhor suporte para sistemas de monitoramento, como Prometheus e StatsD. O uso do Micrômetro com o Spring Boot 1 é direto: requer uma biblioteca do adaptador específica e pelo menos um registro. Inclua o seguinte em seu `pom.xml` a fim de incluir dependências para a biblioteca do adaptador e o registro do Prometheus:

```xml
<properties>
  ...
  <micrometer.version>1.1.1</micrometer.version>
</properties>
<dependencies>
  <dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-spring-legacy</artifactId>
    <version>${micrometer.version}</version>
  </dependency>
  <dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
    <version>${micrometer.version}</version>
  </dependency>
</dependencies>
```
{: codeblock}

Essas dependências criam e expõem um terminal de métricas `/prometheus`. Assim como com o terminal `/metrics` padrão, o terminal `/prometheus` é considerado sensível por padrão. Para remover o requisito de autorização, inclua o seguinte em `application.properties`:

```properties
endpoints.prometheus.sensitive=false
```
{: codeblock}

A saída que é produzida pelo terminal `/prometheus` é semelhante ao Spring Boot 2:

```
$ curl -s localhost:8080/prometheus
# HELP jvm_memory_used_bytes The amount of used memory
# TYPE jvm_memory_used_bytes gauge
jvm_memory_used_bytes{area="heap",id="PS Survivor Space",} 2.2594152E7
...
```
{: screen}

Inclua as anotações a seguir em sua definição de serviço para permitir a extração do Prometheus para o terminal de métricas:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: …
  annotations:
    prometheus.io/scrape: 'true'
    prometheus.io/scheme: http
    prometheus.io/path: /prometheus
    prometheus.io/port: "8080"
  ...
```
{: codeblock}

Para obter mais informações sobre o uso do Micrometer com o Spring Boot 1, consulte [https://micrometer.io/docs/ref/spring/1.5](https://micrometer.io/docs/ref/spring/1.5){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo").

## Chamadas de sincronização com o Micrometer
{: #spring-metrics-timing}

O Spring MVC, o WebFlux e o Jersey Server, cada um, fornecerá a configuração automática para o Spring que coletará automaticamente informações de sincronização para todas as solicitações quando a propriedade `management.metrics.web.server.auto-time-requests` estiver configurada como `true`. A configuração padrão é `true`.

```properties
management.metrics.web.server.auto-time-requests=true
```
{: codeblock}

O Micrometer também suporta a anotação `@Timed`, que pode ser usada para coletar informações de sincronização para chamadas de método suportadas. É importante observar que essa anotação não pode ser usada para métodos arbitrários de tempo, uma vez que requer que o suporte do contêiner colete os dados. As anotações `@Timed` são suportadas para operações que são suportadas por um contêiner do servlet, por exemplo, métodos de serviço Spring MVC, Webflux ou Jersey.

Para ser mais seletivo sobre quais informações de sincronização são coletadas, configure a propriedade `management.metrics.web.server.auto-time-requests` como `false` e use a anotação `@Timed` em métodos de serviço individuais ou controladores:

```java
@Timed
@GetMapping("/myendpoint")
public String getSomething() { ... }
```
{: codeblock}

Se `management.metrics.web.server.auto-time-requests` for `true`, a anotação `@Timed` poderá ser usada para incluir tags extras em uma determinada métrica relatada, por exemplo:

```java
@Timed(extraTags = { "mytagname", "mytagvalue"})
@GetMapping("/myendpoint")
public String getSomething() { ... }
```
{: codeblock}

## Métricas customizadas com Spring e Micrometer
{: #spring-metrics-custom}

Para fazer medidas específicas do domínio do aplicativo, é necessário criar suas próprias métricas customizadas. O Micrometer define alguns tipos básicos do `Meter`:

* um `Counter` rastreia um valor que pode aumentar;
* um `Gauge` mede e retorna o valor observado quando o medidor é publicado (ou consultado);
* um `Timer` rastreia quantas vezes um evento ocorre e o tempo decorrido acumulativo para esse evento; 
* um `DistributionSummary` é semelhante a um `Timer`, mas também rastreia uma distribuição de eventos e não mede o tempo.

Novos medidores são criados usando os métodos auxiliares no `MeterRegistry` ou usando o construtor fluente do Medidor. 

Crie o seu `Meter` no construtor (passando o `MeterRegistry` como um parâmetro) ou em um método `@PostConstruct` depois que o `MeterRegistry` for injetado.

Por exemplo, para usar um `Counter` dentro de um `RestController`, é possível fazer algo como a seguir;

```java
@RestController
public class MyController {
    
    private final Counter myCounter;
    
    public MyController(MeterRegistry meterRegistry) {
    
        // Create the counter using the helper method on the builder
        myCounter = meterRegistry.counter("my.counter", "mytagname", "mytagvalue");
        
    }
```
{: codeblock}

Em seguida, dentro de seus métodos de serviço, é possível atualizar o valor do medidor:

```java
@GetMapping("/")
public String getSomething() {
    myCounter.increment();
    ...
}
```
{: codeblock}


Para saber mais sobre como criar métricas customizadas no Spring:

* [Métricas de Spring Boot](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-metrics.html){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo")
* [Conceitos do Micrometer](https://micrometer.io/docs/concepts){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo")
