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

# Tolerancia a errores con Spring
{: #spring-tolerance}

La creación de un sistema resistente impone requisitos en todos los servicios que hay dentro de él. La naturaleza dinámica de los entornos en la nube demanda que los servicios estén diseñados para responder de forma adecuada a lo inesperado.

Spring utiliza la biblioteca de tolerancia a errores de [Netflix Hystrix](https://github.com/Netflix/Hystrix/wiki){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo") para dar soporte a problemas de tolerancia a errores de nivel de aplicación. Con Hystrix, puede crear retrocesos, interruptores, barreras y métricas asociadas.

Esta información se basa en las prácticas de tolerancia a errores que se describen en [Desarrollo nativo en la nube: tolerancia a errores](/docs/java?topic=cloud-native-fault-tolerance#fault-tolerance).
{: note}

## Utilización de Hystrix con una aplicación Spring
{: #spring-hystrix-starter}

Para facilitar el uso de Hystrix dentro de una aplicación Spring, hay un Spring Boot Starter que habilita un enfoque anotado para integrar Hystrix dentro de su aplicación.

Para añadir Hystrix, añada la dependencia siguiente:

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```
{: codeblock}

Para que Spring pueda utilizar esta nueva función, añada una anotación en su clase de aplicación principal y habilite el proceso de Spring de las anotaciones de Hystrix para interruptores añadiendo `@EnableCircuitBreaker`.

```java
@SpringBootApplication
@EnableCircuitBreaker
public class MyApplication {

	public static void main(String[] args) {
		SpringApplication.run(MyApplication.class, args);
	}
```
{: codeblock}

### Uso de HystrixCommand para definir una reserva
{: #spring-fallback}

Para añadir un comportamiento de interruptor a un método, anótelo con `@HystrixCommand`. Spring descubre métodos dentro de una aplicación que están anotados con `@EnableCircuitBreaker` y los envuelve con el soporte de Hystrix para supervisar errores y tiempos de espera. A continuación, Spring invoca el comportamiento alternativo adecuado cuando sea necesario.

La anotación `@HystrixCommand` solo se admite en métodos de un `@Component` o `@Service`.
{: note}

La anotación `@HystrixCommand` siguiente envuelve la invocación de `service()` para proporcionar un comportamiento de interruptor. El proxy llama al método `fallback()` si falla el método `service()` o si el circuito está abierto.

```java
@Autowired
private MicroService myService;

@GetMapping("/endpoint")
public String value() throws Exception {
    return "Service returned: " + myService.service();
}

@Service
class MicroService {
    @HystrixCommand(fallbackMethod = "fallback")
    public String service() {
        // Realizar algún proceso. Si se genera una excepción
        // se llamará al método fallback
        return "Success";
    }

    public String fallback(Throwable e) {
        return "Fallback! Reason: " + e.getMessage() + "\n";
    }
}
```
{: codeblock}

Obtenga más información consultando el ejemplo de [Hystrix Circuit Breaker](https://spring.io/guides/gs/circuit-breaker/){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo") basado en Spring.
{: tip}

### Utilización de tiempos de espera
{: #spring-timeout}

¿Cómo responde la aplicación a un servicio remoto que no responde? La espera puede durar un poco, o durar para siempre. Hystrix le da la posibilidad de definir la cantidad de tiempo de espera que es aceptable para su aplicación.

Se puede utilizar una adición simple a la anotación `HystrixCommand` para cambiar el valor de tiempo de espera del valor predeterminado de 1 segundo a 30 segundos:

```java
@HystrixCommand(
    fallbackMethod = "fallback",
    commandProperties = {
        @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "30000"),
    }
)
```
{: codeblock}

### Utilización de barreras aislantes
{: #spring-bulkhead}

Hystrix soporta las barreras aislantes basadas en cola y semáforo. El fragmento de código siguiente muestra cómo configurar una barrera aislante basada en cola que asigna cuatro hebras y limita el número de solicitudes pendientes a 10:

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

### Estado del interruptor automático
{: #spring-breaker-status}

El iniciador de Spring de Hystrix tiene un truco adicional bajo la manga, mejora el punto final
`/health` predeterminado para la aplicación, que se proporciona a través de Spring Actuator. Para obtener más información, consulte
[Métricas con Spring](/docs/java?topic=java-spring-metrics#spring-metrics)).

Si el punto final de estado está [configurado para incluir detalles adicionales](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-endpoints.html#production-ready-health){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo"), el estado del interruptor se incluye con la información de comprobación de estado. (Este comportamiento está inhabilitado de forma predeterminada).

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

## Pasos siguientes
{: #spring-tolerance-next-steps notoc}

Para obtener más información sobre la configuración de Hystrix, consulte el [wiki de configuración de Hystrix](https://github.com/Netflix/Hystrix/wiki/Configuration){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo").

Para obtener más información sobre Hystrix con Spring, consulte:

* [A Guide to Spring Cloud Netflix](https://www.baeldung.com/spring-cloud-netflix-hystrix){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo")
* [Spring Circuit Breaker Guide](https://spring.io/guides/gs/circuit-breaker/){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo")
