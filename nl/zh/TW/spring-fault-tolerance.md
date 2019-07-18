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

# Spring 的容錯
{: #spring-tolerance}

建立回復型系統會將需求放在其中的所有服務上。雲端環境的動態本質需要服務設計成溫和地回應非預期情況。

Spring 使用 [Netflix Hystrix](https://github.com/Netflix/Hystrix/wiki){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示") 容錯程式庫來支援應用程式層次的容錯問題。您可以使用 Hystrix，來建立撤回、斷路器、隔板及相關聯的度量值。

本資訊以[雲端原生開發：容錯](/docs/java?topic=cloud-native-fault-tolerance#fault-tolerance)中所述的容錯實務，作為建置基礎。
{: note}

## 搭配使用 Hystrix 與 Spring 應用程式
{: #spring-hystrix-starter}

為了使 Hystrix 更容易在 Spring 應用程式內使用，有一個 Spring Boot Starter 可啟用標註的方法來整合應用程式內的 Hystrix。

若要新增 Hystrix，請新增下列相依關係：

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```
{: codeblock}

若要讓 Spring 可以使用這個新功能，請將註釋新增至您的主要應用程式類別，並讓 Spring 可以新增 `@EnableCircuitBreaker`，來處理斷路的 Hystrix 註釋。

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

若要將斷路器行為新增至方法，請使用 `@HystrixCommand` 來加以註釋。Spring 會在以 `@EnableCircuitBreaker` 標註的應用程式內探索方法，並用 Hystrix 支援包裝它們，來監視是否發生錯誤及逾時。然後，Spring 會在需要時呼叫適當的替代行為。

只有 `@Component` 或 `@Service` 中的方法，才支援 `@HystrixCommand` 註釋。
{: note}

下列 `@HystrixCommand` 註釋會包裝 `service()` 呼叫，以提供斷路器行為。如果 `service()` 方法失敗，或是線路未封閉，則 Proxy 會呼叫 `fallback()` 方法。

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

您的應用程式如何回應未回應的遠端服務？可能只需等待一會兒，或需要永久等待。Hystrix 可讓您定義應用程式可接受的等待時間量。

簡單地新增至 `HystrixCommand` 註釋，即可將逾時值從預設值 1 秒變更為 30 秒：

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

Hystrix 同時支援信號及佇列型隔板。下列 Snippet 顯示如何配置佇列型隔板（其配置四個執行緒並將未完成的要求數限制為 10）：

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

Hystrix Spring 入門範本有一個額外的秘訣，它將加強應用程式的預設 `/health` 端點（透過 Spring 掣動器提供的端點）。如需相關資訊，請參閱[度量值與 Spring](/docs/java?topic=java-spring-metrics#spring-metrics)。

如果性能端點[配置為包括額外詳細資料](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-endpoints.html#production-ready-health){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")，則斷路器狀態會與性能檢查資訊一起包含。（依預設會停用此行為）。

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
