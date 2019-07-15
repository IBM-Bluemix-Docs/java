---

copyright:
  years: 2019
lastupdated: "2019-06-10"

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

A abordagem recomendada para estes tópicos de tolerância a falhas é usar os recursos do Istio. Confira o livro intitulado "Istio - Tolerância a falhas" e navegue para o capítulo "Criando microsserviços - Recursos poliglotas". São explicados muitos tópicos que incluem novas tentativas, tempos limites, disjuntores, anteparas e limites de taxa.

O MicroProfile também oferece uma abordagem, que é descrita no capítulo "Recursos do Java adicionais do Note" sob o título "Tolerância a falhas do MicroProfile". Essa seção mostra como o recurso de fallback pode ser usado junto com o Istio, bem como detalhes sobre o uso de `mpFaultTolerance` em vez de Istio.

O Istio não é capaz de oferecer recursos de fallback porque o fallback requer conhecimento de negócios. Uma abordagem mais avançada é usar a tolerância a falhas do Istio junto com os recursos de fallback do MicroProfile para alcançar o máximo de resiliência. Por exemplo, será possível especificar um backup de fallback quando você chamar um serviço de back-end. Se o Istio não conseguir obter um retorno bem-sucedido, o método de fallback será chamado.

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

Se você usar a capacidade de tolerância a falhas do Istio, será possível desativá-las de Tolerância a falhas do MicroProfile configurando a propriedade de configuração MP_Fault_Tolerance_NonFallback_Enabled como false.

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

Para obter mais informações sobre a comparação de Tolerância a falhas do Istio e de Tolerância a falhas do MicroProfile, consulte [este blog](https://www.eclipse.org/community/eclipse_newsletter/2018/september/MicroProfile_istio.php){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") que é escrito por Emily Jiang.
