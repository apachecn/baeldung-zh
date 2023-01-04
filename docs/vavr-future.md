# 虚拟现实未来简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/vavr-future>

## 1。简介

核心 Java 为异步计算提供了一个基本的 API—`Future.``CompletableFuture`是它的最新实现之一。

Vavr 为`Future` API 提供了新的功能替代品。在本文中，我们将讨论新的 API，并展示如何利用它的一些新特性。

更多关于 Vavr 的文章可以在这里找到[。](/web/20220625231501/https://www.baeldung.com/vavr-tutorial)

## 2。Maven 依赖关系

`Future` API 包含在 Vavr Maven 依赖项中。

所以，让我们把它添加到我们的`pom.xml`:

```
<dependency>
    <groupId>io.vavr</groupId>
    <artifactId>vavr</artifactId>
    <version>0.9.2</version>
</dependency>
```

我们可以在 [Maven Central](https://web.archive.org/web/20220625231501/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22vavr%22%20AND%20g%3A%22io.vavr%22) 上找到依赖的最新版本。

## 3。Vavr 的`Future`

**`Future`可以处于两种状态之一:**

*   待定–计算正在进行
*   completed—计算成功完成并产生结果，因异常而失败或被取消

**相对于核心 Java `Future`的主要优势是我们可以很容易地注册回调，并以非阻塞的方式组合操作。**

## 4。基本`Future`操作

### 4.1。开始异步计算

现在，让我们看看如何使用 Vavr 开始异步计算:

```
String initialValue = "Welcome to ";
Future<String> resultFuture = Future.of(() -> someComputation());
```

### 4.2。从`Future` 中检索值

我们可以通过简单地调用`get()`或`getOrElse()`方法之一从`Future`中提取值:

```
String result = resultFuture.getOrElse("Failed to get underlying value.");
```

`get()`和`getOrElse()`的区别在于`get()`是最简单的解决方案，而`getOrElse()`使我们能够返回任何类型的值，以防我们无法检索到`Future`中的值。

建议使用`getOrElse()`，这样我们可以处理在试图从`Future`中检索值时发生的任何错误。**为了简单起见，我们在接下来的几个例子中只使用`get()`。**

注意，如果需要等待结果的话，`get()`方法会阻塞当前线程。

一种不同的方法是调用非阻塞的`getValue()` 方法，该方法返回一个`Option<Try<T>>` ，只要计算挂起，这个`Option<Try<T>>` **就为空。**

然后，我们可以提取位于`Try`对象内部的计算结果:

```
Option<Try<String>> futureOption = resultFuture.getValue();
Try<String> futureTry = futureOption.get();
String result = futureTry.get();
```

有时，我们需要在检索值之前检查`Future`是否包含值。

我们可以简单地通过使用:

```
resultFuture.isEmpty();
```

重要的是要注意方法`isEmpty()`是阻塞的——它将阻塞线程直到它的操作完成。

### 4.3。`ExecutorService`改变默认

使用一个`ExecutorService`来异步运行他们的计算。默认的`ExecutorService`是`Executors.newCachedThreadPool()`。

我们可以通过传递我们选择的实现来使用另一个`ExecutorService`:

```
@Test
public void whenChangeExecutorService_thenCorrect() {
    String result = Future.of(newSingleThreadExecutor(), () -> HELLO)
      .getOrElse(error);

    assertThat(result)
      .isEqualTo(HELLO);
}
```

## 5。完成后执行动作

API 提供了`onSuccess()`方法，一旦`Future`成功完成，该方法就执行一个动作。

类似地，方法`onFailure()`在`Future`失败时执行。

让我们看一个简单的例子:

```
Future<String> resultFuture = Future.of(() -> appendData(initialValue))
  .onSuccess(v -> System.out.println("Successfully Completed - Result: " + v))
  .onFailure(v -> System.out.println("Failed - Result: " + v));
```

方法`onComplete()`接受一个动作，只要`Future`完成了它的执行，不管`Future`是否成功。方法`andThen()`类似于`onComplete()`——它只是保证回调以特定的顺序执行:

```
Future<String> resultFuture = Future.of(() -> appendData(initialValue))
  .andThen(finalResult -> System.out.println("Completed - 1: " + finalResult))
  .andThen(finalResult -> System.out.println("Completed - 2: " + finalResult));
```

## 6。`Futures`上有用的操作

### 6.1。阻塞当前线程

方法`await()`有两种情况:

*   如果`Future`是未决的，它阻塞当前线程，直到未来线程完成
*   如果`Future`完成，它会立即结束

使用这种方法很简单:

```
resultFuture.await();
```

### 6.2。取消计算

我们总是可以取消计算:

```
resultFuture.cancel();
```

### 6.3。`ExecutorService`检索底层

为了获得被一个`Future`使用的`ExecutorService`，我们可以简单地调用`executorService()`:

```
resultFuture.executorService();
```

### 6.4。从失败的`Future` 中获得一个`Throwable`

我们可以使用`getCause()`方法来实现，该方法返回包装在`io.vavr.control.Option`对象中的`Throwable`。

我们可以稍后从`Option`对象中提取`Throwable`:

```
@Test
public void whenDivideByZero_thenGetThrowable2() {
    Future<Integer> resultFuture = Future.of(() -> 10 / 0)
      .await();

    assertThat(resultFuture.getCause().get().getMessage())
      .isEqualTo("/ by zero");
}
```

此外，我们可以使用`failed()`方法将我们的实例转换为持有`Throwable`实例的`Future`:

```
@Test
public void whenDivideByZero_thenGetThrowable1() {
    Future<Integer> resultFuture = Future.of(() -> 10 / 0);

    assertThatThrownBy(resultFuture::get)
      .isInstanceOf(ArithmeticException.class);
}
```

### 6.5。`isCompleted(), isSuccess(),`和`isFailure()`

这些方法几乎是不言自明的。他们检查`Future`是否完成，它是成功完成还是失败。当然，它们都返回`boolean`值。

我们将在前一个示例中使用这些方法:

```
@Test
public void whenDivideByZero_thenCorrect() {
    Future<Integer> resultFuture = Future.of(() -> 10 / 0)
      .await();

    assertThat(resultFuture.isCompleted()).isTrue();
    assertThat(resultFuture.isSuccess()).isFalse();
    assertThat(resultFuture.isFailure()).isTrue();
}
```

### 6.6。在未来之上应用计算

`map()`方法允许我们在未决的`Future:`之上应用计算

```
@Test
public void whenCallMap_thenCorrect() {
    Future<String> futureResult = Future.of(() -> "from Baeldung")
      .map(a -> "Hello " + a)
      .await();

    assertThat(futureResult.get())
      .isEqualTo("Hello from Baeldung");
}
```

如果我们向`map()`方法传递一个返回`Future`的函数，我们可以得到一个嵌套的`Future`结构。为了避免这种情况，我们可以利用`flatMap()`方法:

```
@Test
public void whenCallFlatMap_thenCorrect() {
    Future<Object> futureMap = Future.of(() -> 1)
      .flatMap((i) -> Future.of(() -> "Hello: " + i));

    assertThat(futureMap.get()).isEqualTo("Hello: 1");
}
```

### 6.7。`Futures`变身

方法`transformValue()`可用于在`Future`之上应用计算，并将其中的值更改为相同类型或不同类型的另一个值:

```
@Test
public void whenTransform_thenCorrect() {
    Future<Object> future = Future.of(() -> 5)
      .transformValue(result -> Try.of(() -> HELLO + result.get()));

    assertThat(future.get()).isEqualTo(HELLO + 5);
}
```

### 6.8。拉拉链`Futures`

API 提供了将`Futures`压缩成元组的`zip()`方法——元组是几个元素的集合，这些元素可能彼此相关，也可能不相关。它们也可以是不同的类型。让我们看一个简单的例子:

```
@Test
public void whenCallZip_thenCorrect() {
    Future<String> f1 = Future.of(() -> "hello1");
    Future<String> f2 = Future.of(() -> "hello2");

    assertThat(f1.zip(f2).get())
      .isEqualTo(Tuple.of("hello1", "hello2"));
}
```

这里要注意的一点是，只要至少有一个基址`Futures`仍然是待定的，那么结果`Future`将是待定的。

### 6.9。`Futures`与`CompletableFutures`和之间的转换

API 支持与`java.util.CompletableFuture`的集成。因此，如果我们想执行只有核心 Java API 支持的操作，我们可以很容易地将一个`Future`转换成一个`CompletableFuture`。

让我们看看如何做到这一点:

```
@Test
public void whenConvertToCompletableFuture_thenCorrect()
  throws Exception {

    CompletableFuture<String> convertedFuture = Future.of(() -> HELLO)
      .toCompletableFuture();

    assertThat(convertedFuture.get())
      .isEqualTo(HELLO);
}
```

我们也可以使用`fromCompletableFuture()`方法将一个`CompletableFuture`转换成一个`Future`。

### 6.10。异常处理

当一个`Future`失败时，我们可以用几种方法来处理这个错误。

例如，我们可以利用方法`recover()`返回另一个结果，比如一个错误消息:

```
@Test
public void whenFutureFails_thenGetErrorMessage() {
    Future<String> future = Future.of(() -> "Hello".substring(-1))
      .recover(x -> "fallback value");

    assertThat(future.get())
      .isEqualTo("fallback value");
}
```

或者，我们可以使用`recoverWith()`返回另一个`Future`计算的结果:

```
@Test
public void whenFutureFails_thenGetAnotherFuture() {
    Future<String> future = Future.of(() -> "Hello".substring(-1))
      .recoverWith(x -> Future.of(() -> "fallback value"));

    assertThat(future.get())
      .isEqualTo("fallback value");
}
```

方法`fallbackTo()`是处理错误的另一种方式。它在一个`Future`上被调用，并接受另一个`Future`作为参数。

如果第一个`Future`成功，那么它返回它的结果。否则，如果第二个`Future`成功，那么它返回它的结果。如果两个`Futures`都失败，那么`failed()`方法返回一个`Throwable`的`Future`，它保存第一个`Future`的错误:

```
@Test
public void whenBothFuturesFail_thenGetErrorMessage() {
    Future<String> f1 = Future.of(() -> "Hello".substring(-1));
    Future<String> f2 = Future.of(() -> "Hello".substring(-2));

    Future<String> errorMessageFuture = f1.fallbackTo(f2);
    Future<Throwable> errorMessage = errorMessageFuture.failed();

    assertThat(
      errorMessage.get().getMessage())
      .isEqualTo("String index out of range: -1");
}
```

## 7。结论

在本文中，我们已经了解了什么是`Future`,并学习了它的一些重要概念。我们还通过几个实际例子介绍了 API 的一些特性。

GitHub 上有完整版本的代码[。](https://web.archive.org/web/20220625231501/https://github.com/eugenp/tutorials/tree/master/vavr)