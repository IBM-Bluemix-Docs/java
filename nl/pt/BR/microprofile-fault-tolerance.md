---

copyright:
  years: 2019
lastupdated: "2019-03-15"

keywords: fault tolerance microprofile, retries microprofile, circuit breakers microprofile, bulkhead microprofile, microprofile limits

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

# Tolerância a falhas com MicroProfile
{: #mp-fault-tolerance}

Nossa abordagem recomendada para esses tópicos de tolerância a falhas é alavancar principalmente os recursos do Istio. Consulte "Istio -- Tolerância a falhas" no capítulo "Criando microsserviços -- Recursos do Polyglot" deste guia. Isso abrange coisas como novas tentativas, tempos limites, disjuntores, anteparas e limites de taxas.

O MicroProfile também oferece uma abordagem, que é descrita no capítulo "Recursos adicionais Java do Note" sob o título "Tolerância a falhas do MicroProfile". Essa seção mostra como o recurso de fallback pode ser usado em conjunto com o Istio, bem como alguns detalhes adicionais sobre os interessados em uma abordagem que aproveita mpFaultTolerance em vez de Istio.

O Istio não pode oferecer recursos de fallback, porque o fallback requer conhecimento de negócios. Uma abordagem mais avançada é usar a tolerância a falhas do Istio junto com os recursos de fallback do MicroProfile para alcançar o máximo de resiliência. Por exemplo, é possível especificar um backup de fallback ao chamar um serviço de back-end. Se o Istio não conseguir obter um retorno bem-sucedido, o método fallback será chamado.

```java
@GET
@Fallback(fallbackMethod="myFallback")
public String callService() {
    //call servicer b
    return  bclient.hello();
}

public String myFallback() {
    return "Greetings\... This is a fallback!";
}
```
{: codeblock}

Se você usar o recurso de tolerância a falhas do Istio, será possível desativá-lo da Tolerância a falhas do MicroProfile configurando a propriedade de configuração MP_Fault_Tolerance_NonFallback_Enabled como false.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: service-config
data:
  serviceB_host: serviceb-service
  serviceB_http_port: "8080"
  lifetime: "0"
  failFrequency: "0"
  MP_Fault_Tolerance_NonFallback_Enabled: "false"
```
{: codeblock}

Para obter mais informações sobre a comparação do Istio Fault Tolerance e da MicroProfile Fault Tolerance, consulte [este blog](https://www.eclipse.org/community/eclipse_newsletter/2018/september/MicroProfile_istio.php){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") escrito por Emily Jiang.
