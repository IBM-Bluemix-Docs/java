---

copyright:
  years: 2019
lastupdated: "2019-05-20"

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

# 使用 JAX-RS 進行性能檢查
{: #jaxrs-healthcheck}

如之前章節中所述，Kubernetes 提供多種方法來整合性能探測，包括執行指令，以及透過 TCP 或 HTTP 端點進行網路檢查。雖然您可以用 Java&trade; 語言實作這些探測，但這裡會詳細討論 HTTP 探測與 MicroProfile Health 的 JAX-RS 實作。

## 定義備妥 JAX-RS 端點
{: #mp-readiness}

最小的備妥端點看起來如下列範例所示：

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

對於 WebSphere Liberty 中部署的應用程式，端點可達成一定程度的準備行為，而不需要額外的努力。在應用程式執行之前，Liberty 無法對性能端點提出要求，並傳回適當的 503 回應。這個基本實作可以加以延伸，以包含必要的功能。例如，請考量評估 `@ApplicationScoped` CDI 資源，以確保相依關係注入處理完整。實作探測之後，即可透過配置 HTTP 探測類型（如之前小節中所附註）來啟用它。

請務必記住容錯的角色及目的：預期微服務會處理與下游服務通訊時所發生的失敗與中斷。您必須包含檢查，看看是否有在備妥檢查中無法撤回的功能。

## 定義存活性 JAX-RS 端點
{: #jaxrs-liveness}

存活性探測對於檢查的對象很慎重，因為失敗會導致立即終止處理程序。簡單的存活性端點看起來如下列範例所示：

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

可以延伸存活性探測實作，以檢查處理程序本端狀況，其會指出無法復原的處理程序狀態。不過，這類檢查的挑戰通常在於如果能夠執行這類處理程序，伺服器很可能運作正常，可以處理其他要求。因此，「保持簡單」口頭禪可以很容易套用至存活性檢查實作。撰寫存活性原始檔的程式碼以啟用探測端點之後，即可在容器編排原始檔中啟用 HTTP 存活性端點，如上一章所述。

為了避免重新啟動循環，用於存活性檢查的 `initialDelaySeconds` 屬性必須比最久的預期伺服器啟動時間還久。對於通常需要 30 秒才能啟動的 Java&trade; 應用程式伺服器，請選擇較大的值，例如 60 秒。

## 使用 MicroProfile Health API
{: #mp-health-api}

[MicroProfileHealth API](https://www.ibm.com/support/knowledgecenter/en/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/twlp_microprofile_healthcheck.html){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示") (mpHealth) 定義一個架構來簡化性能檢查的實作。`mpHealth` API 1.0 版定義單一個端點。下一個版本將提供兩個，一個用於存活性，另一個用於備妥。

若要搭配使用 MicroProfile Health API 與 Liberty，請將 `mpHealth` 特性新增至 `server.xml`：

```xml
<featureManager>
   <feature>mpHealth-1.0</feature>
</featureManager>
```
{: codeblock}

然後，您可以使用瀏覽器，藉由存取 `/health` 端點，來檢查應用程式的狀態。程式碼會在一切正常時傳回有效負載 `{"outcome": "UP"}`。下列程式碼顯示如何使用 MicroProfile Health API 來指出 Pod 是否運作正常。`call` 方法可讓開發人員提供檢查演算法來指出 Pod 的性能狀態。「幾乎耗盡記憶體」之類的內容是個不錯的指標，可以說明 Pod 的性能狀態。檢查下游資料庫連線對於 Pod 的備妥而言，不失為一個不錯的檢查。如果沒有特定的檢查，則可以使用下列來指出 Pod 為作用中或已備妥。

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

然後，您需要為 Kubernetes 指定 `livenessProbe` 及 `readinessProbe`，如下列範例所示。
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

如前所述，您可以使用不同的邏輯來指示備妥和存活性。您可以使用 MicroProfile Health 來指出備妥檢查，然後為了備妥檢查建立一個 JAX-RS 端點，反之亦然。下一版的 MicroProfile Health 將提供兩個端點，一個用於存活性，另一個用於備妥。

OpenLiberty 有一個便利的手冊，提供的範例示範如何將自訂檢查新增至 MicroProfile Health 端點： https://openliberty.io/guides/microprofile-health.html

您可以置換 MicroProfile Health 端點的預設行為，如[執行 MicroProfile 性能檢查](https://www.ibm.com/support/knowledgecenter/en/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/twlp_microprofile_healthcheck.html){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示") 中所述。
