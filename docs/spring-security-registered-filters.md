# 找到注册的 Spring 安全过滤器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-registered-filters>

## 1。概述

Spring 安全性基于一系列 servlet 过滤器。每个过滤器都有特定的职责，根据配置的不同，可以添加或删除过滤器。

在本教程中，**我们将讨论找到注册的 Spring 安全过滤器**的不同方法。

## 2。安全调试

首先，我们将启用安全调试，它将记录每个请求的详细安全信息。

我们可以使用`debug`属性启用安全调试:

```java
@EnableWebSecurity(debug = true)
```

这样，当我们向服务器发送请求时，所有的请求信息都会被记录下来。

我们还将能够看到整个安全过滤器链:

```java
Security filter chain: [
  WebAsyncManagerIntegrationFilter
  SecurityContextPersistenceFilter
  HeaderWriterFilter
  LogoutFilter
  UsernamePasswordAuthenticationFilter
  // ...
]
```

## 3。记录日志

接下来，我们将通过为`FilterChainProxy`启用日志来找到我们的安全过滤器。

我们可以通过向`application.properties`添加下面一行来启用日志记录:

```java
logging.level.org.springframework.security.web.FilterChainProxy=DEBUG
```

以下是相关日志:

```java
DEBUG o.s.security.web.FilterChainProxy - /foos/1 at position 1 of 12 in additional filter chain; firing Filter: 'WebAsyncManagerIntegrationFilter'
DEBUG o.s.security.web.FilterChainProxy - /foos/1 at position 2 of 12 in additional filter chain; firing Filter: 'SecurityContextPersistenceFilter'
DEBUG o.s.security.web.FilterChainProxy - /foos/1 at position 3 of 12 in additional filter chain; firing Filter: 'HeaderWriterFilter'
DEBUG o.s.security.web.FilterChainProxy - /foos/1 at position 4 of 12 in additional filter chain; firing Filter: 'LogoutFilter'
DEBUG o.s.security.web.FilterChainProxy - /foos/1 at position 5 of 12 in additional filter chain; firing Filter: 'UsernamePasswordAuthenticationFilter'
...
```

## 4。以编程方式获取过滤器

现在，我们将了解如何以编程方式获取注册的安全过滤器。

**我们将使用`FilterChainProxy`来获取安全过滤器。**

首先，让我们自动连接`springSecurityFilterChain` bean:

```java
@Autowired
@Qualifier("springSecurityFilterChain")
private Filter springSecurityFilterChain;
```

这里，我们使用了名为`springSecurityFilterChain` 的`@Qualifier`和类型`Filter`而不是`FilterChainProxy. `，这是因为`[WebSecurityConfiguration](https://web.archive.org/web/20220617075715/https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/config/annotation/web/configuration/WebSecurityConfiguration.html#springSecurityFilterChain--),`中的`springSecurityFilterChain()`方法创建了弹簧安全过滤器链，返回类型`Filter`而不是`FilterChainProxy.`

接下来，我们将这个对象转换为`FilterChainProxy`并调用`getFilterChains()`方法:

```java
public void getFilters() {
    FilterChainProxy filterChainProxy = (FilterChainProxy) springSecurityFilterChain;
    List<SecurityFilterChain> list = filterChainProxy.getFilterChains();
    list.stream()
      .flatMap(chain -> chain.getFilters().stream()) 
      .forEach(filter -> System.out.println(filter.getClass()));
}
```

这是一个输出示例:

```java
class org.springframework.security.web.context.request.async.WebAsyncManagerIntegrationFilter
class org.springframework.security.web.context.SecurityContextPersistenceFilter
class org.springframework.security.web.header.HeaderWriterFilter
class org.springframework.security.web.authentication.logout.LogoutFilter
class org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter
...
```

请注意，自 Spring Security 3.1 以来， **`FilterChainProxy`是使用一列`SecurityFilterChain.`** 配置的，然而，大多数应用程序只需要一个`SecurityFilterChain.`

## 5。重要的 Spring 安全过滤器

最后，让我们看看一些重要的安全过滤器:

*   `UsernamePasswordAuthenticationFilter`:流程认证，默认响应“/登录”URL
*   `AnonymousAuthenticationFilter`:当 SecurityContextHolder 中没有认证对象时，它创建一个匿名认证对象并放在那里
*   当访问被拒绝时引发异常
*   `ExceptionTranslationFilter`:捕捉 Spring 安全异常

## 6。结论

在这篇快速文章中，我们探索了如何通过编程和使用日志来找到注册的 Spring 安全过滤器。

和往常一样，源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220617075715/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-web-boot-1)