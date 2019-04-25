---

copyright:
  years: 2018, 2019
lastupdated: "2019-03-15"

keywords: fault tolerance spring, hystrix spring, netflix spring, hystrixcommand spring, bulkhead spring, circuit breaker spring

subcollection: java

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}

# Tolerância a falhas com o Spring
{: #spring-tolerance}

A criação de um sistema resiliente impõe requisitos em todos os serviços dentro dele. A natureza dinâmica dos ambientes de nuvem exige que os serviços sejam projetados para serem preparados e, respondam normalmente, ao inesperado.

O Spring usa a biblioteca de tolerância a falhas do [Netflix Hystrix](https://github.com/Netflix/Hystrix/wiki){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") para suportar preocupações de tolerância a falhas no nível do aplicativo. O Hystrix fornece suporte para fallbacks, disjuntores, anteparas e métricas associadas. 

Estas informações são compiladas em práticas de tolerância a falhas descritas em [Desenvolvimento nativo de nuvem: tolerância a falhas](/docs/java?topic=cloud-native-fault-tolerance#fault-tolerance).
{: note}

## Usando o Hystrix com um Aplicativo Spring
{: #spring-hystrix-starter}

Para tornar o Hystrix mais fácil de usar em um aplicativo Spring, há um Spring Boot Starter que permite que uma abordagem anotada integre o Hystrix em seu aplicativo.

Incluir essa única dependência em seu aplicativo o trará o Hystrix. 

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```
{: codeblock}

Semelhante a muitos outros Spring Starters, para indicar ao Spring que você deseja usar a funcionalidade no aplicativo, você inclui uma anotação de sua classe de aplicativo principal. Para ativar o processamento de Spring das anotações Hystrix para disjuntor, inclua `@EnableCircuitBreaker`.

```java
@SpringBootApplication
@EnableCircuitBreaker
public class MyApplication {

	public static void main(String[] args) {
		SpringApplication.run(MyApplication.class, args);
	}
```
{: codeblock}

### Usando HystrixCommand para definir um fallback
{: #spring-fallback}

Para incluir um comportamento de disjuntor em um método, você anota-o com `@HystrixCommand`. O Spring localizará esses métodos em um aplicativo anotado com `@EnableCircuitBreaker` e os agrupará com a funcionalidade Hystrix para monitorar erros/tempos limites e chamar o comportamento alternativo apropriado quando necessário. 

`@HystrixCommand` é suportado somente em métodos em um `@Component` ou `@Service` {: note}

A anotação `@HystrixCommand` a seguir agrupa a chamada de `service()` para fornecer o comportamento do disjuntor. O proxy chamará o método `fallback()` se o método `service()` falhar ou se o circuito estiver aberto.

```java
@Autowired private MicroService myService;

@GetMapping("/endpoint")
public String value() throws Exception {
    return "Service returned: " + myService.service();
}

@Service class MicroService {
    @HystrixCommand(fallbackMethod = "fallback")
    public String service() {
        // Do some processing. If an exception is thrown
        // the fallback method will be called
        return "Success";
    }

    public String fallback(Throwable e) {
        return "Fallback! Reason: " + e.getMessage() + "\n";
    }
}
```
{: codeblock}

Saiba mais, verificando o exemplo do [Hystrix Circuit Breaker](https://spring.io/guides/gs/circuit-breaker/){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") baseado em Spring.
{: tip}

### Usando tempos limites
{: #spring-timeout}

Ao chamar um serviço remoto, talvez ele não retorne imediatamente; ele pode demorar um pouco ou pode demorar muito. O Hystrix fornece a capacidade de definir o que é aceitável para seu aplicativo. Uma adição simples à anotação `HystrixCommand` pode ser usada para mudar o valor de tempo limite do valor padrão de 1 segundo:

```java
@HystrixCommand(
    fallbackMethod = "fallback",
    commandProperties = {
        @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "30000"),
    }
)
```
{: codeblock}

### Usando anteparas
{: #spring-bulkhead}

O Hystrix suporta as anteparas semáforos e baseadas em fila. O fragmento a seguir mostra como configurar uma antepara com base em fila que aloca 4 encadeamentos e limita o número de solicitações pendentes para 10:

```java
@HystrixCommand(
    fallbackMethod = "fallback",
    threadPoolProperties = {
        @HystrixProperty(name = "coreSize", value = "4"),
        @HystrixProperty(name = "maxQueueSize", value = "10")
    }
)
```
{: codeblock}

### Status do disjuntor
{: #spring-breaker-status}

O iniciador Hystrix Spring tem um truque extra na manga: ele aprimorará o terminal padrão `/health` para o aplicativo (fornecido por meio de um Spring Actuator, para saber mais, confira o [Tópico Métricas](/docs/java?topic=java-spring-metrics#spring-metrics)).

Se o terminal de funcionamento estiver [configurado para incluir detalhes extras](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-endpoints.html#production-ready-health){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo"), o status do Disjuntor será incluído com as informações de verificação de funcionamento. (Esse comportamento é desativado por padrão).

```
{
    "hystrix": {
        "openCircuitBreakers": [
            "MicroService::service"
        ],
        "status": "CIRCUIT_OPEN"
    },
    "status": "UP"
}
```
{: screen}

## Próximas etapas
{: #spring-tolerance-next-steps notoc}

Para obter mais informações sobre a configuração do Hystrix, consulte a [Wiki de configuração do Hystrix](https://github.com/Netflix/Hystrix/wiki/Configuration){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo").

Para saber mais sobre o Hystrix com Spring, consulte:

* [Um Guia para o Spring Cloud Netflix](https://www.baeldung.com/spring-cloud-netflix-hystrix){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo")
* [Guia do disjuntor do Spring](https://spring.io/guides/gs/circuit-breaker/){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo")
