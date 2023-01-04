# 番石榴限速器快速指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/guava-rate-limiter>

## 1。概述

在本文中，我们将关注来自`Guava` 库的`[RateLimiter](https://web.archive.org/web/20220626080705/https://google.github.io/guava/releases/22.0/api/docs/index.html?com/google/common/util/concurrent/RateLimiter.html)` 类。

`RateLimiter`类是一个构造，它允许我们调节某些处理发生的速率。如果我们创建一个有 N 个许可的`RateLimiter` ——这意味着进程每秒最多可以发出 N 个许可。

## 2。Maven 依赖关系

我们将使用番石榴的图书馆:

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

最新版本可以在这里找到[。](https://web.archive.org/web/20220626080705/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.google.guava%22%20AND%20a%3A%22guava%22)

## 3。创建和使用`RateLimiter`

假设我们希望**将`doSomeLimitedOperation()`的执行速率限制为每秒 2 次。**

我们可以使用`create()` 工厂方法创建一个`RateLimiter` 实例:

```java
RateLimiter rateLimiter = RateLimiter.create(2);
```

接下来，为了从`RateLimiter,`获得执行许可，我们需要调用`acquire()`方法:

```java
rateLimiter.acquire(1);
```

为了检查这是否有效，我们将对 throttled 方法进行两次后续调用:

```java
long startTime = ZonedDateTime.now().getSecond();
rateLimiter.acquire(1);
doSomeLimitedOperation();
rateLimiter.acquire(1);
doSomeLimitedOperation();
long elapsedTimeSeconds = ZonedDateTime.now().getSecond() - startTime;
```

为了简化我们的测试，让我们假设`doSomeLimitedOperation()` 方法立即完成。

在这种情况下，对`acquire()` 方法的两次调用都不应该阻塞，经过的时间应该少于或低于一秒——因为两个许可都可以立即获得:

```java
assertThat(elapsedTimeSeconds <= 1);
```

此外，我们可以通过一次`acquire()`调用获得所有许可:

```java
@Test
public void givenLimitedResource_whenRequestOnce_thenShouldPermitWithoutBlocking() {
    // given
    RateLimiter rateLimiter = RateLimiter.create(100);

    // when
    long startTime = ZonedDateTime.now().getSecond();
    rateLimiter.acquire(100);
    doSomeLimitedOperation();
    long elapsedTimeSeconds = ZonedDateTime.now().getSecond() - startTime;

    // then
    assertThat(elapsedTimeSeconds <= 1);
}
```

例如，如果我们需要每秒发送 100 个字节，这就很有用。我们可以发送一百次一个字节，一次获取一个许可。另一方面，我们可以一次发送全部 100 个字节，在一次操作中获得全部 100 个许可。

## 4。以封锁方式获取许可

现在，让我们考虑一个稍微复杂一点的例子。

我们将创建一个有 100 个许可证的`RateLimiter` 。然后，我们将执行一个需要获取 1000 个许可的操作。根据`RateLimiter,` 的规格，这样的动作至少需要 10 秒才能完成，因为我们每秒只能执行 100 个动作单位:

```java
@Test
public void givenLimitedResource_whenUseRateLimiter_thenShouldLimitPermits() {
    // given
    RateLimiter rateLimiter = RateLimiter.create(100);

    // when
    long startTime = ZonedDateTime.now().getSecond();
    IntStream.range(0, 1000).forEach(i -> {
        rateLimiter.acquire();
        doSomeLimitedOperation();
    });
    long elapsedTimeSeconds = ZonedDateTime.now().getSecond() - startTime;

    // then
    assertThat(elapsedTimeSeconds >= 10);
}
```

请注意，我们在这里是如何使用`acquire()` 方法的——这是一种阻塞方法，我们在使用时应该小心。当调用`acquire()`方法时，它会阻塞正在执行的线程，直到获得许可。

**不带参数调用`acquire()` 与带参数 1 调用**是一样的——它将试图获取一个许可。

## 5。超时获取许可

`RateLimiter` API 还有一个非常有用的`acquire()`方法，**接受一个`timeout`和`TimeUnit` 作为参数。**

如果在`timeout.`内没有足够的可用许可，当没有可用许可时调用该方法将导致它等待指定的时间，然后超时

当在给定的超时时间内没有可用的许可时，它返回`false.`。如果`acquire()`成功，它`returns`为真:

```java
@Test
public void givenLimitedResource_whenTryAcquire_shouldNotBlockIndefinitely() {
    // given
    RateLimiter rateLimiter = RateLimiter.create(1);

    // when
    rateLimiter.acquire();
    boolean result = rateLimiter.tryAcquire(2, 10, TimeUnit.MILLISECONDS);

    // then
    assertThat(result).isFalse();
}
```

我们创建了一个有一个许可的`RateLimiter`,所以试图获取两个许可总是会导致`tryAcquire()`返回`false.`

## 6。结论

在这个快速教程中，我们看了一下来自`Guava`库的`RateLimiter` 构造。

我们学习了如何使用`RateLimtiter` 来限制每秒的许可数量。我们看到了如何使用它的阻塞 API，还使用了显式超时来获取许可。

与往常一样，所有这些示例和代码片段的实现都可以在 [GitHub 项目](https://web.archive.org/web/20220626080705/https://github.com/eugenp/tutorials/tree/master/guava-modules/guava-utilities)中找到——这是一个 Maven 项目，因此它应该很容易导入和运行。