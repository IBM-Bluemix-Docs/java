---

copyright:
  years: 2018, 2019
lastupdated: "2019-03-15"

keywords: health check spring, spring health endpoint, spring-boot-actuator, liveness probe spring, readiness probe spring, spring kubernetes probe

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

# Verificações de funcionamento com Spring
{: #spring-healthcheck}

O terminal de funcionamento fornecido pelo atuador do Spring Boot é ligado ao ciclo de vida do aplicativo: ele não estará acessível até que o aplicativo tenha sido iniciado e estará indisponível assim que o aplicativo for interrompido (o que acontece antes que o processo seja encerrado). O Spring Boot Actuator incluirá automaticamente indicadores de funcionamento adicionais para tecnologias detectadas no caminho de classe. Por exemplo, se seu aplicativo usar JDBC, o terminal de funcionamento incluirá alguns testes básicos para assegurar que o armazenamento de dados de suporte possa ser acessado. O terminal de funcionamento definido pelo Actuator é mais adequado para uso como uma análise de prontidão para esse motivo.

Ative o Spring Boot Actuator incluindo a dependência `spring-boot-actuator` em seu arquivo `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-actuator</artifactId>
</dependency>
```
{: codeblock}

Há algumas diferenças de comportamento com o atuador do Spring Boot entre as versões. Usaremos as duas próximas seções para explorar como criar verificações de atividade e prontidão em ambas as versões do Spring Boot, começando com 2 como a mais atual.

## Verificações de funcionamento do Spring Boot 2
{: #spring-health-boot2}

O atuador do Spring Boot 2 define um `/actuator/health endpoint`, que retorna uma carga útil `{"status": "UP"}` quando tudo está bem. Esse terminal é ativado por padrão e não requer nenhum código do aplicativo.

Um exemplo de uma verificação de funcionamento bem-sucedida não autenticada usando o Spring Boot 2 Actuator:
<!-- Spring Boot 2 test project: https://github.com/IBM/spring-alarm-application -->

```
$ curl -i localhost:8080/actuator/health
HTTP/1.1 200
Content-Type: application/vnd.spring-boot.actuator.v2+json;charset=UTF-8
Transfer-Encoding: chunked
Date: Fri, 07 Dec 2018 23:09:09 GMT

{"status":"UP"}
```
{: screen}

Conforme mostrado, o terminal de funcionamento retorna um status de UP ou DOWN simples por padrão. Para verificação de funcionamento automatizada, essa carga útil simples geralmente é suficiente. As informações detalhadas de funcionamento podem ser ativadas configurando a propriedade a seguir em `application.properties`:

```properties
management.endpoint.health.show-details=always
```
{: codeblock}

```
$ curl -i localhost:8080/actuator/health
HTTP/1.1 200
Content-Type: application/vnd.spring-boot.actuator.v2+json;charset=UTF-8
Transfer-Encoding: chunked
Date: Fri, 07 Dec 2018 23:09:09 GMT

{"status":"UP","details":{"diskSpace":{"status":"UP","details":{"total":500068036608,"free":407611420672,"threshold":10485760}},"db":{"status":"UP","details":{"database":"H2","hello":1}}}}
```
{: screen}

Observe a inclusão de algumas informações de base de dados H2. Este é um exemplo de Spring Actuator que inclui automaticamente verificações para serviços auxiliares. Nesse caso, o aplicativo está usando JDBC, e o driver H2 foi descoberto no caminho de classe.

É possível substituir o comportamento padrão do terminal de funcionamento com propriedades ou código, conforme descrito no [Guia de referência do Spring Boot para 2.1.x](https://docs.spring.io/spring-boot/docs/2.1.x/reference/html/production-ready-endpoints.html#production-ready-health){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo").

### Análise de atividade no Spring Boot 2
{: #spring-liveness-boot2}

A estrutura do Actuator no Spring Boot 2 é sua própria estrutura mini-REST que pode ser estendida com terminais customizados. Isso significa que podemos incluir um terminal de atividade simples que é gerenciado da mesma maneira que o indicador de funcionamento integrado. Um terminal de atividade customizado pode ser declarado trivialmente assim:

```java
@Endpoint(id="liveness")
@Component
public class Liveness {
    @ReadOperation
    public String testLiveness() {
            return "{\"status\":\"UP\"}";
    }
}
```
{: codeblock}

Esse terminal customizado deve ser exposto como um terminal da web:

```properties
management.endpoints.web.exposure.include=health,liveness,...
```
{: codeblock}

```
$ curl -i localhost:8080/actuator/liveness
HTTP/1.1 200
Content-Type: application/vnd.spring-boot.actuator.v2+json;charset=UTF-8
Content-Length: 15
Date: Thu, 07 Feb 2019 22:38:27 GMT

{"status":"UP"}
```
{: screen}

### Configurando as análises de prontidão e de atividade do Spring Boot 2 em Kubernetes
{: #spring-probe-config-2}

Inclua a configuração de análise de prontidão e de atividade na definição de contêiner em um manifest de implementação do Kubernetes (observe os valores de caminho e porta):

```yaml
spec:
  containers:
  - name: ...
    image: ...
    readinessProbe:
      httpGet:
        path: /actuator/health
        port: 8080
      initialDelaySeconds: 60
      timeoutSeconds: 5
    livenessProbe:
      httpGet:
        path: /actuator/liveness
        port: 8080
      initialDelaySeconds: 130
      timeoutSeconds: 10
      failureThreshold: 10
```
{: codeblock}

Para evitar os ciclos de reinicialização, configure `livenessProbe.initialDelaySeconds` para ser mais longo, de forma segura, do que a inicialização do serviço. Em seguida, é possível usar um valor mais curto para `readinessProbe.initialDelaySeconds` para rotear as solicitações para o serviço assim que ele estiver pronto.

## Verificações de funcionamento no Spring Boot 1
{: #spring-health-boot1}

O atuador do Spring Boot 1 define um terminal `/health`, que retorna uma carga útil `{"status": "UP"}` quando tudo está bem. O terminal não requer autorização, mas se a autorização estiver configurada e o responsável pela chamada estiver autorizado, informações adicionais serão incluídas na resposta.

Um exemplo de uma verificação de funcionamento bem-sucedida não autenticada usando o Spring Boot 1 Actuator:

```
$ curl -i localhost:8080/health
HTTP/1.1 200
X-Application-Context: application
Content-Type: application/vnd.spring-boot.actuator.v1+json;charset=UTF-8
Content-Length: 15

{"status":"UP"}
```
{: screen}

Um exemplo de uma verificação de funcionamento bem-sucedida autenticada:

```
$ curl -i localhost:8080/health
HTTP/1.1 200 OK
X-Application-Context: application
Content-Type: application/vnd.spring-boot.actuator.v1+json;charset=UTF-8
Content-Length: 221

{
    "status" : "UP",
  "diskSpace" : {
        "status" : "UP",
    "total" : 63251804160,
    "free" : 31316164608,
    "threshold" : 10485760
    },
  "db" : {
        "status" : "UP",
    "database" : "H2",
    "hello" : 1
    }
}
```
{: screen}

É possível substituir o comportamento padrão do terminal de funcionamento com propriedades ou código, conforme descrito em [Guia de Referência do Spring Boot v1.5.x](https://docs.spring.io/spring-boot/docs/1.5.9.RELEASE/reference/htmlsingle/#_auto_configured_healthindicators){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo").

### Análise de atividade no Spring Boot 1
{: #spring-liveness-boot1}

Para o Spring Boot 1, um terminal http simples para a atividade que sempre retorna {"status": "UP"} com código de status 200 seria algo semelhante a isto:

```java
@RestController
public class Liveness {
    private static Map<String, String> alwaysOk = Collections.singletonMap("status", "OK");

    @GetMapping("/liveness")
    public Map<String,String> testLiveness() {
        return alwaysOk;
    }
}
```
{: codeblock}

```
$ curl -i localhost:8080/liveness
HTTP/1.1 200
Content-Type: application/json;charset=UTF-8
Transfer-Encoding: chunked
Date: Thu, 07 Feb 2019 22:43:32 GMT

{"status":"OK"}
```
{: screen}

### Configurando as análises de prontidão e de atividade do Spring Boot 1 em Kubernetes
{: #spring-probe-config-1}

Inclua a configuração de análise de prontidão e de atividade na definição de contêiner em um manifest de implementação do Kubernetes (observe os valores de caminho e porta):

```yaml
spec:
  containers:
  - name: ...
    image: ...
    readinessProbe:
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 60
      timeoutSeconds: 5
    livenessProbe:
      httpGet:
        path: /liveness
        port: 8080
      initialDelaySeconds: 130
      timeoutSeconds: 10
      failureThreshold: 10
```
{: codeblock}

Para evitar os ciclos de reinicialização, configure `livenessProbe.initialDelaySeconds` para ser mais longo, de forma segura, do que a inicialização do serviço. Em seguida, é possível usar um valor mais curto para `readinessProbe.initialDelaySeconds` para rotear as solicitações para o serviço assim que ele estiver pronto.
