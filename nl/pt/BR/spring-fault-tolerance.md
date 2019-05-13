---

copyright:
  years: 2018, 2019
lastupdated: "2019-04-22"

keywords: fault tolerance spring, hystrix spring, netflix spring, hystrixcommand spring, bulkhead spring, circuit breaker spring

subcollection: java

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}

# Tolerância a falhas com o Spring
{: #spring-tolerance}

A criação de um sistema resiliente impõe requisitos em todos os serviços dentro dele. A natureza dinâmica dos ambientes de nuvem exige que os serviços sejam projetados para responder de forma normal ao inesperado.

O Spring usa a biblioteca de tolerância a falhas do [Netflix Hystrix](https://github.com/Netflix/Hystrix/wiki){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") para suportar preocupações de tolerância a falhas no nível do aplicativo. Com o Hystrix, é possível criar fallbacks, disjuntores, anteparas e métricas associadas.

Essas informações são construídas sobre as práticas de tolerância a falhas que estão descritas em [Desenvolvimento nativo de nuvem: tolerância a falhas](/docs/java?topic=cloud-native-fault-tolerance#fault-tolerance).
{: note}

## Usando o Hystrix com um Aplicativo Spring
{: #spring-hystrix-starter}

Para tornar o Hystrix mais fácil de usar em um aplicativo Spring, há um Spring Boot Starter que permite que uma abordagem anotada integre o Hystrix a seu aplicativo.

Para incluir o Hystrix, inclua a dependência a seguir:

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```
{: codeblock}

Para que o Spring use essa nova função, inclua uma anotação em sua classe de aplicativo principal e ative o processamento da Spring das anotações do Hystrix para a quebra de circuito, incluindo `@EnableCircuitBreaker`.

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

Para incluir o comportamento do disjuntor em um método, anote-o com `@HystrixCommand`. O Spring descobre métodos em um aplicativo que é anotado com `@EnableCircuitBreaker` e agrupa-os com o suporte do Hystrix para monitorar erros e tempos limites. Em seguida, o Spring chama o comportamento alternativo apropriado quando necessário.

A anotação `@HystrixCommand` é suportada somente em métodos em um `@Component` ou `@Service`.
{: note}

A anotação `@HystrixCommand` a seguir agrupa a chamada `service()` para fornecer o comportamento do disjuntor. O proxy chamará o método `fallback()` se o método `service()` falhar ou se o circuito estiver aberto.

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

### Usando Tempos Limites
{: #spring-timeout}

Como seu aplicativo responde a um serviço remoto que não está respondendo? A espera pode demorar um pouco ou pode levar uma eternidade. O Hystrix fornece a capacidade de definir qual quantia de tempo de espera é aceitável para seu aplicativo.

Uma adição simples à anotação `HystrixCommand` pode ser usada para mudar o valor de tempo limite do valor padrão de 1 segundo a 30 segundos:

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

O Hystrix suporta as anteparas semáforos e baseadas em fila. O fragmento a seguir mostra como configurar uma antepara baseada em fila que aloca quatro encadeamentos e limita o número de solicitações pendentes para 10:

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

O iniciador do Hystrix Spring tem um truque extra na manga, ele aprimora o terminal `/health` padrão para o aplicativo, que é fornecido por meio de um Spring Actuator. Para obter mais informações, consulte [Métricas com Spring](/docs/java?topic=java-spring-metrics#spring-metrics)).

Se o terminal de funcionamento estiver [configurado para incluir detalhes extras](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-endpoints.html#production-ready-health){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo"), o status do disjuntor será incluído com as informações de verificação de funcionamento. (Esse comportamento é desativado por padrão).

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
