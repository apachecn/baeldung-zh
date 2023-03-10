# Spring MVC 中的 handler 接受器与过滤器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-mvc-handlerinterceptor-vs-filter>

## 1.概观

在本文中，我们将比较 Java servlet `Filter`和 Spring MVC `[HandlerInterceptor](/web/20221220114603/https://www.baeldung.com/spring-mvc-handlerinterceptor),`以及何时一个可能优于另一个。

## 2.`Filter`年代

过滤器是网络服务器的一部分，而不是 Spring 框架的一部分。对于传入的请求，**我们可以使用过滤器来操纵甚至阻止请求到达任何 [servlet](/web/20221220114603/https://www.baeldung.com/java-servlets-containers-intro)** 。反之亦然，我们也可以阻止响应到达客户端。

Spring Security 是使用过滤器进行认证和授权的一个很好的例子。要配置 Spring 安全性，我们只需添加一个过滤器，即 [`DelegatingFilterProxy`](/web/20221220114603/https://www.baeldung.com/spring-delegating-filter-proxy) 。Spring Security 可以拦截所有进出的流量。这也是为什么 Spring Security 可以在 [Spring MVC](/web/20221220114603/https://www.baeldung.com/spring-mvc) 之外使用的原因。

### 2.1.创建一个`Filter`

为了创建一个过滤器，首先，我们创建一个实现`[javax.servlet.Filter](https://web.archive.org/web/20221220114603/https://docs.oracle.com/javaee/7/api/javax/servlet/Filter.html) interface`的类:

```java
@Component
public class LogFilter implements Filter {

    private Logger logger = LoggerFactory.getLogger(LogFilter.class);

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) 
      throws IOException, ServletException {
        logger.info("Hello from: " + request.getLocalAddr());
        chain.doFilter(request, response);
    }

}
```

接下来，我们重写`doFilter`方法，在这里我们可以访问或操作`ServletRequest`、`ServletResponse`或`FilterChain`对象。我们可以使用`FilterChain`对象来允许或阻止请求。

最后，我们通过用`@Component.` 对其进行注释，将`Filter`添加到 Spring 上下文中，Spring 将完成剩下的工作。

## 3.`HandlerInterceptor`年代

**[`HandlerInterceptor`](/web/20221220114603/https://www.baeldung.com/spring-mvc-handlerinterceptor)是 Spring MVC 框架的一部分，位于`DispatcherServlet`和我们的`Controller`**之间，我们可以在请求到达我们的控制器之前，以及视图呈现之前和之后拦截请求。

### 3.1.创建一个`HandlerInterceptor`

为了创建一个`HandlerInterceptor`，我们创建了一个实现`[org.springframework.web.servlet.HandlerInterceptor](https://web.archive.org/web/20221220114603/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/servlet/HandlerInterceptor.html)`接口的类。这为我们提供了覆盖三种方法的选项:

*   `preHandle()`–在调用目标处理程序之前执行
*   `postHandle()`–在目标处理程序之后、`DispatcherServlet`呈现视图之前执行
*   `afterCompletion() –` 请求处理完成后回调并查看渲染

让我们将日志记录添加到测试拦截器的三个方法中:

```java
public class LogInterceptor implements HandlerInterceptor {

    private Logger logger = LoggerFactory.getLogger(LogInterceptor.class);

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) 
      throws Exception {
        logger.info("preHandle");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) 
      throws Exception {
        logger.info("postHandle");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) 
      throws Exception {
        logger.info("afterCompletion");
    }

}
```

## 4.主要差异和使用案例

让我们看一个图表，显示`Filter`和`HandlerInterceptor`在请求/响应流中的位置:

[![filters vs interceptors](img/d192cbf699c8e05c9b576d48d2a8a11e.png)](/web/20221220114603/https://www.baeldung.com/wp-content/uploads/2021/05/filters_vs_interceptors.jpg)

**过滤器在请求到达`DispatcherServlet`之前拦截请求，使其成为粗粒度任务**的理想选择，例如:

*   证明
*   日志记录和审核
*   图像和数据压缩
*   我们希望从 Spring MVC 中分离出来的任何功能

另一方面， **`HandlerIntercepor` s 拦截`DispatcherServlet`和我们的`Controller`**s 之间的请求，这是在 Spring MVC 框架内完成的，提供对`Handler`和`ModelAndView`对象的访问。这减少了重复，并允许更细粒度的功能，例如:

*   处理横切关注点，如应用程序日志记录
*   详细的授权检查
*   操纵 Spring 上下文或模型

## 5.结论

在本文中，我们讨论了 a `Filter`和`HandlerInterceptor`之间的区别。

**关键的一点是，使用`Filter` s，我们可以在请求到达我们的控制器和 Spring MVC 之外之前对其进行处理。**否则，`HandlerInterceptor`对于特定应用的横切关注点来说是一个很好的地方。通过提供对目标`Handler`和`ModelAndView`对象的访问，我们有了更细粒度的控制。

所有这些例子和代码片段的实现都可以在 GitHub 上找到[。](https://web.archive.org/web/20221220114603/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-mvc-3)