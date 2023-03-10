# 使用 WebClient 限制每秒请求数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-webclient-limit-requests-per-second>

## 1.介绍

在本教程中，我们将看到使用 [Spring 5 WebClient](/web/20221106142100/https://www.baeldung.com/spring-5-webclient) 限制每秒请求数量的不同方法。

虽然我们通常想利用它的非阻塞特性，但有些场景可能会迫使我们增加延迟。我们将学习其中一些场景，同时使用一些 [Project Reactor](/web/20221106142100/https://www.baeldung.com/reactor-core) 特性来控制对服务器的请求流。

## 2.初始设置

我们需要限制每秒请求数的一个典型例子是避免服务器不堪重负。此外，一些 [web 服务](/web/20221106142100/https://www.baeldung.com/building-a-restful-web-service-with-spring-and-java-based-configuration)有每小时允许的最大请求数。同样，有些控制每个客户端并发请求的数量。

### 2.1.编写简单的 Web 服务

为了探索这个场景，我们将从一个简单的 [`@RestController`](/web/20221106142100/https://www.baeldung.com/spring-controller-vs-restcontroller) 开始，它提供固定范围内的随机数:

```java
@RestController
@RequestMapping("/random")
public class RandomController {

    @GetMapping
    Integer getRandom() {
        return new Random().nextInt(50));
    }
} 
```

接下来，我们将模拟一个昂贵的操作，并限制并发请求的数量。

### 2.2.限制我们服务器的速率

在看到解决方案之前，让我们更改我们的服务来模拟一个更真实的场景。

首先，我们将限制服务器可以接受的并发请求的数量，当达到限制时抛出一个异常。

其次，我们将添加一个延迟来处理我们的响应，模拟一个昂贵的操作。虽然还有[更强大的解决方案](/web/20221106142100/https://www.baeldung.com/spring-bucket4j#:~:text=Free%3A%2020%20requests%20per%20hour,per%20hour%20per%20API%20client)可用，但我们这样做只是为了举例说明:

```java
public class Concurrency {

    public static final int MAX_CONCURRENT = 5;
    static final AtomicInteger CONCURRENT_REQUESTS = new HashMap<>();

    public static int protect(IntSupplier supplier) {
        try {
            if (CONCURRENT_REQUESTS.incrementAndGet() > MAX_CONCURRENT) {
                throw new UnsupportedOperationException("max concurrent requests reached");
            }

            TimeUnit.SECONDS.sleep(2);
            return supplier.getAsInt();
        } finally {
            CONCURRENT_REQUESTS.decrementAndGet();
        }
    }
}
```

最后，让我们更改端点来使用它:

```java
@GetMapping
Integer getRandom() {
    return Concurrency.protect(() -> new Random().nextInt(50));
}
```

**现在，当我们超过`MAX_CONCURRENT`个请求时，我们的端点拒绝处理请求，向客户端返回一个错误。**

### 2.3.编写简单的客户端

所有示例都将遵循这种模式来生成一个`n`请求的 [`Flux`](/web/20221106142100/https://www.baeldung.com/spring-webflux) ，并向我们的服务发出一个`GET`请求:

```java
Flux.range(1, n)
  .flatMap(i -> {
    // GET request
  });
```

为了减少样板文件，让我们用一种可以在所有示例中重用的方法来实现请求部分。**我们会收到一个`WebClient`，调用`get(),`和`retrieve()`响应体用[泛型](/web/20221106142100/https://www.baeldung.com/java-generics)使用 [`ParameterizedTypeReference`](https://web.archive.org/web/20221106142100/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/ParameterizedTypeReference.html) :**

```java
public interface RandomConsumer {

    static <T> Mono<T> get(WebClient client) {
        return client.get()
          .retrieve()
          .bodyToMono(new ParameterizedTypeReference<T>() {});
    }
} 
```

现在我们可以看到一些方法了。

## 3.用`zipWith(Flux.Interval())`延迟

我们的第一个例子使用 [`zipWith()`](/web/20221106142100/https://www.baeldung.com/reactor-combine-streams) 将我们的请求与固定延迟结合起来:

```java
public class ZipWithInterval {

    public static Flux<Integer> fetch(
      WebClient client, int requests, int delay) {
        return Flux.range(1, requests)
          .zipWith(Flux.interval(Duration.ofMillis(delay)))
          .flatMap(i -> RandomConsumer.get(client));
    }
} 
```

因此，这会将每个请求延迟`delay`毫秒。我们应该**注意，这个延迟在发送请求之前适用。**

## 4.用`Flux.delayElements()`延迟

`Flux`有一个更直接的方法来延迟它的元素:

```java
public class DelayElements {

    public static Flux<Integer> fetch(
      WebClient client, int requests, int delay) {
        return Flux.range(1, requests)
          .delayElements(Duration.ofMillis(delay))
          .flatMap(i -> RandomConsumer.get(client));
    }
} 
```

**使用`delayElements()`，延迟直接应用于`Subscriber.onNext()`信号。**换句话说，它延迟了来自`Flux.range()`的每个元素。所以传入`flatMap()`的函数会受到影响，启动时间更长。例如，如果`delay`的值是`1000`，在我们的请求开始之前会有一秒钟的延迟。

### 4.1.调整我们的解决方案

因此，如果我们不提供足够长的延迟，我们将得到一个错误:

```java
@Test
void givenSmallDelay_whenDelayElements_thenExceptionThrown() {
    int delay = 100;

    int requests = 10;
    assertThrows(InternalServerError.class, () -> {
      DelayElements.fetch(client, requests, delay)
        .blockLast();
    });
}
```

这是因为我们为每个请求等待了`100`毫秒，但是每个请求在服务器端需要两秒钟才能完成。因此，很快，我们的并发请求达到了极限，我们得到了一个`500`错误。

如果我们增加足够的延迟，我们可以摆脱请求限制。但是，我们会有另一个问题——我们会等待不必要的时间。

根据我们的用例，等待太久可能会显著影响性能。因此，接下来，让我们检查一个更合适的方法来处理这个问题，因为我们知道我们服务器的局限性。

## 5.用`flatMap()`进行并发控制

鉴于我们服务的局限性，我们最好的选择是并行发送最多`Concurrency.MAX_CONCURRENT`个请求。**要做到这一点，我们可以在`flatMap()`中增加一个参数来表示并行处理的最大数量:**

```java
public class LimitConcurrency {

    public static Flux<Integer> fetch(
      WebClient client, int requests, int concurrency) {
        return Flux.range(1, requests)
          .flatMap(i -> RandomConsumer.get(client), concurrency);
    }
} 
```

**该参数保证并发请求的最大数量不超过`concurrency`，并且我们的处理不会被不必要的延迟:**

```java
@Test
void givenLimitInsideServerRange_whenLimitedConcurrency_thenNoExceptionThrown() {
    int limit = Concurrency.MAX_CONCURRENT;

    int requests = 10;
    assertDoesNotThrow(() -> {
      LimitConcurrency.fetch(client, TOTAL_REQUESTS, limit)
        .blockLast();
    });
}
```

不过，根据我们的场景和偏好，还有一些其他选项值得讨论。让我们回顾一下其中的一些。

## 6.使用 Resilience4j `RateLimiter`

Resilience4j 是一个多功能的库，用于处理应用程序中的容错。**我们将使用它来限制一个时间间隔内并发请求的数量，并包括一个超时。**

让我们从添加 [resilience4j-reactor](https://web.archive.org/web/20221106142100/https://search.maven.org/search?q=g:io.github.resilience4j%20a:resilience4j-reactor) 和 [resilience4j-ratelimiter](https://web.archive.org/web/20221106142100/https://search.maven.org/search?q=g:io.github.resilience4j%20a:resilience4j-ratelimiter) 依赖关系开始:

```java
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-reactor</artifactId>
    <version>1.7.1</version>
</dependency>
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-ratelimiter</artifactId>
    <version>1.7.1</version>
</dependency>
```

然后我们用`RateLimiter.of()`构建我们的速率限制器，提供名称、发送新请求的时间间隔、并发限制和超时:

```java
public class Resilience4jRateLimit {

    public static Flux<Integer> fetch(
      WebClient client, int requests, int concurrency, int interval) {
        RateLimiter limiter = RateLimiter.of("my-rate-limiter", RateLimiterConfig.custom()
          .limitRefreshPeriod(Duration.ofMillis(interval))
          .limitForPeriod(concurrency)
          .timeoutDuration(Duration.ofMillis(interval * concurrency))
          .build());

        // ...
    }
}
```

**现在我们将它包含在我们的`Flux`和`transformDeferred(),` 中，因此它控制我们的`GET`请求率:**

```java
return Flux.range(1, requests)
  .flatMap(i -> RandomConsumer.get(client)
    .transformDeferred(RateLimiterOperator.of(limiter))
  );
```

我们应该注意到，如果我们将间隔定义得太低，仍然会有问题。但是，如果我们需要与其他操作共享速率限制器规范，这种方法是有帮助的。

## 7.番石榴精确节流

[番石榴](/web/20221106142100/https://www.baeldung.com/guava-guide)有一个通用的[限速器](/web/20221106142100/https://www.baeldung.com/guava-rate-limiter)，它很适合我们的场景。**此外，由于它使用令牌桶算法，它将只在必要时阻塞，而不是每次都阻塞，不像`Flux.delayElements()`。**

首先，我们需要将[番石榴](https://web.archive.org/web/20221106142100/https://search.maven.org/search?q=g:com.google.guava%20a:guava)添加到 pom.xml 中:

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.1-jre</version>
</dependency>
```

要使用它，我们调用`RateLimiter.create()`并向它传递我们希望每秒发送的最大请求数。然后，我们调用`limiter`上的`acquire()`，然后在必要时发送我们的请求来限制执行:

```java
public class GuavaRateLimit {

    public static Flux<Integer> fetch(
      WebClient client, int requests, int requestsPerSecond) {
        RateLimiter limiter = RateLimiter.create(requestsPerSecond);

        return Flux.range(1, requests)
          .flatMap(i -> {
            limiter.acquire();

            return RandomConsumer.get(client);
          });
    }
} 
```

这个解决方案非常简单——它不会让我们的代码块变得过长。例如，如果由于某种原因，一个请求花费的时间比预期的长，下一个请求就不会等待执行。但是，只有当我们在为`requestsPerSecond`设定的范围内时，才会出现这种情况。

## 8.结论

在本文中，我们看到了一些限制我们的`WebClient`速率的方法。之后，我们模拟了一个受控的 web 服务，看看它如何影响我们的代码和测试。此外，我们使用 Project Reactor 和一些库来帮助我们以不同的方式实现相同的目标。

和往常一样，源代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20221106142100/https://github.com/eugenp/tutorials/tree/master//spring-reactive-modules/spring-5-reactive-client-2)