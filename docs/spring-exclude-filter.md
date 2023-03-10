# 在 Spring Web 应用程序中排除过滤器的 URL

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-exclude-filter>

## 1.概观

大多数 web 应用程序都有一个执行请求日志、验证或认证等操作的用例。此外，这样的**任务通常由一组 HTTP 端点**共享。

好消息是 Spring web 框架提供了一个[过滤机制](https://web.archive.org/web/20221115050032/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/filter/package-summary.html)来实现这个目的。

在本教程中，我们将学习对于一组给定的 URL，如何从执行中包含或排除一个**过滤器样式的任务。**

## 2.过滤特定的 URL

假设我们的 web 应用程序需要记录一些关于它的请求的信息，比如它们的路径和内容类型。一种方法是创建日志过滤器。

### 2.1.日志过滤器

首先，让我们在一个`LogFilter`类中创建日志过滤器，该类扩展了 **[`OncePerRequestFilter`](https://web.archive.org/web/20221115050032/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/filter/OncePerRequestFilter.html) 类并实现了 [`doFilterInternal`](https://web.archive.org/web/20221115050032/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/filter/OncePerRequestFilter.html#doFilterInternal-javax.servlet.http.HttpServletRequest-javax.servlet.http.HttpServletResponse-javax.servlet.FilterChain-) 方法:**

```java
@Override
protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response,
  FilterChain filterChain) throws ServletException, IOException {
    String path = request.getRequestURI();
    String contentType = request.getContentType();
    logger.info("Request URL path : {}, Request content type: {}", path, contentType);
    filterChain.doFilter(request, response);
}
```

### 2.1.规则过滤器

假设我们需要只为选择的 URL 模式执行日志记录任务，即`/health`、`/faq/*.` 为此，我们将使用`FilterRegistrationBean` 注册我们的日志过滤器，以便它只匹配所需的 URL 模式:

```java
@Bean
public FilterRegistrationBean<LogFilter> logFilter() {
    FilterRegistrationBean<LogFilter> registrationBean = new FilterRegistrationBean<>();
    registrationBean.setFilter(new LogFilter());
    registrationBean.addUrlPatterns("/health","/faq/*");
    return registrationBean;
}
```

### 2.2.排除过滤器

如果我们想在执行日志记录任务时排除 URL，我们可以通过两种方式轻松实现:

*   对于新的 URL，请确保它与过滤器使用的 URL 模式不匹配
*   对于以前启用了日志记录的旧 URL，我们可以修改 URL 模式以排除此 URL

## 3.过滤所有可能的 URL

我们用最少的努力轻松满足了之前在`LogFilter`中包含 URL 的用例。然而，如果`Filter`使用通配符(*)来匹配所有可能的 URL 模式，那么**就变得更加复杂了。**

在这种情况下，我们需要自己编写包含和排除逻辑。

### 3.1.自定义`Filter`

客户端可以通过使用请求头向服务器发送有用的信息。假设我们的 web 应用程序目前只在美国运行，这意味着我们不想处理来自其他国家的请求。

让我们进一步想象一下，我们的 web 应用程序通过一个`X-Country-Code`请求头来指示地区。因此，每个请求都带有这些信息，我们有一个使用过滤器的明确案例。

让我们实现一个`Filter` 来检查头部，拒绝不符合我们条件的请求:

```java
@Override
protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response,
      FilterChain filterChain) throws ServletException, IOException {

    String countryCode = request.getHeader("X-Country-Code");
    if (!"US".equals(countryCode)) {
        response.sendError(HttpStatus.BAD_REQUEST.value(), "Invalid Locale");
        return;
    }

    filterChain.doFilter(request, response);
}
```

### 3.2.`Filter `注册

首先，l et 使用 **星号(*)通配符来注册我们的过滤器** 以匹配所有可能的 URL 模式:

```java
@Bean
public FilterRegistrationBean<HeaderValidatorFilter> headerValidatorFilter() {
    FilterRegistrationBean<HeaderValidatorFilter> registrationBean = new FilterRegistrationBean<>();
    registrationBean.setFilter(new HeaderValidatorFilter());
    registrationBean.addUrlPatterns("*");
    return registrationBean;
} 
```

在稍后的时间点，我们可以排除执行验证区域设置请求头信息的任务不需要的 URL 模式。

## 4.URL 排除

在本节中，我们将学习如何为我们的客户`Filter`排除 URL。

### 4.1.天真的策略

让我们再想象一下，我们在 */health* 有一个 web 路由，可以用来对应用程序进行乒乓式健康检查。

到目前为止，所有的请求都会触发我们的过滤器。正如我们所猜测的，当涉及到我们的健康检查时，这是一项开销。

因此，让我们简化我们的 */health* 请求，将它们从过滤器的主体中排除:

```java
@Override
protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response,
  FilterChain filterChain) throws ServletException, IOException {
    String path = request.getRequestURI();
    if ("/health".equals(path)) {
    	filterChain.doFilter(request, response);
    	return;
    }

    String countryCode = request.getHeader("X-Country-Code");
    // ... same as before
}
```

**我们必须注意，在`doFilter`方法中添加这个定制逻辑会在 */health* 端点和我们的过滤器**之间引入耦合。因此，它不是最佳的，因为如果我们改变健康检查端点而没有在`doFilter`方法中做出相应的改变，我们可能会破坏过滤逻辑。

### 4.2.使用`shouldNotFilter`方法

使用前面的方法，我们在 URL 排除和过滤器的任务执行逻辑之间引入了紧密耦合。人们可能无意中在一个部分引入了一个 bug，而打算对另一个部分进行修改。

相反，我们可以通过覆盖 [`shouldNotFilter`](https://web.archive.org/web/20221115050032/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/filter/OncePerRequestFilter.html#shouldNotFilter-javax.servlet.http.HttpServletRequest-) 方法来隔离两组逻辑:

```java
@Override
protected boolean shouldNotFilter(HttpServletRequest request)
  throws ServletException {
    String path = request.getRequestURI();
    return "/health".equals(path);
}
```

因此，`doInternalFilter()`方法遵循[单一责任原则](https://web.archive.org/web/20221115050032/https://baeldung.com/solid-principles#s):

```java
@Override
protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response,
  FilterChain filterChain) throws ServletException, IOException {
    String countryCode = request.getHeader("X-Country-Code");
    // ... same as before
}
```

## 5.结论

在本教程中，我们探讨了如何在两个用例中从 Spring Boot web 应用程序的 servlet 过滤器中排除 URL 模式，即日志记录和请求头验证。

此外，我们了解到对于使用*通配符匹配所有可能的 URL 模式的过滤器来说，**很难排除一组特定的 URL。**

和往常一样，该教程的完整源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221115050032/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-web-url)