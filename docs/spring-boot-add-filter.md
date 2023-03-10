# 如何定义 Spring Boot 滤波器？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-add-filter>

## 1.概观

在这个快速教程中，我们将探索如何在 Spring Boot 的帮助下定义自定义过滤器并指定它们的调用顺序。

## 延伸阅读:

## [找到注册的 Spring 安全过滤器](/web/20220626081517/https://www.baeldung.com/spring-security-registered-filters)

Learn how to find all the registered Spring Security filters in an application.[Read more](/web/20220626081517/https://www.baeldung.com/spring-security-registered-filters) →

## [配置 Spring Boot 网络应用](/web/20220626081517/https://www.baeldung.com/spring-boot-application-configuration)

Some of the more useful configs for a Spring Boot application.[Read more](/web/20220626081517/https://www.baeldung.com/spring-boot-application-configuration) →

## 2.定义过滤器和调用顺序

让我们从创建两个过滤器开始:

1.  `TransactionFilter`–开始并提交事务
2.  `RequestResponseLoggingFilter`–记录请求和响应

为了创建一个过滤器，我们只需要实现`Filter`接口:

```java
@Component
@Order(1)
public class TransactionFilter implements Filter {

    @Override
    public void doFilter(
      ServletRequest request, 
      ServletResponse response, 
      FilterChain chain) throws IOException, ServletException {

        HttpServletRequest req = (HttpServletRequest) request;
        LOG.info(
          "Starting a transaction for req : {}", 
          req.getRequestURI());

        chain.doFilter(request, response);
        LOG.info(
          "Committing a transaction for req : {}", 
          req.getRequestURI());
    }

    // other methods 
} 
```

```java
@Component
@Order(2)
public class RequestResponseLoggingFilter implements Filter {

    @Override
    public void doFilter(
      ServletRequest request, 
      ServletResponse response, 
      FilterChain chain) throws IOException, ServletException {

        HttpServletRequest req = (HttpServletRequest) request;
        HttpServletResponse res = (HttpServletResponse) response;
        LOG.info(
          "Logging Request  {} : {}", req.getMethod(), 
          req.getRequestURI());
        chain.doFilter(request, response);
        LOG.info(
          "Logging Response :{}", 
          res.getContentType());
    }

    // other methods
} 
```

为了让 Spring 识别过滤器，我们需要用`@Component`注释将其定义为 bean。

**此外，为了让过滤器以正确的顺序启动，我们需要使用`@Order`注释。**

### 2.1。使用 URL 模式过滤

在上面的例子中，我们的过滤器默认为应用程序中的所有 URL 注册。然而，我们有时可能希望过滤器只应用于某些 URL 模式。

在这种情况下，我们必须从过滤器类定义中移除`@Component`注释，并使用`**FilterRegistrationBean**:`**注册过滤器**

```java
@Bean
public FilterRegistrationBean<RequestResponseLoggingFilter> loggingFilter(){
    FilterRegistrationBean<RequestResponseLoggingFilter> registrationBean 
      = new FilterRegistrationBean<>();

    registrationBean.setFilter(new RequestResponseLoggingFilter());
    registrationBean.addUrlPatterns("/users/*");
    registrationBean.setOrder(2);

    return registrationBean;    
}
```

注意，在这种情况下，我们需要使用一个`setOrder()`方法显式地设置顺序。

现在，过滤器将只适用于匹配`/users/*`模式的路径。

**要为过滤器设置 URL 模式，我们可以使用`addUrlPatterns()`或`setUrlPatterns()`方法。**

## 3.一个简单的例子

现在让我们创建一个简单的端点，并向它发送一个 HTTP 请求:

```java
@RestController
@RequestMapping("/users")
public class UserController {

    @GetMapping()
    public List<User> getAllUsers() {
        // ...
    }
}
```

点击此 API 的应用程序日志如下:

```java
23:54:38 INFO  com.spring.demo.TransactionFilter - Starting Transaction for req :/users
23:54:38 INFO  c.s.d.RequestResponseLoggingFilter - Logging Request  GET : /users
...
23:54:38 INFO  c.s.d.RequestResponseLoggingFilter - Logging Response :application/json;charset=UTF-8
23:54:38 INFO  com.spring.demo.TransactionFilter - Committing Transaction for req :/users
```

这确认了过滤器是以期望的顺序被调用的。

## 4.结论

在这篇简短的文章中，我们总结了如何在 Spring Boot webapp 中定义自定义过滤器。

和往常一样，代码片段可以在 GitHub 上找到[。](https://web.archive.org/web/20220626081517/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-basic-customization)