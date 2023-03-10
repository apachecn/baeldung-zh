# 用 Mockito 测试回调

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mockito-callbacks>

## 1.概观

在这个简短的教程中，我们将关注**如何使用流行的测试框架 [Mockito](https://web.archive.org/web/20220525123704/http://site.mockito.org/) 测试回调**。

我们将探索两种解决方案，**首先使用一个`ArgumentCaptor`，然后是直观的`doAnswer()`方法**。

要了解更多关于使用 Mockito 进行良好测试的信息，请查看我们的 Mockito 系列[这里](/web/20220525123704/https://www.baeldung.com/tag/mockito/)。

## 2.回电简介

**回调是作为参数传递给方法的一段代码，期望它在给定的时间**回调(执行)参数。

在同步回调中，这种执行可能是立即执行的，但在异步回调中，这种执行可能会在稍后发生。

使用回调的一个常见场景是服务交互期间的**，此时我们需要处理来自服务调用**的响应。

在本教程中，我们将使用下面显示的`Service`接口作为测试用例中的合作者:

```java
public interface Service {
    void doAction(String request, Callback<Response> callback);
}
```

在`Callback`参数中，我们传递一个使用`reply(T response)`方法处理响应的类:

```java
public interface Callback<T> {
    void reply(T response);
} 
```

### 2.1.简单的服务

我们还将使用一个简单的**服务示例来演示如何传递和调用回调**:

```java
public void doAction() {
    service.doAction("our-request", new Callback<Response>() {
        @Override
        public void reply(Response response) {
            handleResponse(response);
        }
    });
} 
```

在向`Response`对象添加一些数据之前，`handleResponse`方法检查响应是否有效:

```java
private void handleResponse(Response response) {
    if (response.isValid()) {
        response.setData(new Data("Successful data response"));
    }
}
```

为了清楚起见，我们选择不使用 Java Lamda 表达式，但是`service.doAction` **调用也可以写得更简洁**:

```java
service.doAction("our-request", response -> handleResponse(response)); 
```

要了解更多关于 Lambda 表达式的信息，请看这里的。

## 3.使用`ArgumentCaptor`

现在让我们看看**我们如何使用一个** `**ArgumentCaptor**:`来使用 Mockito 抓取`Callback`对象

```java
@Test
public void givenServiceWithValidResponse_whenCallbackReceived_thenProcessed() {
    ActionHandler handler = new ActionHandler(service);
    handler.doAction();

    verify(service).doAction(anyString(), callbackCaptor.capture());

    Callback<Response> callback = callbackCaptor.getValue();
    Response response = new Response();
    callback.reply(response);

    String expectedMessage = "Successful data response";
    Data data = response.getData();
    assertEquals(
      "Should receive a successful message: ", 
      expectedMessage, data.getMessage());
}
```

在这个例子中，我们首先创建一个`ActionHandler `，然后调用这个处理程序的`doAction` 方法。这只是一个简单服务`doAction`方法调用的包装器，我们在这里调用回调函数。

接下来，我们验证在我们的模拟服务实例上调用了`doAction `，将 **`anyString()`** 作为第一个参数，将 **`callbackCaptor.capture() `作为第二个参数，这是我们捕获`Callback`对象**的地方。然后可以使用`getValue()`方法返回参数的捕获值。

现在我们已经得到了`Callback`对象，在**直接调用`reply`方法并断言响应数据具有正确的值**之前，我们创建一个默认有效的`Response`对象。

## 4.使用`doAnswer()`方法

现在我们来看一个**通用解决方案，它使用 Mockito 的 **`Answer`对象和`doAnswer`方法来存根化 void 方法** `doAction:`回调**的方法

```java
@Test
public void givenServiceWithInvalidResponse_whenCallbackReceived_thenNotProcessed() {
    Response response = new Response();
    response.setIsValid(false);

    doAnswer((Answer<Void>) invocation -> {
        Callback<Response> callback = invocation.getArgument(1);
        callback.reply(response);

        Data data = response.getData();
        assertNull("No data in invalid response: ", data);
        return null;
    }).when(service)
        .doAction(anyString(), any(Callback.class));

    ActionHandler handler = new ActionHandler(service);
    handler.doAction();
} 
```

在我们的第二个例子中，我们首先创建一个无效的`Response`对象，这个对象将在测试中使用。

接下来，我们在模拟服务上设置`Answer` ，这样当`doAction`被调用时，**我们拦截调用，并使用`invocation.getArgument(1)`** 获取方法参数，以获得`Callback` 参数`. `

最后一步是创建`ActionHandler `并调用`doAction`，这将导致`Answer`被调用。

要了解更多关于 stubbing void 方法的信息，请看这里的。

## 3.结论

在这篇简短的文章中，我们介绍了用 Mockito 测试时两种不同的测试回调方法。

和往常一样，这些例子在这个 [GitHub 项目](https://web.archive.org/web/20220525123704/https://github.com/eugenp/tutorials/tree/master/testing-modules/mockito)中可用。