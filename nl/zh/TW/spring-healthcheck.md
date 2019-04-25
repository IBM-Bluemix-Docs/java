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

# 使用 Spring 進行性能檢查
{: #spring-healthcheck}

Spring Boot Actuator 提供的性能端點與應用程式生命週期相關聯：在應用程式啟動之後才可以呼叫到，而且一旦應用程式停止（在處理程序關閉之前發生），便無法使用。Spring Boot Actuator 會自動為在類別路徑上偵測到的技術，新增其他性能指示器。例如，若您的應用程式使用 JDBC，則性能端點會包含一些基本測試，以確保可以呼叫支持的資料儲存庫。基於此原因，「掣動器」定義的性能端點更適合用作備妥探測。

透過將 `spring-boot-actuator` 相依關係新增至 `pom.xml` 檔，來啟用 Spring Boot Actuator：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-actuator</artifactId>
</dependency>
```
{: codeblock}

Spring Boot Actuator 的版本之間有一些行為差異。我們將使用接下來的兩個小節來探索在 Spring Boot 的兩個版本中，如何建立活躍度及備妥檢查，從 2 開始作為最新版本。

## Spring Boot 2 中的性能檢查
{: #spring-health-boot2}

Spring Boot 2 Actuator 定義 `/actuator/health endpoint`，其會在一切正常時傳回 `{"status": "UP"}` 有效負載。依預設會啟用這個端點，且不需要任何應用程式碼。

使用 Spring Boot 2 Actuator 進行未經鑑別的成功性能檢查範例：
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

依預設，顯示時，性能端點會傳回簡單的 UP 或 DOWN 狀態。若為自動化性能檢查，這個簡單的有效負載通常就已足夠。您可以在 `application.properties` 中設定下列內容來啟用詳細的性能資訊：

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

請注意是否包含部分 H2 資料庫資訊。這是 Spring Actuator 自動為支援的服務新增檢查的範例。在此情況下，應用程式使用 JDBC，但在類別路徑上探索到 H2 驅動程式。

您可以將性能端點的預設行為置換為內容或程式碼，如 [Spring Boot Reference Guide for 2.1.x](https://docs.spring.io/spring-boot/docs/2.1.x/reference/html/production-ready-endpoints.html#production-ready-health){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示") 中所述。

### Spring Boot 2 中的活躍度探測
{: #spring-liveness-boot2}

Spring Boot 2 的「掣動器」架構是它自己的迷你 REST 架構，可利用自訂端點來延伸。這表示我們可以新增簡單的活躍度端點，其管理方法同於內建的性能指示器。自訂的活躍度端點可以簡單宣告，如下所示：

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

這個自訂端點必須公開為 Web 端點：

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

### 在 Kubernetes 中配置 Spring Boot 2 備妥及活躍度探測
{: #spring-probe-config-2}

將備妥及活躍度探測配置新增至 Kubernetes 部署資訊清單中的容器定義（請注意路徑及埠值）：

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

為了避免重新啟動循環，請將 `livenessProbe.initialDelaySeconds` 設為比起始設定服務更長，且長度必須足夠安全。然後您便可以針對 `readinessProbe.initialDelaySeconds` 使用較短的值，只要服務就緒便將要求遞送給服務。

## Spring Boot 1 中的性能檢查
{: #spring-health-boot1}

Spring Boot 1 Actuator 定義 `/health` 端點，其會在一切正常時傳回 `{"status": "UP"}` 有效負載。此端點不需要授權，但如果已配置授權且授權了呼叫者，則額外的資訊會包含在回應中。

使用 Spring Boot 1 Actuator 進行未經鑑別的成功性能檢查範例：

```
$ curl -i localhost:8080/health
HTTP/1.1 200
X-Application-Context: application
Content-Type: application/vnd.spring-boot.actuator.v1+json;charset=UTF-8
Content-Length: 15

{"status":"UP"}
```
{: screen}

已鑑別的成功性能檢查範例：

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

您可以將性能端點的預設行為置換為內容或程式碼，如 [Spring Boot Reference v1.5.x](https://docs.spring.io/spring-boot/docs/1.5.9.RELEASE/reference/htmlsingle/#_auto_configured_healthindicators){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示") 中所述。

### Spring Boot 1 中的活躍度探測
{: #spring-liveness-boot1}

對於 Spring Boot 1，簡單的活躍度 http 端點一律傳回 {"status": "UP"} 與狀態碼 200，看起來如下所示：

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

### 在 Kubernetes 中配置 Spring Boot 1 備妥及活躍度探測
{: #spring-probe-config-1}

將備妥及活躍度探測配置新增至 Kubernetes 部署資訊清單中的容器定義（請注意路徑及埠值）：

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

為了避免重新啟動循環，請將 `livenessProbe.initialDelaySeconds` 設為比起始設定服務更長，且長度必須足夠安全。然後您便可以針對 `readinessProbe.initialDelaySeconds` 使用較短的值，只要服務就緒便將要求遞送給服務。
