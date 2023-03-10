# Spring MVC 中的长轮询

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-mvc-long-polling>

## 1。概述

长轮询是服务器应用程序用来保持客户端连接直到信息可用的一种方法。这通常在服务器必须调用下游服务来获取信息并等待结果时使用。

在本教程中，我们将通过使用`[DeferredResult](https://web.archive.org/web/20220628061207/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/context/request/async/DeferredResult.html).` 来探索 Spring MVC 中长轮询的概念。我们将从使用`DeferredResult`来查看一个基本实现  开始 g，然后讨论我们如何处理错误和超时。最后，我们将看看如何测试所有这些。

## 2。长轮询使用`DeferredResult`

**我们可以在 Spring MVC 中使用`DeferredResult` 来异步处理入站 HTTP 请求。**它允许 HTTP 工作线程被释放出来处理其他传入的请求，并将工作卸载给另一个工作线程。因此，它有助于提高需要长时间计算或任意等待时间的请求的服务可用性。

我们上一篇关于 Spring 的[*deferred result*](/web/20220628061207/https://www.baeldung.com/spring-deferred-result)类的文章更深入地讨论了它的功能和用例。

### 2.1.出版者

让我们通过创建一个使用`DeferredResult. `的发布应用程序来开始我们的长轮询示例

**首先，让我们定义一个 Spring `@RestController`，它利用了`DeferredResult`，但是没有把它的工作卸载给另一个工作线程:**

```java
@RestController
@RequestMapping("/api")
public class BakeryController { 
    @GetMapping("/bake/{bakedGood}")
    public DeferredResult<String> publisher(@PathVariable String bakedGood, @RequestParam Integer bakeTime) {
        DeferredResult<String> output = new DeferredResult<>();
        try {
            Thread.sleep(bakeTime);
            output.setResult(format("Bake for %s complete and order dispatched. Enjoy!", bakedGood));
        } catch (Exception e) {
            // ...
        }
        return output;
    }
}
```

该控制器以与常规阻塞控制器相同的方式同步工作。因此，我们的 HTTP 线程被完全阻塞，直到`bakeTime`过去。如果我们的服务有大量的入站流量，这是不理想的。

**现在让我们通过将工作卸载到工作线程来异步设置输出:**

```java
private ExecutorService bakers = Executors.newFixedThreadPool(5);

@GetMapping("/bake/{bakedGood}")
public DeferredResult<String> publisher(@PathVariable String bakedGood, @RequestParam Integer bakeTime) {
    DeferredResult<String> output = new DeferredResult<>();
    bakers.execute(() -> {
        try {
            Thread.sleep(bakeTime);
            output.setResult(format("Bake for %s complete and order dispatched. Enjoy!", bakedGood));
        } catch (Exception e) {
            // ...
        }
    });
    return output;
}
```

在这个例子中，我们现在能够释放 HTTP worker 线程来处理其他请求。我们的`bakers` 池中的一个工作线程正在做这项工作，并将在完成后设置结果。**当 worker 调用`setResult`时，将允许容器线程响应调用客户端。**

我们的代码现在是长轮询的良好候选，与传统的阻塞控制器相比，它将使我们的服务更适用于入站 HTTP 请求。然而，我们还需要处理一些边缘情况，比如错误处理和超时处理。

**为了处理我们的工人抛出的检查错误，我们将使用`DeferredResult` :** 提供的`setErrorResult` 方法

```java
bakers.execute(() -> {
    try {
        Thread.sleep(bakeTime);
        output.setResult(format("Bake for %s complete and order dispatched. Enjoy!", bakedGood));
     } catch (Exception e) {
        output.setErrorResult("Something went wrong with your order!");
     }
});
```

工作线程现在能够优雅地处理任何抛出的异常。

由于长轮询经常被实现来异步和同步地处理来自下游系统的响应，**我们应该添加一个机制来在我们从未收到来自下游系统的响应的情况下强制超时。**`DeferredResult`API 为此提供了一种机制。首先，我们在`DeferredResult`对象的构造函数中传递一个超时参数:

```java
DeferredResult<String> output = new DeferredResult<>(5000L);
```

接下来，让我们实现超时场景。为此，我们将使用`onTimeout:`

```java
output.onTimeout(() -> output.setErrorResult("the bakery is not responding in allowed time"));
```

**它接受一个`Runnable`作为输入——当达到超时阈值时，它由容器线程调用。**如果超时了，那么我们就把它当作一个错误来处理，并相应地使用`setErrorResult` `.`

### 2.2.订户

现在我们已经设置了发布应用程序，让我们编写一个订阅客户端应用程序。

编写一个调用这个长轮询 API 的服务相当简单，因为它本质上与编写一个标准阻塞 REST 调用的客户端是一样的。唯一真正的区别是，由于长轮询的等待时间，我们希望确保我们有一个超时机制。**在 Spring MVC 中，我们可以使用 [`RestTemplate`](/web/20220628061207/https://www.baeldung.com/rest-template) 或 [`WebClient`](/web/20220628061207/https://www.baeldung.com/spring-5-webclient) 来实现这一点，因为两者都有内置的超时处理。**

首先，让我们从一个使用`RestTemplate.` 的例子开始。让我们使用 `RestTemplateBuilder` 创建一个`RestTemplate` 的实例，这样我们就可以设置超时持续时间:

```java
public String callBakeWithRestTemplate(RestTemplateBuilder restTemplateBuilder) {
    RestTemplate restTemplate = restTemplateBuilder
      .setConnectTimeout(Duration.ofSeconds(10))
      .setReadTimeout(Duration.ofSeconds(10))
      .build();

    try {
        return restTemplate.getForObject("/api/bake/cookie?bakeTime=1000", String.class);
    } catch (ResourceAccessException e) {
        // handle timeout
    }
}
```

在这段代码中，通过从长轮询调用中捕获`ResourceAccessException`，我们能够在超时时处理错误。

接下来，让我们创建一个使用`WebClient`来实现相同结果的例子:

```java
public String callBakeWithWebClient() {
    WebClient webClient = WebClient.create();
    try {
        return webClient.get()
          .uri("/api/bake/cookie?bakeTime=1000")
          .retrieve()
          .bodyToFlux(String.class)
          .timeout(Duration.ofSeconds(10))
          .blockFirst();
    } catch (ReadTimeoutException e) {
        // handle timeout
    }
}
```

我们之前关于设置[弹簧休息超时](/web/20220628061207/https://www.baeldung.com/spring-rest-timeout)的文章更深入地讨论了这个主题。

## 3。测试长轮询

现在我们已经启动并运行了我们的应用程序，让我们来讨论如何测试它。**我们可以从使用 [`MockMvc`](https://web.archive.org/web/20220628061207/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/web/servlet/MockMvc.html) 来测试对控制器类**的调用开始

```java
MvcResult asyncListener = mockMvc
  .perform(MockMvcRequestBuilders.get("/api/bake/cookie?bakeTime=1000"))
  .andExpect(request().asyncStarted())
  .andReturn();
```

这里，我们调用我们的`DeferredResult`端点，并断言请求已经启动了一个异步调用。从这里开始，测试将等待异步结果的完成，这意味着我们不需要在测试中添加任何等待逻辑。

**接下来，我们想要断言异步调用何时返回，并且它匹配我们所期望的值:**

```java
String response = mockMvc
  .perform(asyncDispatch(asyncListener))
  .andReturn()
  .getResponse()
  .getContentAsString();

assertThat(response)
  .isEqualTo("Bake for cookie complete and order dispatched. Enjoy!");
```

通过使用`asyncDispatch()`，我们可以获得异步调用的响应并断言其值。

为了测试我们的`DeferredResult`的超时机制，我们需要通过在`asyncListener`和`response`调用之间添加一个超时使能器来稍微修改测试代码:

```java
((MockAsyncContext) asyncListener
  .getRequest()
  .getAsyncContext())
  .getListeners()
  .get(0)
  .onTimeout(null);
```

这段代码可能看起来很奇怪，但是我们这样调用`onTimeout`是有特定原因的。我们这样做是为了让`[AsyncListener](https://web.archive.org/web/20220628061207/https://docs.oracle.com/javaee/7/api/javax/servlet/AsyncListener.html)`知道操作已经超时。这将确保我们为控制器中的`onTimeout`方法实现的`Runnable`类被正确调用。

## 4。结论

在本文中，我们介绍了如何在长轮询环境中使用`DeferredResult`。我们还讨论了如何为长轮询编写订阅客户端，以及如何对其进行测试。源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220628061207/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-rest-http-2)