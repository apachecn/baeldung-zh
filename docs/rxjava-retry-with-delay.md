# 在 RxJava 中延迟重试

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rxjava-retry-with-delay>

## 1.概观

在本教程中，**我们将看到如何在 RxJava** 中使用延迟重试。和`Observable`一起工作的时候，很有可能得到的是`error`而不是`success`。在这种情况下，我们可以使用`retry`延迟一段时间后重试。

## 2.项目设置

首先，让我们创建一个 Maven 或 Gradle 项目。这里，**我们使用基于 Maven 的项目**。让我们将`rxjava`依赖项添加到我们的`pom.xml`文件中:

```java
<dependency>
    <groupId>io.reactivex.rxjava2</groupId>
    <artifactId>rxjava</artifactId>
    <version>2.2.21</version>
</dependency>
```

依赖关系可以在 [Maven 中心页面](https://web.archive.org/web/20221118043414/https://search.maven.org/artifact/io.reactivex.rxjava2/rxjava/2.2.21/jar)上找到。

## 3.在可观察的生命周期中重试

当`Observable`源发出错误时，使用`Retry`。这是对源的重新订阅，希望在下一次尝试中没有错误地完成。因此，`retry()`及其其他变体只有在`Observable` 发出错误时才会出现。

### 3.1.重试的重载变体

除了上一节讨论的`retry()`方法，RxJava 还提供了两个重载变量，`retry(long)` 和`retry(Func2)`。

`retry(long)`返回一个镜像源`Observable`的`Observable`。只要这个镜像调用`onError`指定的次数，我们就重新订阅它。

`retry(Func2)` 方法接受一个接受一个`Throwable` 作为输入并产生一个`boolean `作为输出的函数。这里，我们仅在镜像针对特定的`Exception `和重试次数返回`true `时才重新订阅镜像。

### 3.2.`RetryWhen`对`Retry`

`retryWhen(Func1)`为我们提供了一个包含定制逻辑的选项，以确定我们是否应该重新订阅原始`Observable`的镜像。这个方法接受一个接受由`onError `处理程序抛出的`exception `的函数。

如果函数发出一个项目，我们重新订阅镜像，而如果它产生一个 o `nError`通知，我们就不订阅。

## 4.代码示例

现在，让我们借助一些代码片段来观察这些概念的实际应用。

### 4.1.成功`Observable`

If an `Observable` call succeeds, then the Observer's `onNext` method is invoked. Let's see an example:

```java
@Test
public void givenObservable_whenSuccess_thenOnNext(){
    Observable.just(remoteCallSuccess())
        .subscribe(success -> {
            System.out.println("Success");
            System.out.println(success);
        }, err -> {
            System.out.println("Error");
            System.out.println(err);
        });
}
```

方法`remoteCallSuccess() `成功完成并返回一个`String`。在这种情况下，订户的`onNext()` 方法被调用，打印成功消息。

### 4.2.错误可观察值

Let's introduce a new utility function called `remoteCallError()` that mocks calling a remote server. This method is vulnerable to errors. Therefore, our `retryWhen` logic would kick in:

```java
@Test
public void givenObservable_whenError_thenOnError(){
    Observable.just(remoteCallError())
        .subscribe(success -> {
            System.out.println("Success");
            System.out.println(success);
        }, err -> {
            System.out.println("Error");
            System.out.println(err);
        });
}
```

在这种情况下，`remoteCallError() `方法会导致错误。因此，订户的`onError() `方法被调用。因此，会打印错误消息。

### 4.3.延迟重试

We can add a delay during the resubscription process:

```java
@Test
public void givenError_whenRetry_thenCanDelay(){
    Observable.just(remoteCallError())
        .retryWhen(attempts -> {
            return attempts.flatMap(err -> {
                if (customChecker(err)) {
                    return Observable.timer(5000, TimeUnit.MILLISECONDS);
                } else {
                    return Observable.error(err);
                }
            });
        });
}
```

与前一种情况类似，`remoteCallError() `方法会导致错误。然而，在这里，我们没有通知观察者的`onError() `方法，而是给了这个`Observable`一个重新描述的机会。为此，我们将异常传递给一个`customChecker() `方法，该方法返回一个`boolean`。如果响应是`true`，我们返回一个新的`Observable `，延迟 5000 毫秒(5 秒)。否则，我们返回一个错误并通知观察者。

## 5.结论

In this tutorial, we saw how to use `retryWhen` and some of its related functions. We understood the cases under which `retry` and `retryWhen` help us solve our issues. The source code is available [over on GitHub](https://web.archive.org/web/20221118043414/https://github.com/eugenp/tutorials/tree/master/rxjava-modules/rxjava-core).