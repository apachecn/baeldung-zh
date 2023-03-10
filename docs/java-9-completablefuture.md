# Java 9 CompletableFuture 未来的 API 改进

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-9-completablefuture>

## 1。简介

Java 9 对`CompletableFuture`类做了一些改变。这些变化是作为 [JEP 266](https://web.archive.org/web/20220628093044/https://openjdk.java.net/jeps/266) 的一部分引入的，目的是解决自其在 JDK 8 中引入以来的常见投诉和建议，更具体地说，是对延迟和超时的支持，对子类化和一些实用程序方法的更好支持。

就代码而言，API 提供了八个新方法和五个新静态方法。为了实现这样的添加，大约 2400 行代码中的 1500 行被改变(根据开放 JDK)。

## 2。实例 API 添加

如上所述，实例 API 增加了八个新特性，它们是:

1.  `Executor defaultExecutor()`
2.  `CompletableFuture newIncompleteFuture()`
3.  `CompletableFuture<T> copy()`
4.  `CompletionStage<T> minimalCompletionStage()`
5.  `CompletableFuture<T> completeAsync(Supplier<? extends T> supplier, Executor executor)`
6.  `CompletableFuture<T> complete async(Supplier <?extends T> supplier)`
7.  `CompletableFuture<T> orTimeout(long timeout, TimeUnit unit)`
8.  `CompletableFuture<T> completeOnTimeout(T value, long timeout, TimeUnit unit)`

### 2.1。`defaultExecutor()`法

**签名** : `Executor defaultExecutor()`

返回用于不指定`Executor`的异步方法的默认`Executor`。

```java
new CompletableFuture().defaultExecutor()
```

这可以被返回执行器的子类覆盖，至少提供一个独立的线程。

### 2.2。`newIncompleteFuture()`法

**签名** : `CompletableFuture newIncompleteFuture()`

`newIncompleteFuture`也称为“虚拟构造函数”，用于获取相同类型的新的可完成的未来实例。

```java
new CompletableFuture().newIncompleteFuture()
```

这个方法在子类化`CompletableFuture`时特别有用，主要是因为它在几乎所有返回新`CompletionStage`的方法内部使用，允许子类控制这些方法返回什么子类型。

### 2.3。`copy()`法

**签名** : `CompletableFuture<T> copy()`

该方法返回一个新的`CompletableFuture`,它:

*   当这个正常完成时，新的也正常完成
*   当这个异常 X 异常完成时，新的也异常完成，带有一个以 X 为原因的`CompletionException`

```java
new CompletableFuture().copy()
```

这种方法作为“防御性复制”的一种形式可能是有用的，以防止客户端完成，同时仍然能够在特定的`CompletableFuture`实例上安排依赖的动作。

### 2.4。`minimalCompletionStage()`法

**签名** : `CompletionStage<T> minimalCompletionStage()`

该方法返回一个新的`CompletionStage`,其行为方式与 copy 方法描述的完全相同，但是，这样的新实例在每次尝试检索或设置解析的值时都会抛出`UnsupportedOperationException`。

```java
new CompletableFuture().minimalCompletionStage()
```

一个包含所有可用方法的新的`CompletableFuture`可以通过使用`CompletionStage` API 上可用的`toCompletableFuture`方法来检索。

### 2.5。`completeAsync()`战法

使用提供的`Supplier`给出的值，应该使用`completeAsync`方法异步完成`CompletableFuture`。

**签名**:

```java
CompletableFuture<T> completeAsync(Supplier<? extends T> supplier, Executor executor)
CompletableFuture<T> completeAsync(Supplier<? extends T> supplier)
```

这两个重载方法的区别在于第二个参数的存在，在这里可以指定运行任务的`Executor`。如果没有提供，将使用默认的执行器(由`defaultExecutor`方法返回)。

### 2.6。`orTimeout()`战法

**签名** : `CompletableFuture<T> orTimeout(long timeout, TimeUnit unit)`

```java
new CompletableFuture().orTimeout(1, TimeUnit.SECONDS)
```

用`TimeoutException`异常解析`CompletableFuture` ，除非它在指定的超时前完成。

### 2.7。`completeOnTimeout()`法

**签名** : `CompletableFuture<T> completeOnTimeout(T value, long timeout, TimeUnit unit)`

```java
new CompletableFuture().completeOnTimeout(value, 1, TimeUnit.SECONDS)
```

用指定的值正常完成`CompletableFuture`，除非它在指定的超时前完成。

## 3。静态 API 添加

还添加了一些实用方法。它们是:

1.  `Executor delayedExecutor(long delay, TimeUnit unit, Executor executor)`
2.  `Executor delayedExecutor(long delay, TimeUnit unit)`
3.  ` CompletionStage completedStage(U value)`
4.  ` CompletionStage failedStage(Throwable ex)`
5.  ` CompletableFuture failedFuture(Throwable ex)`

### 3.1。`delayedExecutor`战法

**签名**:

```java
Executor delayedExecutor(long delay, TimeUnit unit, Executor executor)
Executor delayedExecutor(long delay, TimeUnit unit)
```

返回一个新的`Executor`,它在给定的延迟后(如果为非正值，则不延迟)向给定的基础执行器提交任务。每次延迟都从调用返回的执行程序的 execute 方法开始。如果没有指定执行者，将使用默认的执行者(`ForkJoinPool.commonPool()`)。

### 3.2。方法`completedStage`和`failedStage`和

**签名**:

```java
<U> CompletionStage<U> completedStage(U value)
<U> CompletionStage<U> failedStage(Throwable ex)
```

这个实用程序方法返回已经解析的`CompletionStage`实例，或者正常完成并带有一个值(`completedStage`)，或者异常完成并带有给定的异常(`failedStage`)。

### 3.3。`failedFuture`法

**签名** : ` CompletableFuture failedFuture(Throwable ex)`

failedFuture 方法增加了指定已经完成的异常`CompleatebleFuture`实例的能力。

## 4。用例示例

在本节中，我们将展示一些如何使用一些新 API 的例子。

### 4.1。延迟

这个例子将展示如何用一个特定的值将一个`CompletableFuture`的完成延迟一秒。这可以通过使用`completeAsync`方法和`delayedExecutor`来实现。

```java
CompletableFuture<Object> future = new CompletableFuture<>();
future.completeAsync(() -> input, CompletableFuture.delayedExecutor(1, TimeUnit.SECONDS));
```

### 4.2。超时值完成

另一种实现延迟结果的方法是使用`completeOnTimeout`方法。这个例子定义了一个`CompletableFuture`,如果给定的输入在 1 秒钟后仍未被解析，它将被解析。

```java
CompletableFuture<Object> future = new CompletableFuture<>();
future.completeOnTimeout(input, 1, TimeUnit.SECONDS);
```

### 4.3。超时

另一种可能是超时，用`TimeoutException`异常解决未来。例如，让`CompletableFuture`在给定的 1 秒后超时，在此之前不会完成。

```java
CompletableFuture<Object> future = new CompletableFuture<>();
future.orTimeout(1, TimeUnit.SECONDS);
```

## 5。结论

总之，Java 9 对`CompletableFuture` API 进行了一些补充，它现在对子类化有了更好的支持，由于有了`newIncompleteFuture`虚拟构造函数，可以控制大多数`CompletionStage` API 中返回的`CompletionStage`实例。

如前所述，它对延迟和超时有更好的支持。添加的实用方法遵循一种合理的模式，为`CompletableFuture`提供了一种指定解析实例的便捷方式。

本文中使用的例子可以在我们的 [GitHub 资源库](https://web.archive.org/web/20220628093044/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-9-improvements)中找到。