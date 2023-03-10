# 自定义 Zuul 例外

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/zuul-customize-exception>

## 1.概观

**[Zuul](/web/20220524064206/https://www.baeldung.com/spring-rest-with-zuul-proxy) 是网飞**基于 JVM 的路由器和服务器端负载均衡器。Zuul 的规则引擎提供了编写规则和过滤器的灵活性，以增强 Spring Cloud 微服务架构中的路由。

在本文中，我们将通过编写在代码执行过程中发生错误时运行的**自定义错误过滤器，来探索如何在 Zuul 中自定义异常和错误响应。**

## 2.祖尔例外

Zuul 中所有被处理的异常都是`ZuulExceptions`。现在，让我们明确一下 **`ZuulException`不能被 [`@ControllerAdvice`](/web/20220524064206/https://www.baeldung.com/exception-handling-for-rest-with-spring#controlleradvice) 抓住，用 [`@ExceptionHandling`](/web/20220524064206/https://www.baeldung.com/exception-handling-for-rest-with-spring#exceptionhandler)** 注释方法。这是因为 **`ZuulException`** **是从误差过滤器**中抛出的。因此，它会跳过后续的滤波器链，永远不会到达错误控制器。下图描述了 Zuul 中错误处理的层次结构:

[![Customizing Zuul Exceptions](img/a4f7cfd835805bb5dbf1ca61c26a530e.png)](/web/20220524064206/https://www.baeldung.com/wp-content/uploads/2022/02/zuul.png)

当有`ZuulException`时，Zuul 显示以下错误响应:

```java
{
    "timestamp": "2022-01-23T22:43:43.126+00:00",
    "status": 500,
    "error": "Internal Server Error"
}
```

在某些场景中，我们可能需要定制错误消息或状态代码，以响应`ZuulException.` Zuul 过滤器的救援。在下一节中，我们将讨论如何扩展 Zuul 的错误过滤器和定制`ZuulException.`

## 3.自定义 Zuul 例外

`spring-cloud-starter-netflix-zuul`的启动包包括三种类型的过滤器:[前置](/web/20220524064206/https://www.baeldung.com/spring-rest-with-zuul-proxy#filter)、[后置](/web/20220524064206/https://www.baeldung.com/zuul-filter-modifying-response-body)和误差过滤器。在这里，**我们将深入研究错误过滤器，并探索名为`SendErrorFilter`** 的 Zuul 错误过滤器的定制。

首先，我们将禁用自动配置的默认`SendErrorFilter`。这使我们不必担心执行的顺序，因为这是唯一的 Zuul 默认错误过滤器。让我们在`application.yml`中添加属性来禁用它:

```java
zuul:
  SendErrorFilter:
    post:
      disable: true
```

现在，让我们编写一个名为`CustomZuulErrorFilter`的自定义 Zuul 错误过滤器，如果底层服务不可用，它将抛出一个自定义异常:

```java
public class CustomZuulErrorFilter extends ZuulFilter {
}
```

这个自定义过滤器需要扩展`com.netflix.zuul.ZuulFilter`并覆盖它的一些方法``.``

首先，我们必须用**覆盖`filterType()`方法，并将类型作为`“error”`** 返回。这是因为我们要为误差过滤器类型配置 Zuul 过滤器:

```java
@Override
public String filterType() {
    return "error";
}
```

之后，我们**覆盖`filterOrder()`并返回`-1,`，这样过滤器就是链中的第一个**:

```java
@Override
public int filterOrder() {
    return -1;
}
```

然后，我们**覆盖`shouldFilter()`方法并无条件返回`true`** ，因为我们想在所有情况下链接这个过滤器:

```java
@Override
public boolean shouldFilter() {
    return true;
}
```

最后，让我们用**覆盖`run()`方法**:

```java
@Override
public Object run() {
    RequestContext context = RequestContext.getCurrentContext();
    Throwable throwable = context.getThrowable();

    if (throwable instanceof ZuulException) {
        ZuulException zuulException = (ZuulException) throwable;
        if (throwable.getCause().getCause().getCause() instanceof ConnectException) {
            context.remove("throwable");
            context.setResponseBody(RESPONSE_BODY);
            context.getResponse()
                .setContentType("application/json");
            context.setResponseStatusCode(503);
        }
    }
    return null;
}
```

让我们分解这个`run()`方法来理解它在做什么。首先，我们获得了`RequestContext`的实例。接下来，我们验证从`RequestContext`获得的`throwable`是否是`ZuulException`的实例。然后，我们检查`throwable`中嵌套异常的原因是否是`ConnectException`的一个实例。最后，我们用响应的自定义属性设置了上下文。

请注意，在设置自定义响应之前，我们从上下文中清除了`throwable`，以防止后续过滤器中的进一步错误处理。

此外，我们还可以在我们的`run()`方法中设置一个定制的异常，它可以被后续的过滤器处理:

```java
if (throwable.getCause().getCause().getCause() instanceof ConnectException) {
    ZuulException customException = new ZuulException("", 503, "Service Unavailable");
    context.setThrowable(customException);
}
```

上面的代码片段将记录堆栈跟踪，并继续下一个过滤器。

此外，我们可以修改这个例子来处理`ZuulFilter.`中的多个异常

## 4.测试自定义 Zuul 异常

在这一节中，我们将在`CustomZuulErrorFilter`中测试自定义 Zuul 异常。

假设有一个`ConnectException`，上面例子在 Zuul API 响应中的输出将是:

```java
{
    "timestamp": "2022-01-23T23:10:25.584791Z",
    "status": 503,
    "error": "Service Unavailable"
}
```

此外，**我们总是可以通过配置`application.yml`文件中的`error.path`属性**来改变默认的 Zuul 错误转发路径`/error `。

现在，让我们通过一些测试案例来验证它:

```java
@Test
public void whenSendRequestWithCustomErrorFilter_thenCustomError() {
    Response response = RestAssured.get("http://localhost:8080/foos/1");
    assertEquals(503, response.getStatusCode());
}
```

在上面的测试场景中，`/foos/1`的路由被有意保留下来，导致了`java.lang.` `ConnectException`。因此，我们的自定义过滤器将截取并响应 503 状态。

现在，让我们在不注册自定义错误过滤器的情况下对此进行测试:

```java
@Test
public void whenSendRequestWithoutCustomErrorFilter_thenError() {
    Response response = RestAssured.get("http://localhost:8080/foos/1");
    assertEquals(500, response.getStatusCode());
}
```

在没有注册定制错误过滤器的情况下执行上述测试用例会导致 Zuul 以状态 500 进行响应。

## 5.结论

在本教程中，我们学习了错误处理的层次结构，并深入研究了在 Spring Zuul 应用程序中配置自定义 Zuul 错误过滤器。这个错误过滤器提供了定制响应体和响应代码的机会。像往常一样，GitHub 上的示例代码[可用。](https://web.archive.org/web/20220524064206/https://github.com/eugenp/tutorials/tree/master/spring-cloud/spring-cloud-zuul/spring-zuul-ui)