# Java 中的异步编程

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-asynchronous-programming>

## 1.概观

随着编写非阻塞代码需求的增长，我们需要异步执行代码的方法。

在本教程中，我们将看看用 Java 实现[异步编程](/web/20221007090003/https://www.baeldung.com/cs/async-vs-multi-threading)的几种方法。我们还将探索一些提供开箱即用解决方案的 Java 库。

## 2.Java 中的异步编程

### 2.1.`Thread`

我们可以创建一个新线程来异步执行任何操作。随着 Java 8 中 [lambda 表达式](/web/20221007090003/https://www.baeldung.com/java-8-lambda-expressions-tips)的发布，它变得更干净，可读性更强。

让我们创建一个新的线程来计算并打印一个数的阶乘:

```
int number = 20;
Thread newThread = new Thread(() -> {
    System.out.println("Factorial of " + number + " is: " + factorial(number));
});
newThread.start();
```

### 2.2.`FutureTask`

从 Java 5 开始，`Future`接口提供了一种使用`[FutureTask](/web/20221007090003/https://www.baeldung.com/java-future#1-implementing-futures-with-futuretask)`执行异步操作的方法。

我们可以使用`[ExecutorService](https://web.archive.org/web/20221007090003/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/ExecutorService.html)`的`submit`方法异步执行任务，并返回`FutureTask`的实例。

所以让我们找出一个数的阶乘:

```
ExecutorService threadpool = Executors.newCachedThreadPool();
Future<Long> futureTask = threadpool.submit(() -> factorial(number));

while (!futureTask.isDone()) {
    System.out.println("FutureTask is not finished yet..."); 
} 
long result = futureTask.get(); 

threadpool.shutdown();
```

这里我们使用了由`Future`接口提供的`isDone`方法来检查任务是否完成。一旦完成，我们可以使用`get`方法检索结果。

### 2.3.`CompletableFuture`

**Java 8 引入了 [`CompletableFuture`](/web/20221007090003/https://www.baeldung.com/java-completablefuture) 与`Future`[`CompletionStage`](https://web.archive.org/web/20221007090003/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/CompletionStage.html)的组合。**为异步编程提供了`supplyAsync`、`runAsync`、`thenApplyAsync`等多种方法。

现在让我们用`CompletableFuture`代替`FutureTask`来求一个数的阶乘:

```
CompletableFuture<Long> completableFuture = CompletableFuture.supplyAsync(() -> factorial(number));
while (!completableFuture.isDone()) {
    System.out.println("CompletableFuture is not finished yet...");
}
long result = completableFuture.get();
```

我们不需要显式地使用`ExecutorService`。 **`CompletableFuture` 内部使用 [`ForkJoinPool`](/web/20221007090003/https://www.baeldung.com/java-fork-join) 异步处理任务**。因此，它使我们的代码更加干净。

## 3.番石榴

[**番石榴**](/web/20221007090003/https://www.baeldung.com/guava-21-new)提供了`[ListenableFuture](https://web.archive.org/web/20221007090003/https://guava.dev/releases/28.2-jre/api/docs/index.html?com/google/common/util/concurrent/ListenableFuture.html)`类来执行异步操作。

首先，我们来添加最新的 [`guava`](https://web.archive.org/web/20221007090003/https://search.maven.org/search?q=g:com.google.guava%20a:guava) 美文依赖:

```
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

然后让我们用`ListenableFuture`来求一个数的阶乘:

```
ExecutorService threadpool = Executors.newCachedThreadPool();
ListeningExecutorService service = MoreExecutors.listeningDecorator(threadpool);
ListenableFuture<Long> guavaFuture = (ListenableFuture<Long>) service.submit(()-> factorial(number));
long result = guavaFuture.get();
```

这里的`[MoreExecutors](https://web.archive.org/web/20221007090003/https://guava.dev/releases/28.2-jre/api/docs/index.html?com/google/common/util/concurrent/MoreExecutors.html)`类提供了`[ListeningExecutorService](https://web.archive.org/web/20221007090003/https://guava.dev/releases/28.2-jre/api/docs/index.html?com/google/common/util/concurrent/ListeningExecutorService.html)`类的实例。然后`ListeningExecutorService.submit`方法异步执行任务并返回`ListenableFuture`的实例。

**Guava 还有一个 [`Futures`](https://web.archive.org/web/20221007090003/https://guava.dev/releases/28.2-jre/api/docs/com/google/common/util/concurrent/Futures.html) 类，它提供了像`submitAsync`、`scheduleAsync`和`transformAsync`这样的方法来链接类似于`CompletableFuture.`** 的`ListenableFutures,`

例如，让我们看看如何使用`Futures.submitAsync`来代替`ListeningExecutorService.submit`方法:

```
ListeningExecutorService service = MoreExecutors.listeningDecorator(threadpool);
AsyncCallable<Long> asyncCallable = Callables.asAsyncCallable(new Callable<Long>() {
    public Long call() {
        return factorial(number);
    }
}, service);
ListenableFuture<Long> guavaFuture = Futures.submitAsync(asyncCallable, service);
```

这里的`submitAsync`方法需要一个 [`AsyncCallable`](https://web.archive.org/web/20221007090003/https://guava.dev/releases/28.2-jre/api/docs/com/google/common/util/concurrent/AsyncCallable.html) 参数，它是使用 [`Callables`](https://web.archive.org/web/20221007090003/https://guava.dev/releases/28.2-jre/api/docs/com/google/common/util/concurrent/Callables.html) 类创建的。

此外，`Futures`类提供了`addCallback`方法来注册成功和失败的回调:

```
Futures.addCallback(
  factorialFuture,
  new FutureCallback<Long>() {
      public void onSuccess(Long factorial) {
          System.out.println(factorial);
      }
      public void onFailure(Throwable thrown) {
          thrown.getCause();
      }
  }, 
  service);
```

## 4.EA 异步

**电子艺界带来了异步等待功能。NET 通过 [`ea-async`库](https://web.archive.org/web/20221007090003/https://github.com/electronicarts/ea-async)到 Java 生态系统。**

这个库允许顺序编写异步(非阻塞)代码。因此，它使得异步编程更加容易，并且可以自然地伸缩。

首先，我们将最新的 [`ea-async`](https://web.archive.org/web/20221007090003/https://search.maven.org/search?q=g:com.ea.async%20a:ea-async) Maven 依赖项添加到`pom.xml`:

```
<dependency>
    <groupId>com.ea.async</groupId>
    <artifactId>ea-async</artifactId>
    <version>1.2.3</version>
</dependency>
```

然后我们将通过使用 EA 的`[Async](https://web.archive.org/web/20221007090003/https://javadoc.io/doc/com.ea.async/ea-async/latest/com/ea/async/Async.html)`类提供的`await`方法来转换前面讨论的`CompletableFuture`代码:

```
static { 
    Async.init(); 
}

public long factorialUsingEAAsync(int number) {
    CompletableFuture<Long> completableFuture = CompletableFuture.supplyAsync(() -> factorial(number));
    long result = Async.await(completableFuture);
}
```

这里我们调用`static`块中的`Async.init`方法来初始化`Async`运行时工具。

`Async`检测在运行时转换代码，并重写对`await`方法的调用，使其行为类似于使用`CompletableFuture`链。

因此，**对`await` 方法的调用类似于调用`Future.join.`**

我们可以使用–`javaagent`JVM 参数进行编译时检测。这是`Async.init` 方法的替代方法:

```
java -javaagent:ea-async-1.2.3.jar -cp <claspath> <MainClass>
```

现在让我们看另一个顺序编写异步代码的例子。

首先，我们将使用`CompletableFuture`类的`thenComposeAsync`和`thenAcceptAsync` 等组合方法异步执行一些链操作:

```
CompletableFuture<Void> completableFuture = hello()
  .thenComposeAsync(hello -> mergeWorld(hello))
  .thenAcceptAsync(helloWorld -> print(helloWorld))
  .exceptionally(throwable -> {
      System.out.println(throwable.getCause()); 
      return null;
  });
completableFuture.get();
```

然后我们可以使用 EA 的`Async.await()`转换代码:

```
try {
    String hello = await(hello());
    String helloWorld = await(mergeWorld(hello));
    await(CompletableFuture.runAsync(() -> print(helloWorld)));
} catch (Exception e) {
    e.printStackTrace();
}
```

**实现类似于顺序阻塞代码；然而，`await`方法不会阻塞代码。**

如前所述，所有对`await`方法的调用都将被`Async`工具重写，以类似于`Future.join` 方法的方式工作。

所以一旦`hello`方法的异步执行完成，`Future`结果就被传递给`mergeWorld`方法。然后使用`CompletableFuture.runAsync`方法将结果传递给最后一次执行。

## 5.卡图斯

Cactoos 是一个基于面向对象原则的 Java 库。

它是 Google Guava 和 Apache Commons 的替代品，为执行各种操作提供了公共对象。

首先，让我们添加最新的`[cactoos](https://web.archive.org/web/20221007090003/https://search.maven.org/search?q=g:org.cactoos%20a:cactoos)` Maven 依赖项:

```
<dependency>
    <groupId>org.cactoos</groupId>
    <artifactId>cactoos</artifactId>
    <version>0.43</version>
</dependency>
```

这个库为异步操作提供了一个`[Async](https://web.archive.org/web/20221007090003/https://javadoc.io/doc/org.cactoos/cactoos/latest/org/cactoos/func/Async.html)`类。

因此，我们可以使用 Cactoos 的`Async`类的实例找到一个数字的阶乘:

```
Async<Integer, Long> asyncFunction = new Async<Integer, Long>(input -> factorial(input));
Future<Long> asyncFuture = asyncFunction.apply(number);
long result = asyncFuture.get();
```

这里**`apply`方法使用`ExecutorService.submit`方法执行操作，并返回`Future`接口**的一个实例。

类似地，`Async`类有`exec`方法，它提供相同的特性，但没有返回值。

注意:Cactoos 库处于开发的初级阶段，可能还不适合生产使用。

## 6.JCA bi-方面

Jcabi-Aspects 通过 [AspectJ](/web/20221007090003/https://www.baeldung.com/aspectj) AOP 方面为异步编程提供了`[@Async](https://web.archive.org/web/20221007090003/https://aspects.jcabi.com/apidocs-0.22.6/com/jcabi/aspects/Async.html)`注解。

首先，让我们添加最新的 [`jcabi-aspects`](https://web.archive.org/web/20221007090003/https://search.maven.org/search?q=g:com.jcabi%20a:jcabi-aspects) 美文依赖:

```
<dependency>
    <groupId>com.jcabi</groupId>
    <artifactId>jcabi-aspects</artifactId>
    <version>0.22.6</version>
</dependency> 
```

`jcabi-aspects`库需要 AspectJ 运行时支持，所以我们将添加 [`aspectjrt`](https://web.archive.org/web/20221007090003/https://search.maven.org/search?q=g:org.aspectj%20a:aspectjrt) Maven 依赖项:

```
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjrt</artifactId>
    <version>1.9.5</version>
</dependency> 
```

接下来，我们将添加 [`jcabi-maven-plugin`](https://web.archive.org/web/20221007090003/https://search.maven.org/search?q=g:com.jcabi%20a:jcabi-maven-plugin) 插件，用 AspectJ 方面编织二进制文件。插件提供了为我们完成所有工作的`ajc`目标:

```
<plugin>
    <groupId>com.jcabi</groupId>
    <artifactId>jcabi-maven-plugin</artifactId>
    <version>0.14.1</version>
    <executions>
        <execution>
            <goals>
                <goal>ajc</goal>
            </goals>
        </execution>
    </executions>
    <dependencies>
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjtools</artifactId>
            <version>1.9.1</version>
        </dependency>
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.9.1</version>
        </dependency>
    </dependencies>
</plugin>
```

现在我们已经准备好使用 AOP 方面进行异步编程了:

```
@Async
@Loggable
public Future<Long> factorialUsingAspect(int number) {
    Future<Long> factorialFuture = CompletableFuture.completedFuture(factorial(number));
    return factorialFuture;
}
```

当我们编译代码时，**库将通过 AspectJ 编织注入 AOP 通知代替`@Async`注释，**用于异步执行`factorialUsingAspect`方法。

让我们使用 Maven 命令编译这个类:

```
mvn install
```

来自`jcabi-maven-plugin`的输出可能看起来像:

```
 --- jcabi-maven-plugin:0.14.1:ajc (default) @ java-async ---
[INFO] jcabi-aspects 0.18/55a5c13 started new daemon thread jcabi-loggable for watching of @Loggable annotated methods
[INFO] Unwoven classes will be copied to /tutorials/java-async/target/unwoven
[INFO] jcabi-aspects 0.18/55a5c13 started new daemon thread jcabi-cacheable for automated cleaning of expired @Cacheable values
[INFO] ajc result: 10 file(s) processed, 0 pointcut(s) woven, 0 error(s), 0 warning(s)
```

我们可以通过检查 Maven 插件生成的`jcabi-ajc.log`文件中的日志来验证我们的类是否被正确编织:

```
Join point 'method-execution(java.util.concurrent.Future 
com.baeldung.async.JavaAsync.factorialUsingJcabiAspect(int))' 
in Type 'com.baeldung.async.JavaAsync' (JavaAsync.java:158) 
advised by around advice from 'com.jcabi.aspects.aj.MethodAsyncRunner' 
(jcabi-aspects-0.22.6.jar!MethodAsyncRunner.class(from MethodAsyncRunner.java))
```

然后，我们将该类作为一个简单的 Java 应用程序运行，输出如下所示:

```
17:46:58.245 [main] INFO com.jcabi.aspects.aj.NamedThreads - 
jcabi-aspects 0.22.6/3f0a1f7 started new daemon thread jcabi-loggable for watching of @Loggable annotated methods
17:46:58.355 [main] INFO com.jcabi.aspects.aj.NamedThreads - 
jcabi-aspects 0.22.6/3f0a1f7 started new daemon thread jcabi-async for Asynchronous method execution
17:46:58.358 [jcabi-async] INFO com.baeldung.async.JavaAsync - 
#factorialUsingJcabiAspect(20): '[[email protected]](/web/20221007090003/https://www.baeldung.com/cdn-cgi/l/email-protection)[Completed normally]' in 44.64µs
```

正如我们所见，一个新的守护线程`jcabi-async,` 由异步执行任务的库创建。

类似地，日志记录由库提供的 [`@Loggable`](https://web.archive.org/web/20221007090003/https://aspects.jcabi.com/apidocs-0.22.6/com/jcabi/aspects/Loggable.html) 注释启用。

## 7.结论

在本文中，我们学习了几种 Java 异步编程的方法。

首先，我们探索了 Java 的内置特性，比如用于异步编程的`FutureTask`和`CompletableFuture` 。然后我们研究了一些库，比如 EA Async 和 Cactoos，它们都有现成的解决方案。

我们还讨论了使用 Guava 的`ListenableFuture`和`Futures`类异步执行任务的支持。最后，我们提到了 jcabi-AspectJ 库，它通过异步方法调用的`@Async`注释提供了 AOP 特性。

像往常一样，所有的代码实现都可以在 GitHub 上获得[。](https://web.archive.org/web/20221007090003/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-advanced-3)