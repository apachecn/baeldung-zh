# 重试假装呼叫

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/feign-retry>

## 1.介绍

通过 REST 端点调用外部服务是一种常见的活动，像 [Feign](/web/20220617075715/https://www.baeldung.com/intro-to-feign) 这样的库使这种活动变得非常简单。然而，在这样的通话中，很多事情都可能出错。这些问题中有许多是随机的或暂时的。

在本教程中，我们将学习如何重试失败的调用，并使 REST 客户端更有弹性。

## 2.假装客户端设置

首先，让我们创建一个简单的 Feign client builder，稍后我们将使用重试特性来增强它。我们将使用 [`OkHttpClient`](/web/20220617075715/https://www.baeldung.com/guide-to-okhttp) 作为 HTTP 客户端。此外，我们将使用`GsonEncoder`和`GsonDecoder`对请求和响应进行编码和解码。最后，我们需要指定目标的 URI 和响应类型:

```java
public class ResilientFeignClientBuilder {
    public static <T> T createClient(Class<T> type, String uri) {
        return Feign.builder()
          .client(new OkHttpClient())
          .encoder(new GsonEncoder())
          .decoder(new GsonDecoder())
          .target(type, uri);
    }
}
```

或者，如果我们使用 Spring，我们可以让它用可用的 beans 自动连接 Feign client。

## 3.假装`Retryer`

幸运的是，重试能力已经内置在 Feign 中，只需要进行配置即可。**我们可以通过向客户端构建器提供`Retryer`接口的实现来做到这一点。**

它最重要的方法，`continueOrPropagate,` 接受`RetryableException`作为参数，不返回任何内容。在执行时，它要么抛出异常，要么成功退出(通常在休眠后)。**如果没有抛出异常，Feign 会继续重试调用。**如果异常被抛出，它将被传播，并有效地以错误结束调用。

### 3.1.天真的实现

让我们编写一个非常简单的 Retryer 实现，它总是在等待一秒钟后重试调用:

```java
public class NaiveRetryer implements feign.Retryer {
    @Override
    public void continueOrPropagate(RetryableException e) {
        try {
            Thread.sleep(1000L);
        } catch (InterruptedException ex) {
            Thread.currentThread().interrupt();
            throw e;
        }
    }
} 
```

因为`Retryer`实现了`Cloneable`接口，我们还需要覆盖`clone`方法。

```java
@Override
public Retryer clone() {
    return new NaiveRetryer();
} 
```

最后，我们需要将我们的实现添加到客户端构建器中:

```java
public static <T> T createClient(Class<T> type, String uri) {
    return Feign.builder()
      // ...
      .retryer(new NaiveRetryer())    
      // ...
}
```

或者，如果我们正在使用 Spring，我们可以用 [`@Component`](/web/20220617075715/https://www.baeldung.com/spring-component-annotation) 注释来注释`NaiveRetryer`，或者在配置类中定义一个 [bean](/web/20220617075715/https://www.baeldung.com/spring-bean) ，让 Spring 完成剩下的工作:

```java
@Bean
public Retryer retryer() {
    return new NaiveRetryer();
}
```

### 3.2.默认实现

Feign 提供了一个合理的默认实现`Retryer`接口。**它只会重试给定的次数，以一定的时间间隔开始，然后随着每次重试而增加，直到达到规定的最大值。**我们来定义一下，开始间隔 100 毫秒，最大间隔 3 秒，最大尝试次数 5:

```java
public static <T> T createClient(Class<T> type, String uri) {
    return Feign.builder()
// ...
      .retryer(new Retryer.Default(100L, TimeUnit.SECONDS.toMillis(3L), 5))    
// ...
}
```

### 3.3.不重试

如果我们不想让 Feign 重试任何调用，我们可以向客户端构建器提供`Retryer.NEVER_RETRY` 实现。它每次都会简单地传播异常。

## 4.创建可重试的异常

在上一节中，我们学习了如何控制重试呼叫的频率。现在让我们看看如何控制何时我们想要重试调用，何时我们想要简单地抛出异常。

### 4.1.`ErrorDecoder`和`RetryableException`

当我们收到一个错误的响应时，Feign 将它传递给`ErrorDecoder`接口的一个实例，该实例决定如何处理它。最重要的是，解码器可以将异常映射到`RetryableException,`的一个实例，使`Retryer`能够重试调用。**`ErrorDecoder`的默认实现仅在响应包含“重试后”报头时创建一个`RetryableExeception`实例。最常见的是，我们可以在 503 服务不可用响应中找到它。**

这是很好的默认行为，但有时我们需要更加灵活。例如，我们可能正在与一个外部服务进行通信，该服务不时地随机响应 500 内部服务器错误，而我们没有能力修复它。我们能做的是重试调用，因为我们知道下次它可能会工作。为了实现这一点，我们需要编写一个定制的`ErrorDecoder`实现。

### 4.2.创建自定义错误解码器

在我们的自定义解码器中，我们只需要实现一个方法:`decode`。它接受两个参数，一个`String`方法键和一个`Response`对象。它返回一个异常，应该是一个`RetryableException`的实例或者其他一些依赖于实现的异常。

我们的`decode`方法将简单地检查响应的状态代码是否大于或等于 500。如果是这种情况，它将创建`RetryableException`。如果没有，它将从`FeignException`类返回用`errorStatus`工厂函数创建的基本`FeignException`:

```java
public class Custom5xxErrorDecoder implements ErrorDecoder {
    @Override
    public Exception decode(String methodKey, Response response) {
        FeignException exception = feign.FeignException.errorStatus(methodKey, response);
        int status = response.status();
        if (status >= 500) {
            return new RetryableException(
              response.status(),
              exception.getMessage(),
              response.request().httpMethod(),
              exception,
              null,
              response.request());
        }
        return exception;
    }
}
```

请注意，在这种情况下，我们创建并返回异常，而不是抛出异常。

最后，我们需要将解码器插入客户端构建器:

```java
public static <T> T createClient(Class<T> type, String uri) {
    return Feign.builder()
      // ...
      .errorDecoder(new Custom5xxErrorDecoder())
      // ...
}
```

## 5.摘要

在本文中，我们学习了如何控制 Feign 库的重试逻辑。我们研究了`Retryer`接口，以及如何使用它来控制重试的时间和次数。然后我们创建了我们的`ErrorDecoder`来控制哪些响应需要重试。

和往常一样，所有代码示例都可以在 GitHub 上找到[。](https://web.archive.org/web/20220617075715/https://github.com/eugenp/tutorials/tree/master/feign)