---

copyright:
  years: 2019
lastupdated: "2019-04-23"

keywords: health check jax-rs, jax-rs endpoint, jax-rs status, readiness jax-rs, liveness jax-rs, microprofile health

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

# Verificações de funcionamento com JAX-RS
{: #jaxrs-healthcheck}

Conforme discutido na seção anterior, o Kubernetes fornece múltiplos métodos para integrar as análises de funcionamento, incluindo a execução de um comando, bem como a verificação de rede por meio de terminais TCP ou HTTP. Embora seja possível implementar qualquer uma dessas análises na linguagem Java&trade, uma implementação JAX-RS de análises HTTP e do MicroProfile Health é discutida em detalhes aqui.

## Definindo um terminal JAX-RS de prontidão
{: #mp-readiness}

Um terminal de prontidão mínima é algo semelhante ao exemplo a seguir:

```java
@Path("ready")
public class ReadinessEndpoint {
  @GET
  @Produces(MediaType.APPLICATION_JSON)
  public Response testReadiness() {
    if ( ! ready ) {
      return Response.status(503).entity("{\"status\":\"DOWN\"}").build();
    }
    return Response.ok("{\"status\":\"UP\"}").build();
  }
}
```
{: codeblock}

Um terminal como esse em um aplicativo que é implementado no WebSphere Liberty atinge um determinado nível de comportamento de prontidão sem mais esforço. O Liberty falha as solicitações para o terminal de funcionamento com uma resposta 503 apropriada até que o aplicativo esteja em execução. Essa implementação básica deve ser estendida para incluir recursos necessários. Por exemplo, considere avaliar os recursos CDI do `@ApplicationScoped` para assegurar que o processamento de injeção de dependência tenha sido concluído. Depois que a análise tiver sido implementada, ela poderá ser ativada configurando o tipo de análise HTTP, conforme observado na seção anterior.

Lembre-se sempre da função e do propósito da tolerância a falhas: espera-se que os microsserviços manipulem a falha e a interrupção em comunicações com os serviços de recebimento de dados. Apenas inclua verificações de recursos que não tenham fallback viável dentro de uma verificação de prontidão.

## Definindo um terminal JAX-RS de atividade
{: #jaxrs-liveness}

Uma análise de vivacidade deve ser deliberada sobre o que ela verifica, uma vez que uma falha resulta no término imediato do processo. Um terminal de atividade simples pode ser semelhante ao seguinte:

```java
@Path("liveness")
public class LivenessEndpoint {
    @GET
    @Produces(MediaType.APPLICATION_JSON)
    public Response testLiveness() {
      return Response.ok("{\"status\":\"UP\"}").build();
    }
}
```
{: codeblock}

A implementação da análise de vivacidade pode ser ampliada para consultar as condições locais do processo, que indicam um estado de processo irrecuperável. No entanto, o desafio com tais verificações geralmente reside no fato de que, se o processamento desse tipo for capaz de ser executado, o servidor provavelmente é suficientemente saudável para executar outras solicitações. Por essa razão, um mantra de "mantenha simples" deve ser prontamente aplicado à implementação de verificação de atividade. Depois que as origens de vivacidade são codificadas para ativar o terminal de análise, o terminal de vivacidade HTTP pode ser ativado nas origens de orquestração de contêiner, conforme explicado no último capítulo.

Para evitar os ciclos de reinicialização, o atributo initialDelaySeconds para a verificação de atividade deve ser maior que o tempo de início do servidor mais longo esperado. Para um servidor de aplicativos Java&trade que leva normalmente 30 segundos para ser iniciado, escolha um valor maior, como 60 segundos.

## Usando a API de funcionamento do MicroProfile
{: #mp-health-api}

A [API de funcionamento do MicroProfile](https://www.ibm.com/support/knowledgecenter/en/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/twlp_microprofile_healthcheck.html){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") (mpHealth) define uma estrutura para simplificar a implementação de verificações de funcionamento. A versão 1.0 da API `mpHealth` define um único terminal. A próxima versão fornecerá dois, um para a atividade e um para prontidão.

Para usar a API de funcionamento do MicroProfile com o Liberty, inclua o recurso `mpHealth` em `server.xml`:

```xml
<featureManager>
   <feature>mpHealth-1.0</feature>
</featureManager>
```
{: codeblock}

Em seguida, é possível verificar o status do aplicativo com um navegador acessando o terminal `/health`. O código retorna uma carga útil `{"outcome": "UP"}` quando tudo está bem. O código a seguir mostra como usar as APIs MicroProfile Health para indicar se o pod está funcional ou pronto. O método `call` permite que os desenvolvedores forneçam algoritmo de verificação para indicar o status de funcionamento de um pod. Algo como quase esgotando a memória é um bom indicador para o estado de funcionamento de um pod. A verificação da conexão com o banco de dados de recebimento de dados pode ser uma boa verificação para a prontidão do pod. Se não houver verificação específica, será possível usar o seguinte para indicar se um pod está ativo ou pronto.

```java
import org.eclipse.microprofile.health.Health;
import org.eclipse.microprofile.health.HealthCheck;
import org.eclipse.microprofile.health.HealthCheckResponse;

import javax.enterprise.context.ApplicationScoped;

@Health
@ApplicationScoped
public class HealthResource implements HealthCheck {

    @Override
    public HealthCheckResponse call() {
        return HealthCheckResponse.named("service-a").withData("service-a", "ok").up().build();
    }
}
```
{: codeblock}

Em seguida, é necessário especificar o `livenessProbe` e o `readinessProbe` para o Kubernetes, conforme mostrado abaixo.
```yaml
livenessProbe:
  exec:
    command: ["sh", "-c", "curl -s http://localhost:9080/health| grep -q service-a"]
  initialDelaySeconds: 60
readinessProbe:
  exec:
    command: ["sh", "-c", "curl -s http://localhost:9080/health | grep -q service-a"]
  initialDelaySeconds: 40
```
{: codeblock}

Conforme mencionado anteriormente, é possível usar lógicas diferentes para indicar prontidão e atividade. É possível usar o MicroProfile Health para indicar a verificação de prontidão e, em seguida, criar um terminal JAX-RS para verificação de prontidão ou vice-versa. A próxima versão do MicroProfile Health fornecerá dois terminais, um para a atividade e outro para a prontidão.

O OpenLiberty tem um guia prático que fornece um exemplo de como incluir verificações customizadas no terminal de funcionamento do MicroProfile: https://openliberty.io/guides/microprofile-health.html

É possível substituir o comportamento padrão do terminal de funcionamento do MicroProfile, conforme descrito em [Executando verificações de funcionamento do MicroProfile](https://www.ibm.com/support/knowledgecenter/en/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/twlp_microprofile_healthcheck.html){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo").
