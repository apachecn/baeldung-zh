# 春季委托过滤器代理的概述和需求

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-delegating-filter-proxy>

## 1。概述

`DelegatingFilterProxy `是一个 servlet 过滤器，它允许将控制权传递给能够访问 Spring 应用程序上下文的`Filter`类。Spring Security 非常依赖这种技术。

在本教程中，我们将详细介绍它。

## 2。`DelegatingFilterProxy`

`DelegatingFilterProxy`的 Javadoc 声明它是一个

> 标准 Servlet 过滤器的代理，委托给实现过滤器接口的 Spring 管理的 bean。

当使用 servlet 过滤器时，我们显然需要在 Java-config 中将它们声明为`filter-class`或`web.xml`，否则，servlet 容器会忽略它们。Spring 的`DelegatingFilterProxy`提供了`web.xml`和应用程序上下文之间的链接。

### 2.1。`DelegatingFilterProxy` 内功

让我们看看`DelegatingFilterProxy`是如何将控制权转移给我们的 Spring bean 的。

在初始化期间，`DelegatingFilterProxy`获取`filter-name`并从 Spring 应用程序上下文中检索具有该名称的 bean。该 bean 必须是`javax.Servlet.Filter, `类型，即“普通”servlet 过滤器。传入的请求将被传递到这个过滤器 bean。

**简而言之，`DelegatingFilterProxy's` `doFilter()`方法将所有调用委托给一个 Spring bean，使我们能够在我们的过滤器 bean 中使用所有 Spring 特性。**

如果我们使用基于 Java 的配置，我们在`ApplicationInitializer`中的过滤器注册将被定义为:

```java
@Override
protected javax.servlet.Filter[] getServletFilters() {
    DelegatingFilterProxy delegateFilterProxy = new DelegatingFilterProxy();
    delegateFilterProxy.setTargetBeanName("applicationFilter");
    return new Filter[]{delegateFilterProxy};
}
```

如果我们使用 XML，那么，在`web.xml`文件中:

```java
<filter>
    <filter-name>applicationFilter</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
</filter>
```

这意味着任何请求都可以通过名为`applicationFilter`的 Spring bean 过滤器。

### 2.2。`DelegatingFilterProxy`需要为

`DelegatingFilterProxy`是 Spring 的 Web 模块中的一个类。它提供了使 HTTP 调用在到达实际目的地之前通过过滤器的功能。在`DelegatingFilterProxy,` 的帮助下，实现`javax.Servlet.Filter `接口的类可以连接到过滤器链中。

例如，Spring Security 利用`DelegatingFilterProxy` 来利用 Spring 的依赖注入特性和安全过滤器的生命周期接口。

`DelegatingFilterProxy`还通过在 Spring 的应用程序上下文或`web.xml.`中提供配置，利用根据请求 URI 路径调用特定或多个过滤器

## 3。创建自定义过滤器

如上所述，`DelegatingFilterProxy`本身是一个 servlet 过滤器，它委托给一个实现了`Filter`接口的特定 Spring 管理的 bean。

在接下来的几节中，我们将创建一个定制过滤器，并使用基于 Java 和 XML 的配置来配置它。

### 3.1。过滤器等级

我们将创建一个简单的过滤器，在请求进一步处理之前记录请求信息。

让我们首先创建一个自定义过滤器类:

```java
@Component("loggingFilter")
public class CustomFilter implements Filter {

    private static Logger LOGGER = LoggerFactory.getLogger(CustomFilter.class);

    @Override
    public void init(FilterConfig config) throws ServletException {
        // initialize something
    }

    @Override
    public void doFilter(
      ServletRequest request, ServletResponse response, 
      FilterChain chain) throws IOException, ServletException {

        HttpServletRequest req = (HttpServletRequest) request;
        LOGGER.info("Request Info : " + req);
        chain.doFilter(request, response);
    }

    @Override
    public void destroy() {
        // cleanup code, if necessary
    }
} 
```

`CustomFilter` 实现`javax.Servlet.Filter`。这个类有一个`@Component`注释，用于在应用程序上下文中注册为 Spring bean。这样，`DelegatingFilterProxy`类可以在初始化过滤器链时找到我们的过滤器类。

**注意，Spring bean 的名称必须与在`ApplicationInitializer`类中注册定制过滤器时提供的`filter-name`中的值相同，或者与随后**中的`web.xml`中的值相同，因为`the DelegatingFilterProxy`类将在应用程序上下文中查找具有完全相同名称的过滤器 bean。

如果找不到这个名称的 bean，它将在应用程序启动时引发一个异常。

### 3.2。通过 Java 配置配置过滤器

要使用 Java 配置注册自定义过滤器，我们需要覆盖`AbstractAnnotationConfigDispatcherServletInitializer`的`getServletFilters()`方法:

```java
public class ApplicationInitializer 
  extends AbstractAnnotationConfigDispatcherServletInitializer {
    // some other methods here

    @Override
    protected javax.servlet.Filter[] getServletFilters() {
        DelegatingFilterProxy delegateFilterProxy = new DelegatingFilterProxy();
        delegateFilterProxy.setTargetBeanName("loggingFilter");
        return new Filter[]{delegateFilterProxy};
    }
}
```

### 3.3。通过`web.xml` 配置过滤器

让我们看看`web.xml`中的滤波器配置是怎样的:

```java
<filter>
    <filter-name>loggingFilter</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
</filter>
<filter-mapping>
    <filter-name>loggingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

`filter-class`参数属于`DelegatingFilterProxy`类型，而不是我们创建的过滤器类。如果我们运行这段代码并点击任何 URL，那么`CustomFilter`的`doFilter()`方法将被执行并在日志文件中显示请求信息细节。

## 4。结论

在本文中，我们已经介绍了`DelegatingFilterProxy` 如何工作以及如何使用它。

Spring Security 广泛使用`DelegatingFilterProxy`来保护 web API 调用和资源免受未授权的访问。

GitHub 上的[提供了源代码。](https://web.archive.org/web/20221208143917/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-core)