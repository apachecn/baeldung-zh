# Vavr 试用指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/vavr-try>

## 1。概述

在本文中，**我们将看看一种函数式的错误处理方式，而不是标准的`try-catch`块。**

我们将使用来自`Vavr`库的`Try` 类，这将允许我们通过将错误处理嵌入到正常的程序处理流程中来创建更加流畅和有意识的 API。

如果你想了解更多关于 Vavr 的信息，可以查看[这篇文章](/web/20220625071934/https://www.baeldung.com/vavr)。

## 2。处理异常的标准方式

假设我们有一个简单的接口，带有一个方法`call()` ，该方法返回一个`Response` 或者抛出一个`ClientException`，这是一个在失败的情况下被检查的异常:

```java
public interface HttpClient {
    Response call() throws ClientException;
} 
```

`Response` 是一个简单的类，只有一个`id`字段:

```java
public class Response {
    public final String id;

    public Response(String id) {
        this.id = id;
    }
}
```

假设我们有一个调用`HttpClient,` 的服务，那么我们需要在一个标准的`try-catch`块中处理这个被检查的异常:

```java
public Response getResponse() {
    try {
        return httpClient.call();
    } catch (ClientException e) {
        return null;
    }
}
```

当我们想要创建流畅的、函数式编写的 API 时，每个抛出检查异常的方法都会破坏程序流，并且我们的程序代码由许多`try-catch`块组成，使得它非常难以阅读。

理想情况下，我们希望有一个特殊的类来封装结果状态(成功或失败)，然后我们可以根据结果来链接操作。

## 3。用`Try` 处理异常

Vavr 库为我们提供了一个特殊的容器**，它代表一个可能导致异常或者成功完成的计算**。

在`Try` 对象中封闭操作给了我们一个结果，要么是`Success` 要么是`Failure.`，然后我们可以根据该类型执行进一步的操作。

让我们看看使用`Try:`的方法`getResponse()`会是什么样子

```java
public class VavrTry {
    private HttpClient httpClient;

    public Try<Response> getResponse() {
        return Try.of(httpClient::call);
    }

    // standard constructors
}
```

需要注意的重要事情是返回类型`Try<Response>.` 当一个方法返回这样的结果类型时，我们需要正确地处理它，并且记住，结果类型可以是`Success`或`Failure`，所以我们需要在编译时显式地处理它。

### 3.1。处理`Success`

让我们编写一个测试用例，在`httpClient`返回成功结果的情况下使用我们的`Vavr`类。方法`getResponse()` 返回`Try<Resposne>` 对象。因此，我们可以对其调用`map()` 方法，该方法只有在`Try` 为`Success`类型时才会对`Response` 执行动作:

```java
@Test
public void givenHttpClient_whenMakeACall_shouldReturnSuccess() {
    // given
    Integer defaultChainedResult = 1;
    String id = "a";
    HttpClient httpClient = () -> new Response(id);

    // when
    Try<Response> response = new VavrTry(httpClient).getResponse();
    Integer chainedResult = response
      .map(this::actionThatTakesResponse)
      .getOrElse(defaultChainedResult);
    Stream<String> stream = response.toStream().map(it -> it.id);

    // then
    assertTrue(!stream.isEmpty());
    assertTrue(response.isSuccess());
    response.onSuccess(r -> assertEquals(id, r.id));
    response.andThen(r -> assertEquals(id, r.id)); 

    assertNotEquals(defaultChainedResult, chainedResult);
}
```

函数`actionThatTakesResponse()` 只是将`Response` 作为一个参数，并返回一个`id field:`的`hashCode`

```java
public int actionThatTakesResponse(Response response) {
    return response.id.hashCode();
}
```

一旦我们使用`actionThatTakesResponse()` 函数`map`我们的值，我们就执行方法`getOrElse()`。

如果`Try` 里面有一个`Success`，它返回`Try, o`的值，否则，它返回`defaultChainedResult`。我们的`httpClient`执行成功，因此`isSuccess` 方法返回 true。然后我们可以执行对一个`Response` 对象进行操作的`onSuccess()` 方法。`Try` 还有一个方法`andThen` ,该方法采用一个`Consumer` ,当值为`Success.` 时，该方法消耗一个`Try` 值

我们可以将我们的`Try` 反应视为一个流。为此，我们需要使用`toStream()`方法将其转换为`Stream` ，然后`Stream` 类中所有可用的操作都可以用于对该结果进行操作。

如果我们想在`Try` 类型上执行一个动作，我们可以使用以`Try` 作为参数的`transform()` 方法，并且在不打开封闭值`:`的情况下对其执行一个动作

```java
public int actionThatTakesTryResponse(Try<Response> response, int defaultTransformation){
    return response.transform(responses -> response.map(it -> it.id.hashCode())
      .getOrElse(defaultTransformation));
}
```

### 3.2。处理`Failure`

我们来写一个例子，我们的`HttpClient`在执行时会抛出`ClientException` 。

与前面的例子相比，我们的`getOrElse` 方法将返回`defaultChainedResult` ,因为`Try`将属于`Failure`类型:

```java
@Test
public void givenHttpClientFailure_whenMakeACall_shouldReturnFailure() {
    // given
    Integer defaultChainedResult = 1;
    HttpClient httpClient = () -> {
        throw new ClientException("problem");
    };

    // when
    Try<Response> response = new VavrTry(httpClient).getResponse();
    Integer chainedResult = response
        .map(this::actionThatTakesResponse)
        .getOrElse(defaultChainedResult);
     Option<Response> optionalResponse = response.toOption();

    // then
    assertTrue(optionalResponse.isEmpty());
    assertTrue(response.isFailure());
    response.onFailure(ex -> assertTrue(ex instanceof ClientException));
    assertEquals(defaultChainedResult, chainedResult);
}
```

方法`getReposnse()` 返回`Failure` ，因此方法`isFailure` 返回 true。

我们可以对返回的响应执行`onFailure()`回调，并看到异常属于`ClientException` 类型。属于`Try`类型的对象可以使用`toOption()` 方法映射到`Option` 类型。

当我们不想在所有代码库中携带我们的`Try` 结果，但是我们有使用`Option` 类型处理显式缺少值的方法时，这是有用的。当我们将我们的`Failure` 映射到`Option,` 时，方法`isEmpty()` 返回 true。当`Try` 对象是类型`Success` 时，在其上调用`toOption` 将使`Option`被定义，因此方法`isDefined()`将返回 true。

### 3.3。利用模式匹配

当我们的`httpClient`返回一个`Exception`时，我们可以对那个`Exception.`的类型进行模式匹配，然后根据`recover() a` 方法中的那个`Exception`的类型，我们可以决定我们是否想要从那个异常中恢复并将我们的`Failure` 转换为`Success`，或者我们是否想要将我们的计算结果保留为`Failure:`

```java
@Test
public void givenHttpClientThatFailure_whenMakeACall_shouldReturnFailureAndNotRecover() {
    // given
    Response defaultResponse = new Response("b");
    HttpClient httpClient = () -> {
        throw new RuntimeException("critical problem");
    };

    // when
    Try<Response> recovered = new VavrTry(httpClient).getResponse()
      .recover(r -> Match(r).of(
          Case(instanceOf(ClientException.class), defaultResponse)
      ));

    // then
    assertTrue(recovered.isFailure());
```

只有当异常的类型是`ClientException.`时，`recover()` 方法中的模式匹配才会将`Failure` 转换为`Success` ，否则，它会将其作为`Failure().` 我们看到我们的 httpClient 正在抛出`RuntimeException` ，因此我们的恢复方法不会处理这种情况，因此`isFailure()` 返回 true。

如果我们想从`recovered` 对象获得结果，但在关键故障的情况下会再次抛出该异常，我们可以使用`getOrElseThrow()`方法来实现:

```java
recovered.getOrElseThrow(throwable -> {
    throw new RuntimeException(throwable);
});
```

有些错误很严重，当它们发生时，我们希望通过在调用堆栈中抛出更高的异常来明确地发出信号，让调用者决定进一步的异常处理。在这种情况下，像上面的例子一样重新抛出异常是非常有用的。

当我们的客户抛出一个非关键异常时，我们在一个`recover()` 方法中的模式匹配将把我们的`Failure`变成`Success.`，我们正在从两种类型的异常`ClientException` 和`IllegalArgumentException`中恢复:

```java
@Test
public void givenHttpClientThatFailure_whenMakeACall_shouldReturnFailureAndRecover() {
    // given
    Response defaultResponse = new Response("b");
    HttpClient httpClient = () -> {
        throw new ClientException("non critical problem");
    };

    // when
    Try<Response> recovered = new VavrTry(httpClient).getResponse()
      .recover(r -> Match(r).of(
        Case(instanceOf(ClientException.class), defaultResponse),
        Case(instanceOf(IllegalArgumentException.class), defaultResponse)
       ));

    // then
    assertTrue(recovered.isSuccess());
}
```

我们看到`isSuccess()` 返回 true，因此我们的恢复处理代码成功工作。

## 4。结论

本文展示了 Vavr 库中`Try`容器的一个实际应用。我们查看了通过以更具功能性的方式处理故障来使用该结构的实际例子。使用`Try` 将允许我们创建更多功能性和可读性更好的 API。

所有这些示例和代码片段的实现都可以在 [GitHub 项目](https://web.archive.org/web/20220625071934/https://github.com/eugenp/tutorials/tree/master/vavr)中找到——这是一个基于 Maven 的项目，因此它应该很容易导入和运行。