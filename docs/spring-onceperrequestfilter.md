# 什么是 OncePerRequestFilter？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-onceperrequestfilter>

## 1.概观

在本教程中，我们将学习`OncePerRequestFilter`，春天的一种特殊类型的过滤器。我们将通过一个简单的例子来看看它解决了什么问题，并理解如何使用它。

## 2.什么是`OncePerRequestFilter`？

我们先来了解一下滤镜的工作原理。可以在 servlet 执行之前或之后调用`[Filter](/web/20220525122016/https://www.baeldung.com/spring-boot-add-filter)`。当一个请求被分派给一个 servlet 时，`RequestDispatcher`可能会将它转发给另一个 servlet。有可能另一个 servlet 也有相同的过滤器。在这种情况下，同一个**过滤器会被多次调用。**

但是，我们可能希望确保每个请求只调用一次特定的过滤器。一个常见的用例是使用 Spring Security。当请求通过过滤器链时，我们可能希望某些身份验证操作只对请求发生一次。

在这种情况下，我们可以延长`OncePerRequestFilter`。 **Spring 保证对于一个给定的请求,`OncePerRequestFilter`只执行一次。**

## 3.使用`OncePerRequestFilter`进行同步请求

我们举个例子来了解一下这个滤镜的使用方法。我们将定义一个扩展了`OncePerRequestFilter`的类`AuthenticationFilter`，并覆盖`doFilterInternal()`方法:

```
public class AuthenticationFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(
            HttpServletRequest request,
            HttpServletResponse response,
            FilterChain filterChain) throws
            ServletException, IOException {
        String usrName = request.getHeader(“userName”);
        logger.info("Successfully authenticated user  " +
                userName);
        filterChain.doFilter(request, response);
    }
}
```

由于`OncePerRequestFilter`只支持 HTTP 请求**，**，所以不需要像我们实现`Filter`接口时那样转换`request`和`response`对象。

## 4.对异步请求使用`OncePerRequestFilter`

对于异步请求，默认情况下不会应用`OncePerRequestFilter`。我们需要覆盖方法`shouldNotFilterAsyncDispatch()`和`shouldNotFilterErrorDispatch()`来支持这一点。

有时，我们需要仅在初始请求线程中应用过滤器，而不是在异步分派中创建的附加线程中。其他时候，我们可能需要在每个额外的线程中至少调用一次过滤器。在这种情况下，我们需要覆盖`shouldNotFilterAsyncDispatch()`方法。

如果`shouldNotFilterAsyncDispatch()` 方法返回`true`，那么过滤器将不会被后续的异步分派调用。然而，如果它返回`false`，过滤器将为每个异步分派调用，每个线程恰好调用一次。

类似地，**我们将覆盖`shouldNotFilterErrorDispatch()`方法，并返回`true`或`false`，这取决于我们是否要过滤错误派单**:

```
@Component
public class AuthenticationFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(
      HttpServletRequest request,
      HttpServletResponse response,
      FilterChain filterChain) throws ServletException, IOException {
        String usrName = request.getHeader("userName");
        logger.info("Successfully authenticated user  " +
          usrName);
        filterChain.doFilter(request, response);
    }

    @Override
    protected boolean shouldNotFilterAsyncDispatch() {
        return false;
    }

    @Override
    protected boolean shouldNotFilterErrorDispatch() {
        return false;
    }
}
```

## 5.有条件地跳过请求

我们可以通过覆盖`shouldNotFilter()`方法，有条件地将过滤器仅应用于某些特定的请求，并跳过其他请求:

```
@Override
protected boolean shouldNotFilter(HttpServletRequest request) throws ServletException {
    return Boolean.TRUE.equals(request.getAttribute(SHOULD_NOT_FILTER));
}
```

## 6.快速示例

让我们看一个简单的例子来理解`OncePerRequestFilter`的行为。
首先，我们将定义一个`Controller`，它使用 Spring 的 [`DeferredResult`](/web/20220525122016/https://www.baeldung.com/spring-deferred-result) 异步处理请求:

```
@Controller
public class HelloController  {
    @GetMapping(path = "/greeting")
    public DeferredResult<String> hello(HttpServletResponse response) throws Exception {
        DeferredResult<String> deferredResult = new DeferredResult<>();
        executorService.submit(() -> perform(deferredResult));
        return deferredResult;
    }
    private void perform(DeferredResult<String> dr) {
        // some processing 
        dr.setResult("OK");
    }
}
```

当异步处理请求时，两个线程都经过同一个过滤器链。因此，过滤器被调用两次:第一次是在容器线程处理请求时，第二次是在异步调度程序完成之后。一旦异步处理完成，响应就返回给客户机。

现在，让我们定义一个实现`OncePerRequestFilter`的`Filter`:

```
@Component
public class MyOncePerRequestFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
      throws ServletException, IOException {
        logger.info("Inside Once Per Request Filter originated by request {}", request.getRequestURI());
        filterChain.doFilter(request, response);
    }

    @Override
    protected boolean shouldNotFilterAsyncDispatch() {
        return true;
    }
}
```

在上面的代码中，我们有意从`shouldNotFilterAsyncDispatch()`方法中返回了`true`。这是为了证明我们的过滤器只为容器线程调用一次，而不是为后续的异步线程调用。

让我们调用端点来演示这一点:

```
curl -X GET http://localhost:8082/greeting 
```

**输出:**

```
10:23:24.175 [http-nio-8082-exec-1] INFO  o.a.c.c.C.[Tomcat].[localhost].[/] - Initializing Spring DispatcherServlet 'dispatcherServlet'
10:23:24.175 [http-nio-8082-exec-1] INFO  o.s.web.servlet.DispatcherServlet - Initializing Servlet 'dispatcherServlet'
10:23:24.176 [http-nio-8082-exec-1] INFO  o.s.web.servlet.DispatcherServlet - Completed initialization in 1 ms
10:23:26.814 [http-nio-8082-exec-1] INFO  c.b.O.MyOncePerRequestFilter - Inside OncePer Request Filter originated by request /greeting
```

现在，让我们看看我们希望请求和异步调度都调用我们的过滤器的情况。我们只需要覆盖`shouldNotFilterAsyncDispatch()`以返回`false`来实现这一点:

```
@Override
protected boolean shouldNotFilterAsyncDispatch() {
    return false;
}
```

**输出:**

```
2:53.616 [http-nio-8082-exec-1] INFO  o.a.c.c.C.[Tomcat].[localhost].[/] - Initializing Spring DispatcherServlet 'dispatcherServlet'
10:32:53.616 [http-nio-8082-exec-1] INFO  o.s.web.servlet.DispatcherServlet - Initializing Servlet 'dispatcherServlet'
10:32:53.617 [http-nio-8082-exec-1] INFO  o.s.web.servlet.DispatcherServlet - Completed initialization in 1 ms
10:32:53.633 [http-nio-8082-exec-1] INFO  c.b.O.MyOncePerRequestFilter - Inside OncePer Request Filter originated by request /greeting
10:32:53.663 [http-nio-8082-exec-2] INFO  c.b.O.MyOncePerRequestFilter - Inside OncePer Request Filter originated by request /greeting
```

我们可以从上面的输出中看到，我们的过滤器被调用了两次——第一次被容器线程调用，然后被另一个线程调用。

## 7.结论

在本文中，我们研究了`OncePerRequestFilter`，它解决了什么问题，以及如何通过一些实际例子来实现它。

和往常一样，完整的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220525122016/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-basic-customization-2)