# 如何用 Java 提取单声道的内容

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-string-from-mono>

## 1.概观

在我们的[对 Project Reactor、](/web/20220723143545/https://www.baeldung.com/reactor-core)的介绍中，我们了解了`Mono<T>,` ，它是类型`T`的一个实例的发布者。

在这个快速教程中，我们将展示从`Mono` : `block`和`subscribe`中提取`T `的阻塞和非阻塞方式。

## 2. ****堵路****

 **一般来说，`Mono`通过在某个时间点发射一个元素来成功完成。

让我们从一个示例发布者`Mono<String>`开始:

```java
Mono<String> blockingHelloWorld() {
    return Mono.just("Hello world!");
}

String result = blockingHelloWorld().block();
assertEquals("Hello world!", result);
```

这里，只要发布者不发出值，我们就阻塞执行。但是，它可能需要很长时间才能完成。

为了获得更多的控制，让我们设置一个明确的持续时间:

```java
String result = blockingHelloWorld().block(Duration.of(1000, ChronoUnit.MILLIS));
assertEquals(expected, result);
```

如果发布者没有在设定的持续时间内发出一个值，则抛出一个 `RuntimeException` 。

另外，`Mono`可以是空的，上面的`block()`方法将返回`null`。相反，我们可以在这种情况下使用`block` `Optional`:

```java
Optional<String> result = Mono.<String>empty().blockOptional();
assertEquals(Optional.empty(), result);
```

一般来说，阻塞违背了反应式编程的原则。非常不鼓励在反应式应用程序中阻塞执行。

那么，现在让我们看看如何以非阻塞的方式获取值。

## 3. **无阻塞** **方式**

首先要使用`subscribe()`方法以非阻塞的方式订阅。此外，我们将指定最终值的使用者:

```java
blockingHelloWorld()
  .subscribe(result -> assertEquals(expected, result));
```

在这里，**即使需要一些时间来产生值，执行也会立即继续，而不会阻塞对`subscribe()` 调用**。

在某些情况下，我们希望在中间步骤中消费价值。因此，我们可以使用一个操作符来添加行为:

```java
blockingHelloWorld()
  .doOnNext(result -> assertEquals(expected, result))
  .subscribe();
```

## 4.结论

在这篇短文中，我们探讨了消费由`Mono<String>`产生的值的两种方式。

和往常一样，代码示例可以在 GitHub 上找到[。](https://web.archive.org/web/20220723143545/https://github.com/eugenp/tutorials/tree/master/reactor-core)**