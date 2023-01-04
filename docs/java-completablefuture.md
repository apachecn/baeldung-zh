# CompletableFuture 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-completablefuture>

## 1。简介

本教程是对作为 Java 8 并发 API 改进引入的`CompletableFuture`类的功能和用例的指南。

## 延伸阅读:

## [Java 中的可运行与可调用](/web/20220812013211/https://www.baeldung.com/java-runnable-callable)

Learn the difference between Runnable and Callable interfaces in Java.[Read more](/web/20220812013211/https://www.baeldung.com/java-runnable-callable) →

## [Java . util . concurrent . future 指南](/web/20220812013211/https://www.baeldung.com/java-future)

A guide to java.util.concurrent.Future with an overview of its several implementations[Read more](/web/20220812013211/https://www.baeldung.com/java-future) →

## 2。Java 中的异步计算

异步计算很难推理。通常我们想把任何计算看作是一系列的步骤，但是在异步计算的情况下，被表示为回调的**动作往往要么分散在代码中，要么相互嵌套**。当我们需要处理其中一个步骤中可能发生的错误时，事情会变得更糟。

Java 5 中添加了`Future`接口作为异步计算的结果，但是它没有任何方法来组合这些计算或处理可能的错误。

**Java 8 引入了`CompletableFuture`类。**和`Future`接口一起，它还实现了`CompletionStage`接口。该接口为我们可以与其他步骤结合的异步计算步骤定义了契约。

`CompletableFuture`同时也是一个构建模块和框架，**有大约 50 种不同的方法来组成、组合和执行异步计算步骤以及处理错误**。

如此庞大的 API 可能会让人不知所措，但这些大多属于几个清晰而独特的用例。

## 3。使用`CompletableFuture`作为简单的`Future`

首先，`CompletableFuture`类实现了`Future`接口，所以我们可以**将其作为`Future`实现，但是带有额外的完成逻辑**。

例如，我们可以用一个无参数的构造函数创建这个类的一个实例来表示一些未来的结果，将它分发给消费者，并在未来的某个时间使用`complete`方法完成它。消费者可以使用`get`方法来阻塞当前线程，直到提供这个结果。

在下面的例子中，我们有一个方法，它创建一个`CompletableFuture`实例，然后在另一个线程中进行一些计算，并立即返回`Future`。

当计算完成时，该方法通过向`complete`方法提供结果来完成`Future`:

```
public Future<String> calculateAsync() throws InterruptedException {
    CompletableFuture<String> completableFuture = new CompletableFuture<>();

    Executors.newCachedThreadPool().submit(() -> {
        Thread.sleep(500);
        completableFuture.complete("Hello");
        return null;
    });

    return completableFuture;
}
```

为了剥离计算，我们使用了`Executor` API。这种创建和完成`CompletableFuture`的方法可以与任何并发机制或 API 一起使用，包括原始线程。

注意，`calculateAsync`方法的**返回了一个`Future`实例**。

我们简单地调用该方法，接收`Future`实例，并在准备好阻塞结果时对其调用`get`方法。

还可以观察到，`get`方法抛出了一些检查过的异常，即`ExecutionException`(封装了在计算过程中发生的异常)和`InterruptedException`(表示执行方法的线程被中断的异常):

```
Future<String> completableFuture = calculateAsync();

// ... 

String result = completableFuture.get();
assertEquals("Hello", result);
```

**如果我们已经知道了一个计算**的结果，我们可以使用静态`completedFuture`方法，带有一个表示这个计算结果的参数。因此，`Future`的`get`方法永远不会阻塞，而是立即返回这个结果:

```
Future<String> completableFuture = 
  CompletableFuture.completedFuture("Hello");

// ...

String result = completableFuture.get();
assertEquals("Hello", result);
```

作为一个替代方案，我们可能要将 [**取消`Future`**](/web/20220812013211/https://www.baeldung.com/java-future#2-canceling-a-future-with-cancel) 的执行。

## 4。`CompletableFuture`具有封装的计算逻辑

上面的代码允许我们选择任何并发执行的机制，但是如果我们想跳过这个样板文件，简单地异步执行一些代码呢？

静态方法`runAsync`和`supplyAsync`允许我们相应地从`Runnable`和`Supplier`功能类型中创建一个`CompletableFuture`实例。

由于新的 Java 8 特性，`Runnable`和`Supplier`都是函数接口，允许将它们的实例作为 lambda 表达式传递。

`Runnable`接口是线程中使用的同一个旧接口，它不允许返回值。

`Supplier`接口是一个普通的函数接口，它有一个没有参数的方法，并返回一个参数化类型的值。

这允许我们**提供一个`Supplier`的实例作为 lambda 表达式，执行计算并返回结果**。这很简单，因为:

```
CompletableFuture<String> future
  = CompletableFuture.supplyAsync(() -> "Hello");

// ...

assertEquals("Hello", future.get());
```

## 5。异步计算的处理结果

处理计算结果的最普通的方法是将它提供给一个函数。`thenApply`方法正是这样做的；它接受一个`Function`实例，用它来处理结果，并返回一个保存函数返回值的`Future`:

```
CompletableFuture<String> completableFuture
  = CompletableFuture.supplyAsync(() -> "Hello");

CompletableFuture<String> future = completableFuture
  .thenApply(s -> s + " World");

assertEquals("Hello World", future.get());
```

如果我们不需要沿着`Future`链返回一个值，我们可以使用`Consumer`函数接口的一个实例。它的单个方法接受一个参数并返回`void`。

在`CompletableFuture.` 中有一个用于这个用例的方法。`thenAccept`方法接收一个`Consumer`，并把计算结果传递给它。然后最后一个`future.get()`调用返回一个`Void`类型的实例:

```
CompletableFuture<String> completableFuture
  = CompletableFuture.supplyAsync(() -> "Hello");

CompletableFuture<Void> future = completableFuture
  .thenAccept(s -> System.out.println("Computation returned: " + s));

future.get();
```

最后，如果我们既不需要计算的值，也不想在链的末端返回某个值，那么我们可以将一个`Runnable` lambda 传递给`thenRun`方法。在下面的例子中，我们只是在调用了`future.get():`之后在控制台中打印了一行

```
CompletableFuture<String> completableFuture 
  = CompletableFuture.supplyAsync(() -> "Hello");

CompletableFuture<Void> future = completableFuture
  .thenRun(() -> System.out.println("Computation finished."));

future.get();
```

## 6。组合期货

`CompletableFuture` API 最好的部分是在一系列计算步骤中组合`CompletableFuture`实例的**能力。**

这种链接的结果本身就是一个`CompletableFuture`，它允许进一步的链接和组合。这种方法在函数式语言中无处不在，通常被称为一元设计模式。

**在下面的例子中，我们使用`thenCompose`方法来顺序链接两个`Futures`。**

注意，这个方法采用了一个返回`CompletableFuture`实例的函数。此函数的自变量是上一个计算步骤的结果。这允许我们在下一个`CompletableFuture`的 lambda 中使用这个值:

```
CompletableFuture<String> completableFuture 
  = CompletableFuture.supplyAsync(() -> "Hello")
    .thenCompose(s -> CompletableFuture.supplyAsync(() -> s + " World"));

assertEquals("Hello World", completableFuture.get());
```

`thenCompose`方法和`thenApply,`一起实现了一元模式的基本构件。它们与 Java 8 中的`Stream`和`Optional`类的`map`和`flatMap`方法密切相关。

两种方法都接收一个函数并将其应用于计算结果，但是`thenCompose` ( `flatMap`)方法**接收一个函数，该函数返回另一个相同类型的对象**。这种功能结构允许将这些类的实例组合成构建块。

如果我们想执行两个独立的`Futures`并对它们的结果做一些事情，我们可以使用接受带有两个参数的`Future`和`Function`的`thenCombine`方法来处理两个结果:

```
CompletableFuture<String> completableFuture 
  = CompletableFuture.supplyAsync(() -> "Hello")
    .thenCombine(CompletableFuture.supplyAsync(
      () -> " World"), (s1, s2) -> s1 + s2));

assertEquals("Hello World", completableFuture.get());
```

一个更简单的情况是，当我们想要用两个`Futures`结果做一些事情，但是不需要通过`Future`链传递任何结果值。`thenAcceptBoth`方法有助于:

```
CompletableFuture future = CompletableFuture.supplyAsync(() -> "Hello")
  .thenAcceptBoth(CompletableFuture.supplyAsync(() -> " World"),
    (s1, s2) -> System.out.println(s1 + s2));
```

## 7。`thenApply()`与`thenCompose()`和的区别

在前面的章节中，我们已经展示了关于`thenApply()`和`thenCompose()`的例子。这两个 API 都有助于链接不同的`CompletableFuture`调用，但是这两个函数的用法是不同的。

### 7.1。`thenApply()`

我们可以使用这个方法来处理前一次调用的结果。然而，要记住的一个关键点是，返回类型将是所有调用的组合。

所以当我们想要转换一个`CompletableFuture `调用的结果时，这个方法是有用的:

```
CompletableFuture<Integer> finalResult = compute().thenApply(s-> s + 1);
```

### 7.2。`thenCompose()`

`thenCompose()`方法与`thenApply()`相似，都返回一个新的完成阶段。然而， **`thenCompose()`使用前一阶段作为自变量**。它将展平并直接返回一个`Future`结果，而不是像我们在`thenApply():`中看到的那样嵌套未来

```
CompletableFuture<Integer> computeAnother(Integer i){
    return CompletableFuture.supplyAsync(() -> 10 + i);
}
CompletableFuture<Integer> finalResult = compute().thenCompose(this::computeAnother);
```

所以如果想法是链接`CompletableFuture`方法，那么最好使用`thenCompose()`。

另外，注意这两种方法之间的区别类似于[`map()``flatMap()`](/web/20220812013211/https://www.baeldung.com/java-difference-map-and-flatmap)`.`之间的区别

## 8。并行运行多个`Futures`

当我们需要并行执行多个`Futures`时，我们通常希望等待它们全部执行，然后处理它们的组合结果。

`CompletableFuture.allOf`静态方法允许等待作为 var-arg 提供的所有`Futures`的完成:

```
CompletableFuture<String> future1  
  = CompletableFuture.supplyAsync(() -> "Hello");
CompletableFuture<String> future2  
  = CompletableFuture.supplyAsync(() -> "Beautiful");
CompletableFuture<String> future3  
  = CompletableFuture.supplyAsync(() -> "World");

CompletableFuture<Void> combinedFuture 
  = CompletableFuture.allOf(future1, future2, future3);

// ...

combinedFuture.get();

assertTrue(future1.isDone());
assertTrue(future2.isDone());
assertTrue(future3.isDone());
```

注意，`CompletableFuture.allOf()`的返回类型是一个`CompletableFuture<Void>`。这种方法的局限性在于，它不会返回所有`Futures`的组合结果。相反，我们必须手动从`Futures`获取结果。幸运的是，`CompletableFuture.join()`方法和 Java 8 Streams API 使它变得简单:

```
String combined = Stream.of(future1, future2, future3)
  .map(CompletableFuture::join)
  .collect(Collectors.joining(" "));

assertEquals("Hello Beautiful World", combined);
```

`CompletableFuture.join()`方法类似于`get`方法，但是如果`Future`没有正常完成，它会抛出一个未检查的异常。这使得在`Stream.map()`方法中使用它作为方法参考成为可能。

## 9。处理错误

对于一系列异步计算步骤中的错误处理，我们必须以类似的方式调整`throw/catch`习语。

与在语法块中捕捉异常不同，`CompletableFuture`类允许我们用一种特殊的`handle`方法来处理它。该方法接收两个参数:计算结果(如果成功完成)，以及抛出的异常(如果某个计算步骤没有正常完成)。

在下面的例子中，当由于没有提供名称而导致问候语的异步计算出错时，我们使用`handle`方法来提供默认值:

```
String name = null;

// ...

CompletableFuture<String> completableFuture  
  =  CompletableFuture.supplyAsync(() -> {
      if (name == null) {
          throw new RuntimeException("Computation error!");
      }
      return "Hello, " + name;
  }).handle((s, t) -> s != null ? s : "Hello, Stranger!");

assertEquals("Hello, Stranger!", completableFuture.get());
```

作为另一个场景，假设我们想要手动完成带有值的`Future`,就像第一个例子一样，但是也有能力用异常来完成它。`completeExceptionally`方法就是为此而设计的。以下示例中的`completableFuture.get()`方法抛出一个以`RuntimeException`为原因的`ExecutionException`:

```
CompletableFuture<String> completableFuture = new CompletableFuture<>();

// ...

completableFuture.completeExceptionally(
  new RuntimeException("Calculation failed!"));

// ...

completableFuture.get(); // ExecutionException
```

在上面的例子中，我们可以用`handle`方法异步处理异常，但是使用`get`方法，我们可以使用更典型的同步异常处理方法。

## 10。异步方法

`CompletableFuture`类中的大多数 fluent API 方法都有两个带`Async`后缀的附加变量。这些方法通常是为了让**在另一个线程**中运行相应的执行步骤。

没有后缀`Async`的方法使用调用线程运行下一个执行阶段。相比之下，没有`Executor`参数的`Async`方法使用`Executor`的公共`fork/join`池实现运行一个步骤，该池实现通过`ForkJoinPool.commonPool()`方法访问。最后，带有`Executor`参数的`Async`方法使用传递的`Executor`运行一个步骤。

下面是一个修改后的例子，用一个`Function`实例处理计算结果。唯一可见的区别是`thenApplyAsync`方法，但是在幕后，函数的应用程序被包装在一个`ForkJoinTask`实例中(关于`fork/join`框架的更多信息，请参见文章[“Java 中的 Fork/Join 框架指南”](/web/20220812013211/https://www.baeldung.com/java-fork-join))。这使我们能够进一步并行化计算，并更高效地利用系统资源:

```
CompletableFuture<String> completableFuture  
  = CompletableFuture.supplyAsync(() -> "Hello");

CompletableFuture<String> future = completableFuture
  .thenApplyAsync(s -> s + " World");

assertEquals("Hello World", future.get());
```

## 11\. JDK 9 `CompletableFuture` API

Java 9 通过以下变化增强了`CompletableFuture` API:

*   添加了新的工厂方法
*   支持延迟和超时
*   改进了对子类化的支持

和新实例 API:

*   `Executor defaultExecutor()`
*   `CompletableFuture newIncompleteFuture()`
*   `CompletableFuture<T> copy()`
*   `CompletionStage<T> minimalCompletionStage()`
*   `CompletableFuture<T> completeAsync(Supplier<? extends T> supplier, Executor executor)`
*   `CompletableFuture<T> completeAsync(Supplier<? extends T> supplier)`
*   `CompletableFuture<T> orTimeout(long timeout, TimeUnit unit)`
*   `CompletableFuture<T> completeOnTimeout(T value, long timeout, TimeUnit unit)`

我们现在也有一些静态实用程序方法:

*   `Executor delayedExecutor(long delay, TimeUnit unit, Executor executor)`
*   `Executor delayedExecutor(long delay, TimeUnit unit)`
*   ` CompletionStage completedStage(U value)`
*   ` CompletionStage failedStage(Throwable ex)`
*   ` CompletableFuture failedFuture(Throwable ex)`

最后，为了解决超时问题，Java 9 引入了另外两个新函数:

*   `orTimeout()`
*   `completeOnTimeout()`

下面是可供进一步阅读的详细文章:[Java 9 completable 未来 API 改进](/web/20220812013211/https://www.baeldung.com/java-9-completablefuture)。

## 12。结论

在本文中，我们描述了`CompletableFuture`类的方法和典型用例。

这篇文章的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220812013211/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-simple)