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

# Spring 的容錯
{: #spring-tolerance}

建立回復型系統會將需求放在其中的所有服務上。雲端環境的動態性需要將服務設計成已準備好（並溫和地回應）非預期情況。

Spring 使用 [Netflix Hystrix](https://github.com/Netflix/Hystrix/wiki){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示") 容錯程式庫來支援應用程式層次的容錯問題。Hystrix 支援撤回、斷路器、隔板及相關聯的度量。 

本資訊以[雲端原生開發：容錯](/docs/java?topic=cloud-native-fault-tolerance#fault-tolerance)中所述的容錯實務，作為建置基礎。
{: note}

## 搭配使用 Hystrix 與 Spring 應用程式
{: #spring-hystrix-starter}

為了使 Hystrix 更容易在 Spring 應用程式內使用，有一個 Spring Boot Starter 可啟用標註的方法來整合應用程式內的 Hystrix。

將這個單一相依關係新增至應用程式，將顯示 Hystrix。 

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```
{: codeblock}

類似於許多其他 Spring Starter，為了向 Spring 指出您要使用應用程式中的功能，可以在主要應用程式類別中新增註釋。若要啟用 Spring 處理 Hystrix 對於電路岔斷的註釋，請新增 `@EnableCircuitBreaker`。

```java
@SpringBootApplication
@EnableCircuitBreaker
public class MyApplication {

	public static void main(String[] args) {
		SpringApplication.run(MyApplication.class, args);
	}
```
{: codeblock}

### 使用 HystrixCommand 來定義撤回
{: #spring-fallback}

若要將電路岔斷行為新增至方法，請使用 `@HystrixCommand` 來加以註釋。Spring 會在標註 `@EnableCircuitBreaker` 的應用程式內找到這些方法，並將它們包裝為 Hystrix 功能，以監視錯誤/逾時，並在必要時呼叫適當的替代行為。 

只有 `@Component` 或 `@Service` 中的方法，才支援 `@HystrixCommand`
{: note}

下列 `@HystrixCommand` 註釋會包裝 `service()` 呼叫，以提供電路岔斷行為。如果 `service()` 方法失敗，或開啟電路，則 Proxy 會呼叫 `fallback()` 方法。

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

藉由參閱 Spring 型 [Hystrix 斷路器](https://spring.io/guides/gs/circuit-breaker/){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示") 範例來進一步瞭解。
{: tip}

### 使用逾時
{: #spring-timeout}

當呼叫遠端服務時，它可能不會立即傳回、可能需要花費一些時間，或者可能永遠不會傳回。Hystrix 可讓您定義應用程式可接受的範圍。簡單地新增至 `HystrixCommand` 註釋，即可變更逾時值（預設值為 1 秒）：

```java
@HystrixCommand(
    fallbackMethod = "fallback",
    commandProperties = {
        @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "30000"),
    }
)
```
{: codeblock}

### 使用隔板
{: #spring-bulkhead}

Hystrix 同時支援信號及佇列型隔板。下列 Snippet 顯示如何配置佇列型隔板（其配置 4 個執行緒並將未完成的要求數限制為 10）：

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

### 斷路器狀態
{: #spring-breaker-status}

Hystrix Spring 入門範本有一個額外的秘訣，它將加強應用程式的預設 `/health` 端點（透過 Spring Actuator 提供，如需相關資訊，請查閱[度量主題](/docs/java?topic=java-spring-metrics#spring-metrics)）。

如果性能端點[配置為包括額外詳細資料](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-endpoints.html#production-ready-health){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")，則斷路器狀態會併入性能檢查資訊中。（依預設會停用此行為）。

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

## 後續步驟
{: #spring-tolerance-next-steps notoc}

如需 Hystrix 配置的相關資訊，請參閱 [Hystrix Configuration Wiki](https://github.com/Netflix/Hystrix/wiki/Configuration){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")。

如需含 Spring 之 Hystrix 的相關資訊，請參閱：

* [A Guide to Spring Cloud Netflix](https://www.baeldung.com/spring-cloud-netflix-hystrix){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")
* [Spring Circuit Breaker Guide](https://spring.io/guides/gs/circuit-breaker/){: new_window}![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")
