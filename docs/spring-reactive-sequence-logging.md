# 记录反应序列

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-reactive-sequence-logging>

## 1。概述

随着 Spring WebFlux 的引入，我们有了另一个强大的工具来编写反应式、非阻塞的应用程序。虽然现在使用这项技术比以前容易多了，但是**在 Spring WebFlux 中调试反应序列可能会非常麻烦**。

在这个快速教程中，我们将看到如何轻松地以异步序列记录事件，以及如何避免一些简单的错误。

## 2。Maven 依赖关系

让我们将 Spring WebFlux 依赖项添加到我们的项目中，这样我们就可以创建反应流:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency> 
```

我们可以从 Maven Central 获得最新的 [`spring-boot-starter-webflux`](https://web.archive.org/web/20221126220628/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-webflux%22) 依赖关系。

## 3。创建反应流

首先，让我们使用`Flux`创建一个反应流，并使用`log() `方法来启用日志记录:

```java
Flux<Integer> reactiveStream = Flux.range(1, 5).log(); 
```

接下来，我们将订阅它来消费生成的值:

```java
reactiveStream.subscribe(); 
```

## 4。记录反应流

运行上面的应用程序后，我们看到我们的日志记录器在运行:

```java
2018-11-11 22:37:04 INFO | onSubscribe([Synchronous Fuseable] FluxRange.RangeSubscription)
2018-11-11 22:37:04 INFO | request(unbounded)
2018-11-11 22:37:04 INFO | onNext(1)
2018-11-11 22:37:04 INFO | onNext(2)
2018-11-11 22:37:04 INFO | onNext(3)
2018-11-11 22:37:04 INFO | onNext(4)
2018-11-11 22:37:04 INFO | onNext(5)
2018-11-11 22:37:04 INFO | onComplete() 
```

我们可以看到发生在我们信息流上的每一个事件。发出五个值，然后用一个`onComplete()`事件关闭流。

## 5。高级日志场景

我们可以修改我们的应用程序，看到一个更有趣的场景。让我们将`take()`添加到`Flux`中，这将指示流只提供特定数量的事件:

```java
Flux<Integer> reactiveStream = Flux.range(1, 5).log().take(3); 
```

执行代码后，我们将看到以下输出:

```java
2018-11-11 22:45:35 INFO | onSubscribe([Synchronous Fuseable] FluxRange.RangeSubscription)
2018-11-11 22:45:35 INFO | request(unbounded)
2018-11-11 22:45:35 INFO | onNext(1)
2018-11-11 22:45:35 INFO | onNext(2)
2018-11-11 22:45:35 INFO | onNext(3)
2018-11-11 22:45:35 INFO | cancel() 
```

正如我们所见，`take()`在发出三个事件后导致流取消。

**在你的信息流中`log()`的位置至关重要**。让我们看看将`log()`放在`take()`之后会产生怎样不同的输出:

```java
Flux<Integer> reactiveStream = Flux.range(1, 5).take(3).log(); 
```

输出是:

```java
2018-11-11 22:49:23 INFO | onSubscribe([Fuseable] FluxTake.TakeFuseableSubscriber)
2018-11-11 22:49:23 INFO | request(unbounded)
2018-11-11 22:49:23 INFO | onNext(1)
2018-11-11 22:49:23 INFO | onNext(2)
2018-11-11 22:49:23 INFO | onNext(3)
2018-11-11 22:49:23 INFO | onComplete() 
```

正如我们所看到的，改变观察点会改变输出。现在这个流产生了三个事件，但是我们看到的不是`cancel(),`而是`onComplete()`。**这是因为我们观察到使用`take() `而不是该方法所要求的输出。**

## 6。结论

在这篇简短的文章中，我们看到了如何使用内置的`log()`方法记录反应流。

和往常一样，上面例子的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221126220628/https://github.com/eugenp/tutorials/tree/master/spring-reactive-modules/spring-5-reactive-3)