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

# Tolerancia a errores con MicroProfile
{: #mp-fault-tolerance}

Nuestro enfoque recomendado para estos temas de tolerancia a errores es principalmente aprovechar las características de Istio. Consulte "Istio -- Tolerancia a errores" en el capítulo "Creación de microservicios -- funciones de varios lenguajes" de este manual. Se tratan temas como los reintentos, los tiempos de espera, los interruptores, las barreras aislantes y los límites de velocidad.

MicroProfile también ofrece un enfoque que se describe en el capítulo "Características destacadas adicionales de Java" bajo la cabecera "Tolerancia de errores de MicroProfile". En esta sección, se muestra cómo se puede utilizar la capacidad de reserva en combinación con Istio, así como algunos detalles más sobre los casos de interés para utilizar un enfoque con mpFaultTolerance en lugar de Istio.

Istio no es capaz de ofrecer funciones de reserva, porque la reserva requiere conocimientos empresariales. Un enfoque más avanzado consiste en utilizar la tolerancia a errores de Istio junto con las funciones de reserva de MicroProfile para lograr el máximo de resiliencia. Por ejemplo, puede especificar una copia de seguridad de reserva al llamar a un servicio de fondo. Si Istio no puede gestionar un retorno satisfactorio, se invocará el método de reserva.

```java
@GET
@Fallback(fallbackMethod="myFallback")
public String callService() {
    //llamar al servicio b
    return  bclient.hello();
}

public String myFallback() {
    return "Greetings\... This is a fallback!";
}
```
{: codeblock}

Si utiliza la capacidad de tolerancia a errores de Istio, puede desactivar la tolerancia a errores de MicroProfile estableciendo la propiedad de configuración MP_Fault_Tolerance_NonFallback_Enabled en false.

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

Para obtener más información sobre la comparación de la tolerancia a errores de Istio Fault Tolerance y la tolerancia a errores de MicroProfile, consulte [este blog](https://www.eclipse.org/community/eclipse_newsletter/2018/september/MicroProfile_istio.php){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo") escrito por Emily Jiang.
