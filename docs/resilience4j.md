# Resilience4j 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/resilience4j>

## 1。概述

在本教程中，我们将讨论 [Resilience4j](https://web.archive.org/web/20220706132740/https://resilience4j.github.io/resilience4j/) 库。

该库通过管理远程通信的容错来帮助实现弹性系统。

这个库的灵感来自于 [Hystrix](/web/20220706132740/https://www.baeldung.com/introduction-to-hystrix) ，但是它提供了一个更加方便的 API 和许多其他特性，比如速率限制器(阻止过于频繁的请求)、隔板(避免过多的并发请求)等等。

## 2。Maven 设置

首先，我们需要将目标模块添加到我们的`pom.xml` (例如，这里我们添加了断路器)`:`

```java
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-circuitbreaker</artifactId>
    <version>0.12.1</version>
</dependency>
```

这里，我们使用的是`circuitbreaker `模块。所有模块及其最新版本都可以在 [Maven Central](https://web.archive.org/web/20220706132740/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22io.github.resilience4j%22) 上找到。

在接下来的部分中，我们将浏览该库中最常用的模块。

## 3。断路器

注意，对于这个模块，我们需要上面显示的`resilience4j-circuitbreaker`依赖关系。

[断路器模式](https://web.archive.org/web/20220706132740/https://martinfowler.com/bliki/CircuitBreaker.html)有助于我们在远程服务中断时防止一连串的故障。

在多次尝试失败后，我们可以认为服务不可用/过载，并急切地拒绝对它的所有后续请求。这样，我们可以为可能失败的呼叫节省系统资源。

让我们看看如何用 Resilience4j 实现这一点。

首先，我们需要定义要使用的设置。最简单的方法是使用默认设置:

```java
CircuitBreakerRegistry circuitBreakerRegistry
  = CircuitBreakerRegistry.ofDefaults();
```

也可以使用自定义参数:

```java
CircuitBreakerConfig config = CircuitBreakerConfig.custom()
  .failureRateThreshold(20)
  .ringBufferSizeInClosedState(5)
  .build();
```

这里，我们将速率阈值设置为 20%,最少尝试 5 次呼叫。

然后，我们创建一个`CircuitBreaker`对象，并通过它调用远程服务:

```java
interface RemoteService {
    int process(int i);
}

CircuitBreakerRegistry registry = CircuitBreakerRegistry.of(config);
CircuitBreaker circuitBreaker = registry.circuitBreaker("my");
Function<Integer, Integer> decorated = CircuitBreaker
  .decorateFunction(circuitBreaker, service::process);
```

最后，让我们通过 JUnit 测试来看看这是如何工作的。

我们将尝试呼叫服务 10 次。我们应该能够验证呼叫至少尝试了 5 次，然后在 20%的呼叫失败后立即停止:

```java
when(service.process(any(Integer.class))).thenThrow(new RuntimeException());

for (int i = 0; i < 10; i++) {
    try {
        decorated.apply(i);
    } catch (Exception ignore) {}
}

verify(service, times(5)).process(any(Integer.class));
```

### 3.1。断路器的**状态和设置**

`CircuitBreaker`可以处于三种状态之一:

*   一切正常，没有短路
*   `OPEN`–远程服务器关闭，对它的所有请求都被短路
*   `HALF_OPEN`–自进入打开状态后，已经过了配置的时间量，并且`CircuitBreaker`允许请求检查远程服务是否恢复在线

我们可以配置以下设置:

*   失败率阈值，高于该阈值时`CircuitBreaker`会打开并开始短路呼叫
*   等待时间，定义了`CircuitBreaker`在切换到半开之前应该保持打开多长时间
*   `CircuitBreaker`半开或关闭时环形缓冲器的大小
*   处理`CircuitBreaker`事件的自定义`CircuitBreakerEventListener`
*   自定义`Predicate`评估异常是否应该算作失败，从而增加失败率

## 4。限速器

与上一节类似，此功能需要 [`resilience4j-ratelimiter`](https://web.archive.org/web/20220706132740/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22resilience4j-ratelimiter%22) 的依赖。

顾名思义，**该功能允许限制对某些服务的访问**。它的 API 和`CircuitBreaker's`很像——有`Registry`、`Config`和`Limiter`类。

下面是一个例子:

```java
RateLimiterConfig config = RateLimiterConfig.custom().limitForPeriod(2).build();
RateLimiterRegistry registry = RateLimiterRegistry.of(config);
RateLimiter rateLimiter = registry.rateLimiter("my");
Function<Integer, Integer> decorated
  = RateLimiter.decorateFunction(rateLimiter, service::process);
```

现在，修饰服务块上的所有调用都必须符合速率限制器配置。

我们可以配置如下参数:

*   限额刷新的周期
*   刷新周期的权限限制
*   默认的等待权限持续时间

## 5。舱壁

在这里，我们首先需要 [`resilience4j-bulkhead`](https://web.archive.org/web/20220706132740/https://search.maven.org/classic/#search%7Cga%7C1%7Cresilience4j-bulkhead) 的依赖关系。

可以限制某个特定服务的并发调用数量。

让我们看一个使用隔板 API 来配置一个并发调用的最大数量的例子:

```java
BulkheadConfig config = BulkheadConfig.custom().maxConcurrentCalls(1).build();
BulkheadRegistry registry = BulkheadRegistry.of(config);
Bulkhead bulkhead = registry.bulkhead("my");
Function<Integer, Integer> decorated
  = Bulkhead.decorateFunction(bulkhead, service::process);
```

为了测试这个配置，我们将调用一个模拟服务的方法。

然后，我们确保`Bulkhead`不允许任何其他调用:

```java
CountDownLatch latch = new CountDownLatch(1);
when(service.process(anyInt())).thenAnswer(invocation -> {
    latch.countDown();
    Thread.currentThread().join();
    return null;
});

ForkJoinTask<?> task = ForkJoinPool.commonPool().submit(() -> {
    try {
        decorated.apply(1);
    } finally {
        bulkhead.onComplete();
    }
});
latch.await();
assertThat(bulkhead.isCallPermitted()).isFalse();
```

我们可以配置以下设置:

*   隔板允许的最大并行执行量
*   尝试进入饱和隔板时，线程等待的最长时间

## 6。重试

对于这个特性，我们需要将 [`resilience4j-retry`](https://web.archive.org/web/20220706132740/https://search.maven.org/classic/#search%7Cga%7C1%7Cresilience4j-retry) 库添加到项目中。

我们可以使用重试 API 自动重试失败的调用:

```java
RetryConfig config = RetryConfig.custom().maxAttempts(2).build();
RetryRegistry registry = RetryRegistry.of(config);
Retry retry = registry.retry("my");
Function<Integer, Void> decorated
  = Retry.decorateFunction(retry, (Integer s) -> {
        service.process(s);
        return null;
    });
```

现在让我们模拟一种情况，在远程服务调用期间抛出异常，并确保库自动重试失败的调用:

```java
when(service.process(anyInt())).thenThrow(new RuntimeException());
try {
    decorated.apply(1);
    fail("Expected an exception to be thrown if all retries failed");
} catch (Exception e) {
    verify(service, times(2)).process(any(Integer.class));
}
```

我们还可以配置以下内容:

*   最大尝试次数
*   重试前的等待时间
*   修改失败后等待时间间隔的自定义功能
*   评估异常是否会导致重试调用的自定义`Predicate`

## 7。缓存

缓存模块需要 [`resilience4j-cache`](https://web.archive.org/web/20220706132740/https://search.maven.org/classic/#search%7Cga%7C1%7Cresilience4j-cache) 的依赖。

初始化看起来与其他模块略有不同:

```java
javax.cache.Cache cache = ...; // Use appropriate cache here
Cache<Integer, Integer> cacheContext = Cache.of(cache);
Function<Integer, Integer> decorated
  = Cache.decorateSupplier(cacheContext, () -> service.process(1));
```

这里缓存是由使用的 [JSR-107 缓存](/web/20220706132740/https://www.baeldung.com/jcache)实现完成的，而 Resilience4j 提供了一种应用它的方法。

注意没有修饰函数的 API(像`Cache.decorateFunction(Function)`)，API 只支持`Supplier`和`Callable` 类型。

## 8。时间限制器

对于这个模块，我们必须添加 [`resilience4j-timelimiter`](https://web.archive.org/web/20220706132740/https://search.maven.org/classic/#search%7Cga%7C1%7Cresilience4j-timelimiter) 依赖项。

使用 TimeLimiter 可以限制调用远程服务花费的时间。

为了演示，让我们设置一个配置了 1 毫秒超时的`TimeLimiter`:

```java
long ttl = 1;
TimeLimiterConfig config
  = TimeLimiterConfig.custom().timeoutDuration(Duration.ofMillis(ttl)).build();
TimeLimiter timeLimiter = TimeLimiter.of(config);
```

接下来，让我们验证 Resilience4j 调用`Future.get()`的预期超时:

```java
Future futureMock = mock(Future.class);
Callable restrictedCall
  = TimeLimiter.decorateFutureSupplier(timeLimiter, () -> futureMock);
restrictedCall.call();

verify(futureMock).get(ttl, TimeUnit.MILLISECONDS);
```

我们也可以把它和`CircuitBreaker`结合起来:

```java
Callable chainedCallable
  = CircuitBreaker.decorateCallable(circuitBreaker, restrictedCall);
```

## 9。附加模块

Resilience4j 还提供了许多附加模块，简化了与流行框架和库的集成。

一些比较著名的集成有:

*   Spring Boot—`resilience4j-spring-boot`模块
*   rat pack-`resilience4j-ratpack`模块
*   改装-`resilience4j-retrofit`模块
*   顶点–
*   drop wizard-`resilience4j-metrics`模块
*   普罗米修斯-`resilience4j-prometheus`模块

## 10。结论

在本文中，我们介绍了 Resilience4j 库的不同方面，并学习了如何使用它来解决服务器间通信中的各种容错问题。

和往常一样，以上示例的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220706132740/https://github.com/eugenp/tutorials/tree/master/libraries-6)