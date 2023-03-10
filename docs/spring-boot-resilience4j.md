# Spring Boot resilience 4j 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-resilience4j>

## 1.概观

Resilience4j 是一个轻量级的容错库，它为 web 应用程序提供了多种容错和稳定性模式。在本教程中，我们将**学习如何通过一个简单的 Spring Boot 应用程序**来使用这个库。

## 2.设置

在这一部分，让我们关注**为我们的 Spring Boot 项目**设置关键方面。

### 2.1.Maven 依赖性

首先，我们需要添加`[spring-boot-starter-web](https://web.archive.org/web/20221128052426/https://search.maven.org/search?q=g:org.springframework.boot%20AND%20a:spring-boot-starter-web)`依赖项来引导一个简单的 web 应用程序:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

接下来，**我们需要 [`resilience4j-spring-boot2`](https://web.archive.org/web/20221128052426/https://search.maven.org/search?q=resilience4j-spring-boot2) 和`[spring-boot-starter-aop](https://web.archive.org/web/20221128052426/https://search.maven.org/search?q=spring-boot-starter-aop)`依赖项来使用 Resilience-4j 库中的特性，在我们的 Spring Boot 应用程序中使用注释**:

```java
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-spring-boot2</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

此外，我们还需要添加`[spring-boot-starter-actuator](https://web.archive.org/web/20221128052426/https://search.maven.org/artifact/org.springframework.boot/spring-boot-starter-actuator)`依赖项，通过一组公开的端点来监控应用程序的当前状态:

```java
<dependency>
    <groupId>org.springframework.boot</groupId> 
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

最后，让我们添加`[wiremock-jre8](https://web.archive.org/web/20221128052426/https://search.maven.org/search?q=g:com.github.tomakehurst%20AND%20a:wiremock-jre8)`依赖项，因为它将帮助我们使用[模拟 HTTP 服务器](/web/20221128052426/https://www.baeldung.com/introduction-to-wiremock)测试 REST APIs:

```java
<dependency>
    <groupId>com.github.tomakehurst</groupId>
    <artifactId>wiremock-jre8</artifactId>
    <scope>test</scope>
</dependency>
```

### 2.2.`RestController`和外部 API 调用者

在使用 Resilience4j 库的不同特性时，我们的 web 应用程序需要与外部 API 进行交互。因此，让我们继续为`[RestTemplate](/web/20221128052426/https://www.baeldung.com/rest-template) `添加一个 bean，它将帮助我们进行 API 调用。

```java
@Bean
public RestTemplate restTemplate() {
    return new RestTemplateBuilder().rootUri("http://localhost:9090")
      .build();
}
```

接下来，让我们将`ExternalAPICaller`类定义为一个`Component`，并将 *restTemplate* bean 用作成员:

```java
@Component
public class ExternalAPICaller {
    private final RestTemplate restTemplate;

    @Autowired
    public ExternalAPICaller(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }
}
```

在这之后，我们可以**定义`ResilientAppController`类，该类公开 REST API 端点并在内部使用`ExternalAPICaller` bean 调用外部 API** :

```java
@RestController
@RequestMapping("/api/")
public class ResilientAppController {
    private final ExternalAPICaller externalAPICaller;
}
```

### 2.3.执行器端点

我们可以通过 [Spring Boot 执行器](/web/20221128052426/https://www.baeldung.com/spring-boot-actuators)来**暴露健康端点，从而在任何给定的时间知道应用程序**的确切状态。

因此，让我们将配置添加到`application.properties` 文件并启用端点:

```java
management.endpoints.web.exposure.include=*
management.endpoint.health.show-details=always

management.health.circuitbreakers.enabled=true
management.health.ratelimiters.enabled=true
```

此外，当我们需要时，我们将在同一个`application.properties` 文件中添加特定于特性的配置。

### 2.4.单元测试

我们的 web 应用程序将在现实场景中调用外部服务。然而，我们可以通过使用`WireMockExtension`类启动一个外部服务来**模拟这样一个正在运行的服务的存在。**

因此，让我们将`EXTERNAL_SERVICE`定义为`ResilientAppControllerUnitTest`类中的静态成员:

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class ResilientAppControllerUnitTest {

    @RegisterExtension
    static WireMockExtension EXTERNAL_SERVICE = WireMockExtension.newInstance()
      .options(WireMockConfiguration.wireMockConfig()
      .port(9090))
      .build();
```

此外，让我们添加一个`TestRestTemplate`实例来调用 API:

```java
@Autowired
private TestRestTemplate restTemplate;
```

### 2.5.异常处理程序

Resilience4j 库将根据上下文中的容错模式抛出一个异常来保护服务资源。但是，这些异常应该转化为 HTTP 响应，并为客户端提供有意义的状态代码。

所以，让我们**定义`ApiExceptionHandler`类来保存不同异常**的处理程序:

```java
@ControllerAdvice
public class ApiExceptionHandler {
}
```

当我们探索不同的容错模式时，我们将在这个类中添加处理程序。

## 3.断路器

**[断路器模式](/web/20221128052426/https://www.baeldung.com/resilience4j#circuit-breaker)通过限制上游服务在部分或完全停机期间调用下游服务来保护下游服务**。

让我们从暴露`/api/circuit-breaker`端点并添加`@CircuitBreaker`注释开始:

```java
@GetMapping("/circuit-breaker")
@CircuitBreaker(name = "CircuitBreakerService")
public String circuitBreakerApi() {
    return externalAPICaller.callApi();
}
```

根据需要，我们还需要在`ExternalAPICaller`类中定义`callApi()`方法来调用外部端点`/api/external`:

```java
public String callApi() {
    return restTemplate.getForObject("/api/external", String.class);
}
```

接下来，让我们在`application.properties`文件中添加断路器的配置:

```java
resilience4j.circuitbreaker.instances.CircuitBreakerService.failure-rate-threshold=50
resilience4j.circuitbreaker.instances.CircuitBreakerService.minimum-number-of-calls=5
resilience4j.circuitbreaker.instances.CircuitBreakerService.automatic-transition-from-open-to-half-open-enabled=true
resilience4j.circuitbreaker.instances.CircuitBreakerService.wait-duration-in-open-state=5s
resilience4j.circuitbreaker.instances.CircuitBreakerService.permitted-number-of-calls-in-half-open-state=3
resilience4j.circuitbreaker.instances.CircuitBreakerService.sliding-window-size=10
resilience4j.circuitbreaker.instances.CircuitBreakerService.sliding-window-type=count_based
```

本质上，该配置将允许对处于[关闭状态](/web/20221128052426/https://www.baeldung.com/resilience4j#1-circuit-breakers-states-and-settings)的服务的 50%的失败调用，之后它将打开电路并开始拒绝带有`CallNotPermittedException`的请求。因此，在`ApiExceptionHandler`类中为这个异常添加一个处理程序是个好主意:

```java
@ExceptionHandler({CallNotPermittedException.class})
@ResponseStatus(HttpStatus.SERVICE_UNAVAILABLE)
public void handleCallNotPermittedException() {
}
```

最后，让我们通过使用`EXTERNAL_SERVICE:`模拟下游服务停机的场景来测试`/api/circuit-breaker` API 端点

```java
@Test
public void testCircuitBreaker() {
    EXTERNAL_SERVICE.stubFor(WireMock.get("/api/external")
      .willReturn(serverError()));

    IntStream.rangeClosed(1, 5)
      .forEach(i -> {
          ResponseEntity response = restTemplate.getForEntity("/api/circuit-breaker", String.class);
          assertThat(response.getStatusCode()).isEqualTo(HttpStatus.INTERNAL_SERVER_ERROR);
      });

    IntStream.rangeClosed(1, 5)
      .forEach(i -> {
          ResponseEntity response = restTemplate.getForEntity("/api/circuit-breaker", String.class);
          assertThat(response.getStatusCode()).isEqualTo(HttpStatus.SERVICE_UNAVAILABLE);
      });

    EXTERNAL_SERVICE.verify(5, getRequestedFor(urlEqualTo("/api/external")));
}
```

我们可以注意到，由于下游服务中断，前五次调用都失败了。之后，电路切换到开路状态。因此，随后的五次尝试都被拒绝，并带有`503` HTTP 状态代码，而没有真正调用底层 API。

## 4.重试

**[重试模式](/web/20221128052426/https://www.baeldung.com/resilience4j#retry)通过从瞬态问题中恢复来为系统提供弹性**。让我们从添加带有`@Retry`注释的`/api/retry` API 端点开始:

```java
@GetMapping("/retry")
@Retry(name = "retryApi", fallbackMethod = "fallbackAfterRetry")
public String retryApi() {
    return externalAPICaller.callApi();
}
```

此外，我们可以**可选地提供一个后备机制，当所有的重试尝试都失败时**。在这种情况下，我们提供了`fallbackAfterRetry `作为后备方法:

```java
public String fallbackAfterRetry(Exception ex) {
    return "all retries have exhausted";
}
```

接下来，让我们更新`application.properties`文件，添加控制重试行为的配置:

```java
resilience4j.retry.instances.retryApi.max-attempts=3
resilience4j.retry.instances.retryApi.wait-duration=1s
resilience4j.retry.metrics.legacy.enabled=true
resilience4j.retry.metrics.enabled=true
```

因此，我们计划最多重试 3 次，每次延迟`1s`。

最后，让我们测试一下`/api/retry` API 端点的重试行为:

```java
@Test
public void testRetry() {
    EXTERNAL_SERVICE.stubFor(WireMock.get("/api/external")
      .willReturn(ok()));
    ResponseEntity<String> response1 = restTemplate.getForEntity("/api/retry", String.class);
    EXTERNAL_SERVICE.verify(1, getRequestedFor(urlEqualTo("/api/external")));

    EXTERNAL_SERVICE.resetRequests();

    EXTERNAL_SERVICE.stubFor(WireMock.get("/api/external")
      .willReturn(serverError()));
    ResponseEntity<String> response2 = restTemplate.getForEntity("/api/retry", String.class);
    Assert.assertEquals(response2.getBody(), "all retries have exhausted");
    EXTERNAL_SERVICE.verify(3, getRequestedFor(urlEqualTo("/api/external")));
}
```

我们可以注意到，在第一个场景中，没有问题，所以一次尝试就足够了。另一方面，当出现问题时，有三次尝试，之后 API 通过回退机制做出响应。

## 5.时间限制器

我们可以**使用[时间限制器模式](/web/20221128052426/https://www.baeldung.com/resilience4j#time-limiter)来设置对外部系统**进行异步调用的超时阈值。

让我们添加内部调用慢速 API 的`/api/time-limiter` API 端点:

```java
@GetMapping("/time-limiter")
@TimeLimiter(name = "timeLimiterApi")
public CompletableFuture<String> timeLimiterApi() {
    return CompletableFuture.supplyAsync(externalAPICaller::callApiWithDelay);
}
```

此外，让我们通过在`callApiWithDelay()`方法中添加睡眠时间来模拟外部 API 调用中的延迟:

```java
public String callApiWithDelay() {
    String result = restTemplate.getForObject("/api/external", String.class);
    try {
        Thread.sleep(5000);
    } catch (InterruptedException ignore) {
    }
    return result;
}
```

接下来，我们需要为`application.properties`文件中的`timeLimiterApi` 提供配置:

```java
resilience4j.timelimiter.metrics.enabled=true
resilience4j.timelimiter.instances.timeLimiterApi.timeout-duration=2s
resilience4j.timelimiter.instances.timeLimiterApi.cancel-running-future=true
```

我们可以注意到阈值被设置为 2s。之后，Resilience4j 库用一个`TimeoutException`在内部取消异步操作。因此，让我们**在`ApiExceptionHandler`类中为这个异常添加一个处理程序，以返回一个带有`408` HTTP 状态代码**的 API 响应:

```java
@ExceptionHandler({TimeoutException.class})
@ResponseStatus(HttpStatus.REQUEST_TIMEOUT)
public void handleTimeoutException() {
}
```

最后，让我们验证为`/api/time-limiter` API 端点配置的时间限制模式:

```java
@Test
public void testTimeLimiter() {
    EXTERNAL_SERVICE.stubFor(WireMock.get("/api/external").willReturn(ok()));
    ResponseEntity<String> response = restTemplate.getForEntity("/api/time-limiter", String.class);

    assertThat(response.getStatusCode()).isEqualTo(HttpStatus.REQUEST_TIMEOUT);
    EXTERNAL_SERVICE.verify(1, getRequestedFor(urlEqualTo("/api/external")));
}
```

正如预期的那样，由于下游 API 调用被设置为花费 5 秒多的时间来完成，我们目睹了 API 调用的超时。

## 6.防水壁

**[隔板模式](/web/20221128052426/https://www.baeldung.com/resilience4j#bulkhead)限制了对外部服务的最大并发调用数。**

让我们从添加带有`@Bulkhead`注释的`/api/bulkhead` API 端点开始:

```java
@GetMapping("/bulkhead")
@Bulkhead(name="bulkheadApi")
public String bulkheadApi() {
    return externalAPICaller.callApi();
}
```

接下来，让我们在`application.properties`文件中定义配置来控制隔板功能:

```java
resilience4j.bulkhead.metrics.enabled=true
resilience4j.bulkhead.instances.bulkheadApi.max-concurrent-calls=3
resilience4j.bulkhead.instances.bulkheadApi.max-wait-duration=1
```

这样，我们希望将并发调用的最大数量限制为`3`，这样，如果隔板已满，每个线程只能等待`1ms`。此后，请求被拒绝，并出现`BulkheadFullException`例外。此外，我们希望向客户机返回一个有意义的 HTTP 状态代码，所以让我们添加一个异常处理程序:

```java
@ExceptionHandler({ BulkheadFullException.class })
@ResponseStatus(HttpStatus.BANDWIDTH_LIMIT_EXCEEDED)
public void handleBulkheadFullException() {
}
```

最后，让我们通过并行调用五个请求来测试隔板行为:

```java
@Test
public void testBulkhead() throws InterruptedException {
    EXTERNAL_SERVICE.stubFor(WireMock.get("/api/external")
      .willReturn(ok()));
    Map<Integer, Integer> responseStatusCount = new ConcurrentHashMap<>();

    IntStream.rangeClosed(1, 5)
      .parallel()
      .forEach(i -> {
          ResponseEntity<String> response = restTemplate.getForEntity("/api/bulkhead", String.class);
          int statusCode = response.getStatusCodeValue();
          responseStatusCount.put(statusCode, responseStatusCount.getOrDefault(statusCode, 0) + 1);
      });

    assertEquals(2, responseStatusCount.keySet().size());
    assertTrue(responseStatusCount.containsKey(BANDWIDTH_LIMIT_EXCEEDED.value()));
    assertTrue(responseStatusCount.containsKey(OK.value()));
    EXTERNAL_SERVICE.verify(3, getRequestedFor(urlEqualTo("/api/external")));
}
```

我们注意到**只有三个请求是成功的，而其他请求都被拒绝，并显示了`BANDWIDTH_LIMIT_EXCEEDED ` HTTP 状态代码**。

## 7.限速器

**[速率限制器模式](/web/20221128052426/https://www.baeldung.com/resilience4j#rate-limiter)限制对资源的请求速率。**

让我们从添加带有`@RateLimiter`注释的`/api/rate-limiter` API 端点开始:

```java
@GetMapping("/rate-limiter")
@RateLimiter(name = "rateLimiterApi")
public String rateLimitApi() {
    return externalAPICaller.callApi();
}
```

接下来，让我们在`application.properties`文件中定义速率限制器的配置:

```java
resilience4j.ratelimiter.metrics.enabled=true
resilience4j.ratelimiter.instances.rateLimiterApi.register-health-indicator=true
resilience4j.ratelimiter.instances.rateLimiterApi.limit-for-period=5
resilience4j.ratelimiter.instances.rateLimiterApi.limit-refresh-period=60s
resilience4j.ratelimiter.instances.rateLimiterApi.timeout-duration=0s
resilience4j.ratelimiter.instances.rateLimiterApi.allow-health-indicator-to-fail=true
resilience4j.ratelimiter.instances.rateLimiterApi.subscribe-for-events=true
resilience4j.ratelimiter.instances.rateLimiterApi.event-consumer-buffer-size=50
```

有了这个配置，我们想把 API 调用速率限制在`5` `req/min`而不等待。**达到允许速率的阈值后，请求将被拒绝，例外情况为`RequestNotPermitted`**。因此，让我们在`ApiExceptionHandler`类中定义一个处理程序，将其转换成有意义的 HTTP 状态响应代码:

```java
@ExceptionHandler({ RequestNotPermitted.class })
@ResponseStatus(HttpStatus.TOO_MANY_REQUESTS)
public void handleRequestNotPermitted() {
}
```

最后，让我们用`50`请求来测试我们的速率受限的 API 端点:

```java
@Test
public void testRatelimiter() {
    EXTERNAL_SERVICE.stubFor(WireMock.get("/api/external")
      .willReturn(ok()));
    Map<Integer, Integer> responseStatusCount = new ConcurrentHashMap<>();

    IntStream.rangeClosed(1, 50)
      .parallel()
      .forEach(i -> {
          ResponseEntity<String> response = restTemplate.getForEntity("/api/rate-limiter", String.class);
          int statusCode = response.getStatusCodeValue();
          responseStatusCount.put(statusCode, responseStatusCount.getOrDefault(statusCode, 0) + 1);
      });

    assertEquals(2, responseStatusCount.keySet().size());
    assertTrue(responseStatusCount.containsKey(TOO_MANY_REQUESTS.value()));
    assertTrue(responseStatusCount.containsKey(OK.value()));
    EXTERNAL_SERVICE.verify(5, getRequestedFor(urlEqualTo("/api/external")));
}
```

不出所料，**只有五个请求成功，而所有其他请求都失败了，HTTP 状态代码为**。

## 8.执行器端点

我们已经将应用程序配置为支持用于监控目的的执行器端点。使用这些端点，我们可以使用一个或多个已配置的容错模式来确定应用程序随时间的行为。

首先，我们通常可以使用对`/actuator`端点的 GET 请求来**找到所有公开的端点:**

```java
http://localhost:8080/actuator/
{
    "_links" : {
        "self" : {...},
        "bulkheads" : {...},
        "circuitbreakers" : {...},
        "ratelimiters" : {...},
        ...
    }
}
```

我们可以看到一个 JSON 响应，带有类似于`bulkheads`、*断路器*、*速率限制器*等字段。每个字段根据其与容错模式的关联为我们提供特定的信息。

接下来，让我们看看与重试模式相关的字段:

```java
"retries": {
  "href": "http://localhost:8080/actuator/retries",
  "templated": false
},
"retryevents": {
  "href": "http://localhost:8080/actuator/retryevents",
  "templated": false
},
"retryevents-name": {
  "href": "http://localhost:8080/actuator/retryevents/{name}",
  "templated": true
},
"retryevents-name-eventType": {
  "href": "http://localhost:8080/actuator/retryevents/{name}/{eventType}",
  "templated": true
}
```

接下来，让我们检查应用程序以查看重试实例列表:

```java
http://localhost:8080/actuator/retries
{
    "retries" : [ "retryApi" ]
}
```

正如所料，我们可以在配置的重试实例列表中看到`retryApi`实例。

最后，让我们通过浏览器向`/api/retry` API 端点发出 GET 请求，然后**使用`/actuator/retryevents`端点**观察重试事件:

```java
{
    "retryEvents": [
    {
        "retryName": "retryApi",
        "type": "RETRY",
        "creationTime": "2022-10-16T10:46:31.950822+05:30[Asia/Kolkata]",
        "errorMessage": "...",
        "numberOfAttempts": 1
    },
    {
        "retryName": "retryApi",
        "type": "RETRY",
        "creationTime": "2022-10-16T10:46:32.965661+05:30[Asia/Kolkata]",
        "errorMessage": "...",
        "numberOfAttempts": 2
    },
    {
        "retryName": "retryApi",
        "type": "ERROR",
        "creationTime": "2022-10-16T10:46:33.978801+05:30[Asia/Kolkata]",
        "errorMessage": "...",
        "numberOfAttempts": 3
    }
  ]
}
```

由于下游服务关闭，我们可以看到三次重试尝试，任意两次尝试之间的等待时间为`1s`。就像我们配置的一样。

## 9.结论

在本文中，我们学习了如何在 Sprint Boot 应用程序中使用 Resilience4j 库。此外，**我们深入研究了几种容错模式，如断路器、速率限制器、时间限制器、隔板和重试。**

和往常一样，该教程的完整源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221128052426/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-libraries-2)