# 处理 Spring 安全异常

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-exceptions>

 ![](img/cb4a8399e8dd1fed1bd69af406c88e35.png)

It’s just plain hard to get true, **real-time visibility into a running auth flow.**

Parts of the process can be completely hidden from us; if the complete authorization process requires a redirect from a remote OAuth production server, then every debugging effort must go through the production server.

It’s practically unfeasible to debug this locally. There’s no way to reproduce the exact state and no way to inspect what is actually happening under the hood. Not ideal.

Knowing these types of challenges, we built Lightrun - a real-time production debugging tool - to allow you to understand complicated flows with code-level information. Add logs, take snapshots (virtual breakpoints), and instrument metrics without a remote debugger, without stopping the running service, and, most importantly - **in real-time and without side effects**.

**Learn more with this 5-minute tutorial** focused on debugging these kinds of scenarios using Lightrun:

[>> Debugging Authentication and Authorization Using Lightrun](/web/20220524051832/https://www.baeldung.com/lightrun-n-security)

## 1.概观

在这篇文章中，我们将看看**如何处理由我们的 Spring 安全[资源服务器](/web/20220524051832/https://www.baeldung.com/spring-security-oauth-resource-server)** 产生的 Spring 安全异常。为此，我们还将使用一个实例来解释所有必要的配置。首先，我们来简单介绍一下春季安全。

## 2.春天安全

Spring Security 是 Spring 项目的一部分。它试图将 Spring 项目上用户访问控制的所有功能组合在一起。访问控制允许限制应用程序中一组给定用户或角色可以执行的选项。在这个方向上， **Spring Security 控制对业务逻辑的调用，或者限制 HTTP 请求对某些 URL 的访问**。记住这一点，我们必须通过告诉 Spring Security 安全层应该如何表现来配置应用程序。

在我们的例子中，我们将关注异常处理程序的配置。 **Spring Security 提供了三个不同的接口来实现这个目的并控制产生的事件:**

*   认证成功处理程序
*   认证失败处理程序
*   拒绝访问处理程序

首先，让我们仔细看看配置。

## 3.安全配置

首先，我们的配置类必须扩展`WebSecurityConfigurerAdapter`类。**这将负责管理应用程序的所有安全配置。**所以，在这里我们必须介绍我们的训练员。

一方面，我们将定义所需的配置:

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.csrf()
      .disable()
      .httpBasic().disable()
      .authorizeRequests()
      .antMatchers("/login").permitAll()
      .antMatchers("/customError").permitAll()
      .antMatchers("/access-denied").permitAll()
      .antMatchers("/secured").hasRole("ADMIN")
      .anyRequest().authenticated()
      .and()
      .formLogin()
      .failureHandler(authenticationFailureHandler())
      .successHandler(authenticationSuccessHandler())
      .and()
      .exceptionHandling()
      .accessDeniedHandler(accessDeniedHandler())
      .and()
      .logout();
    }
}
```

有趣的是，重定向 URL，比如`“/login”`、`“/customError”,`和`“/access-denied”`，不需要任何类型的限制就可以访问它们。所以，我们将它们标注为`permitAll()`。

另一方面，我们必须定义定义我们可以处理的异常类型的 Beans:

```java
@Bean
public AuthenticationFailureHandler authenticationFailureHandler() {
    return new CustomAuthenticationFailureHandler();
} 

@Bean
public AuthenticationSuccessHandler authenticationSuccessHandler() {
   return new CustomAuthenticationSuccessHandler();
}

@Bean
public AccessDeniedHandler accessDeniedHandler() {
   return new CustomAccessDeniedHandler();
}
```

由于 [`AuthenticationSuccessHandler`](/web/20220524051832/https://www.baeldung.com/spring_redirect_after_login) 处理快乐路径，我们将为异常情况定义剩下的两个 beans。**这两个处理程序** **是我们现在必须根据我们的需求进行调整和实现的**。因此，让我们继续实现它们中的每一个。

## 4.认证失败处理程序

一方面，我们有了`AuthenticationFailureHandler`界面。负责**管理用户无法登录**时产生的异常。这个接口为我们提供了定制处理程序逻辑的`onAuthenticationFailure()`方法。**它将在登录失败时被 Spring Security 调用**。记住这一点，让我们定义异常处理程序，以便在登录失败时将我们重定向到错误页面:

```java
public class CustomAuthenticationFailureHandler implements AuthenticationFailureHandler {

    @Override
    public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response, AuthenticationException exception) 
      throws IOException {
        response.sendRedirect("/customError");
    }
}
```

## 5.拒绝访问处理程序

另一方面，当未经授权的用户试图访问安全或受保护的页面时， **Spring Security 将抛出一个拒绝访问异常**。Spring Security 提供了一个默认的 403 拒绝访问页面，我们可以自定义它。这是由`AccessDeniedHandler`接口管理的。**此外，它还提供了`handle()`方法，用于在将用户重定向到 403 页面**之前自定义逻辑:

```java
public class CustomAccessDeniedHandler implements AccessDeniedHandler {

    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response, AccessDeniedException exc) throws IOException {
        response.sendRedirect("/access-denied");
    }
}
```

## 6.结论

在这篇简短的文章中，**我们学习了如何处理** **Spring 安全异常，以及如何通过创建和定制我们的类**来控制它们。此外，我们还创建了一个全功能示例，帮助我们理解所解释的概念。

GitHub 上有这篇文章的完整源代码。