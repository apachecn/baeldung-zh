# Spring WebFlux 中的重试指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-webflux-retry>

## 1.概观

当我们在分布式云环境中构建应用程序时，我们需要为失败做好准备。这通常涉及重试。

Spring WebFlux 为我们提供了一些重试失败操作的工具。

在本教程中，我们将了解如何在 Spring WebFlux 应用程序中添加和配置重试。

## 2.用例

对于我们的例子，我们将使用 [`MockWebServer`](/web/20220523144947/https://www.baeldung.com/spring-mocking-webclient#mockwebserver) 并模拟一个外部系统暂时不可用，然后变得可用。

让我们为连接到这个 REST 服务的组件创建一个简单的测试:

```java
@Test
void givenExternalServiceReturnsError_whenGettingData_thenRetryAndReturnResponse() {

    mockExternalService.enqueue(new MockResponse()
      .setResponseCode(SERVICE_UNAVAILABLE.code()));
    mockExternalService.enqueue(new MockResponse()
      .setResponseCode(SERVICE_UNAVAILABLE.code()));
    mockExternalService.enqueue(new MockResponse()
      .setResponseCode(SERVICE_UNAVAILABLE.code()));
    mockExternalService.enqueue(new MockResponse()
      .setBody("stock data"));

    StepVerifier.create(externalConnector.getData("ABC"))
      .expectNextMatches(response -> response.equals("stock data"))
      .verifyComplete();

    verifyNumberOfGetRequests(4);
}
```

## 3.添加重试次数

在`Mono`和`Flux`API 中内置了两个关键的重试操作符。

### 3.1.使用`retry`

首先，让我们使用`retry`方法，它防止应用程序立即返回错误并重新订阅指定的次数:

```java
public Mono<String> getData(String stockId) {
    return webClient.get()
        .uri(PATH_BY_ID, stockId)
        .retrieve()
        .bodyToMono(String.class)
        .retry(3);
}
```

这将重试三次，无论从 web 客户端返回什么错误。

### 3.2.使用`retryWhen`

接下来，让我们使用`retryWhen`方法尝试一个可配置的策略:

```java
public Mono<String> getData(String stockId) {
    return webClient.get()
        .uri(PATH_BY_ID, stockId)
        .retrieve()
        .bodyToMono(String.class)
        .retryWhen(Retry.max(3));
}
```

这允许我们配置一个 [`Retry`](https://web.archive.org/web/20220523144947/https://projectreactor.io/docs/core/release/api/reactor/util/retry/Retry.html) 对象来描述期望的逻辑。

这里，我们使用了`max`策略来重试最大次数的尝试。这相当于我们的第一个例子，但允许我们更多的配置选项。特别是，我们应该注意，在这种情况下，**每次重试都尽可能快地发生**。

## 4.添加延迟

没有任何延迟地重试的主要缺点是，这没有给失败的服务时间来恢复。它可能会压倒 It，使问题变得更糟，减少恢复的机会。

### 4.1.用`fixedDelay`重试

我们可以使用`fixedDelay`策略在每次尝试之间添加延迟:

```java
public Mono<String> getData(String stockId) {
    return webClient.get()
      .uri(PATH_BY_ID, stockId)
      .retrieve()
      .bodyToMono(String.class)
      .retryWhen(Retry.fixedDelay(3, Duration.ofSeconds(2)));
}
```

这种配置允许两次尝试之间有两秒钟的延迟，这可能会增加成功的机会。但是，如果服务器经历了更长时间的中断，那么我们应该等待更长时间。但是，如果我们将所有延迟都配置为很长时间，那么短暂的信号会使我们的服务速度更慢。

### 4.2.用`backoff`重试

我们可以使用`backoff`策略，而不是以固定的时间间隔重试:

```java
public Mono<String> getData(String stockId) {
    return webClient.get()
      .uri(PATH_BY_ID, stockId)
      .retrieve()
      .bodyToMono(String.class)
      .retryWhen(Retry.backoff(3, Duration.ofSeconds(2)));
}
```

实际上，这个**在尝试**之间增加了一个逐渐增加的延迟——在我们的例子中大约是 2、4、然后 8 秒的间隔。这种**给了外部系统一个更好的机会从常见的连接问题中恢复**或者处理积压的工作。

### 4.3.用`jitter`重试

`backoff`策略的另一个好处是，它将随机性或[抖动](/web/20220523144947/https://www.baeldung.com/resilience4j-backoff-jitter#jitter)添加到计算的延迟间隔中。因此，**抖动有助于减少多个客户端同步重试的重试风暴**。

默认情况下，该值设置为 0.5，对应的抖动最多为计算延迟的 50%。

让我们使用`jitter`方法来配置一个不同的值 0.75，以表示最多为计算延迟的 75%的抖动:

```java
public Mono<String> getData(String stockId) {
    return webClient.get()
      .uri(PATH_BY_ID, stockId)
      .accept(MediaType.APPLICATION_JSON)
      .retrieve()
      .bodyToMono(String.class)
      .retryWhen(Retry.backoff(3, Duration.ofSeconds(2)).jitter(0.75));
}
```

我们应该注意，值的可能范围在 0(无抖动)和 1(抖动最多为计算延迟的 100%)之间。

## 5.过滤误差

此时，来自服务的任何错误都将导致重试尝试，包括 4xx 错误，如`400:Bad Request`或`401:Unauthorized`。

显然，我们不应该重试这种客户端错误，因为服务器响应不会有任何不同。因此，让我们看看如何**仅在特定错误**的情况下应用重试策略。

首先，让我们创建一个异常来表示服务器错误:

```java
public class ServiceException extends RuntimeException {

    public ServiceException(String message, int statusCode) {
        super(message);
        this.statusCode = statusCode;
    }
}
```

接下来，我们将为 5xx 错误创建一个错误`Mono`，并使用`filter`方法来配置我们的策略:

```java
public Mono<String> getData(String stockId) {
    return webClient.get()
      .uri(PATH_BY_ID, stockId)
      .retrieve()
      .onStatus(HttpStatus::is5xxServerError, 
          response -> Mono.error(new ServiceException("Server error", response.rawStatusCode())))
      .bodyToMono(String.class)
      .retryWhen(Retry.backoff(3, Duration.ofSeconds(5))
          .filter(throwable -> throwable instanceof ServiceException));
}
```

现在，我们只在`WebClient`管道中抛出`ServiceException`时重试。

## 6.处理用尽的重试

最后，我们可以考虑所有重试尝试都不成功的可能性。在这种情况下，策略的默认行为是传播一个`RetryExhaustedException`，包装上一个错误。

相反，让我们通过使用`onRetryExhaustedThrow`方法来覆盖这种行为，并为我们的`ServiceException`提供一个生成器:

```java
public Mono<String> getData(String stockId) {
    return webClient.get()
      .uri(PATH_BY_ID, stockId)
      .retrieve()
      .onStatus(HttpStatus::is5xxServerError, response -> Mono.error(new ServiceException("Server error", response.rawStatusCode())))
      .bodyToMono(String.class)
      .retryWhen(Retry.backoff(3, Duration.ofSeconds(5))
          .filter(throwable -> throwable instanceof ServiceException)
          .onRetryExhaustedThrow((retryBackoffSpec, retrySignal) -> {
              throw new ServiceException("External Service failed to process after max retries", HttpStatus.SERVICE_UNAVAILABLE.value());
          }));
}
```

现在，在一系列失败的重试之后，请求将会因我们的`ServiceException`而失败。

## 7.结论

在本文中，我们研究了如何使用`retry`和`retryWhen`方法在 Spring WebFlux 应用程序中添加重试。

最初，我们增加了失败操作的最大重试次数。然后，我们通过使用和配置各种策略引入了尝试之间的延迟。

最后，我们研究了重试某些错误，以及在所有尝试都失败后定制行为。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220523144947/https://github.com/eugenp/tutorials/tree/master/spring-5-webflux)