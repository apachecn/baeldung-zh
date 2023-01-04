# spring Security–登录后重定向到以前的 URL

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-redirect-login>

## 1。概述

本文将关注**如何在用户登录**后将用户重定向回最初请求的 URL。

之前，我们已经看到了[如何针对不同类型的用户使用`Spring Security`](/web/20221004133411/https://www.baeldung.com/spring_redirect_after_login) 登录后重定向到不同的页面，以及使用`Spring MVC` 的各种类型的[重定向。](/web/20221004133411/https://www.baeldung.com/spring-redirect-and-forward)

文章基于 [Spring 安全登录](/web/20221004133411/https://www.baeldung.com/spring-security-login)教程之上。

## 2。惯例

登录后实现重定向逻辑的最常见方法是:

*   使用`HTTP Referer`割台
*   在会话中保存原始请求
*   将原始 URL 附加到重定向的登录 URL

**使用`HTTP Referer`头**是一种简单的方法，因为大多数浏览器和`HTTP`客户端会自动设置`Referer`。但是，由于`Referer`是可伪造的，并且依赖于客户端实现，所以一般不建议使用`HTTP Referer`头来实现重定向。

**在会话**中保存原始请求是实现这种重定向的一种安全可靠的方式。除了原始 URL，我们还可以在会话中存储原始请求属性和任何自定义属性。

在 SSO 实现中，通常会看到将原始 URL 附加到重定向的登录 URL 上。当通过 SSO 服务进行身份验证时，用户将被重定向到最初请求的页面，并附加 URL。我们必须确保附加的 URL 编码正确。

另一个类似的实现是将原始请求 URL 放在登录表单内的隐藏字段中。但是这并不比使用`HTTP Referer`更好

在 Spring Security 中，前两种方法是本机支持的。

必须注意的是**对于 Spring Boot 的新版本，默认情况下，Spring Security 能够在登录到我们试图访问的安全资源**后进行重定向。如果我们需要总是重定向到一个特定的 URL，我们可以通过一个特定的`HttpSecurity`配置来强制重定向。

## 3。`AuthenticationSuccessHandler`

在基于表单的认证中，重定向在登录后立即发生，这在`Spring Security`中的 [`AuthenticationSuccessHandler`](https://web.archive.org/web/20221004133411/https://github.com/spring-projects/spring-security/blob/master/web/src/main/java/org/springframework/security/web/authentication/AuthenticationSuccessHandler.java) 实例中处理。

系统提供了三种默认实现: [`SimpleUrlAuthenticationSuccessHandler`](https://web.archive.org/web/20221004133411/https://github.com/spring-projects/spring-security/blob/master/web/src/main/java/org/springframework/security/web/authentication/SimpleUrlAuthenticationSuccessHandler.java) 、 [`SavedRequestAwareAuthenticationSuccessHandler`](https://web.archive.org/web/20221004133411/https://github.com/spring-projects/spring-security/blob/master/web/src/main/java/org/springframework/security/web/authentication/SavedRequestAwareAuthenticationSuccessHandler.java) 和 [`ForwardAuthenticationSuccessHandler`](https://web.archive.org/web/20221004133411/https://github.com/spring-projects/spring-security/blob/master/web/src/main/java/org/springframework/security/web/authentication/ForwardAuthenticationSuccessHandler.java) 。我们将关注前两个实现。

### 3.1。 `SavedRequestAwareAuthenticationSuccessHandler`

`SavedRequestAwareAuthenticationSuccessHandler`使用保存在会话中的请求。成功登录后，用户将被重定向到原始请求中保存的 URL。

对于表单登录，`SavedRequestAwareAuthenticationSuccessHandler`被用作默认`AuthenticationSuccessHandler`。

```
@Configuration
@EnableWebSecurity
public class RedirectionSecurityConfig extends WebSecurityConfigurerAdapter {

    //...

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
          .authorizeRequests()
          .antMatchers("/login*")
          .permitAll()
          .anyRequest()
          .authenticated()
          .and()
          .formLogin();
    }

}
```

等效的 XML 应该是:

```
<http>
    <intercept-url pattern="/login" access="permitAll"/>
    <intercept-url pattern="/**" access="isAuthenticated()"/>
    <form-login />
</http>
```

假设我们在位置“/secured”有一个安全的资源。第一次访问资源时，我们会被重定向到登录页面；填写凭据并发布登录表单后，我们将被重定向回最初请求的资源位置:

```
@Test
public void givenAccessSecuredResource_whenAuthenticated_thenRedirectedBack() 
  throws Exception {

    MockHttpServletRequestBuilder securedResourceAccess = get("/secured");
    MvcResult unauthenticatedResult = mvc
      .perform(securedResourceAccess)
      .andExpect(status().is3xxRedirection())
      .andReturn();

    MockHttpSession session = (MockHttpSession) unauthenticatedResult
      .getRequest()
      .getSession();
    String loginUrl = unauthenticatedResult
      .getResponse()
      .getRedirectedUrl();
    mvc
      .perform(post(loginUrl)
        .param("username", userDetails.getUsername())
        .param("password", userDetails.getPassword())
        .session(session)
        .with(csrf()))
      .andExpect(status().is3xxRedirection())
      .andExpect(redirectedUrlPattern("**/secured"))
      .andReturn();

    mvc
      .perform(securedResourceAccess.session(session))
      .andExpect(status().isOk());
}
```

### 3.2。`SimpleUrlAuthenticationSuccessHandler`

与 `SavedRequestAwareAuthenticationSuccessHandler`相比，`SimpleUrlAuthenticationSuccessHandler`在重定向决策上给了我们更多的选择。

我们可以通过`setUserReferer(true)`启用基于引用的重定向:

```
public class RefererRedirectionAuthenticationSuccessHandler 
  extends SimpleUrlAuthenticationSuccessHandler
  implements AuthenticationSuccessHandler {

    public RefererRedirectionAuthenticationSuccessHandler() {
        super();
        setUseReferer(true);
    }

}
```

然后将其作为`RedirectionSecurityConfig`中的`AuthenticationSuccessHandler`:

```
@Override
protected void configure(HttpSecurity http) throws Exception {
    http
      .authorizeRequests()
      .antMatchers("/login*")
      .permitAll()
      .anyRequest()
      .authenticated()
      .and()
      .formLogin()
      .successHandler(new RefererAuthenticationSuccessHandler());
}
```

对于 XML 配置:

```
<http>
    <intercept-url pattern="/login" access="permitAll"/>
    <intercept-url pattern="/**" access="isAuthenticated()"/>
    <form-login authentication-success-handler-ref="refererHandler" />
</http>

<beans:bean 
  class="RefererRedirectionAuthenticationSuccessHandler" 
  name="refererHandler"/>
```

### 3.3。引擎盖下

`Spring Security`中这些易于使用的功能并没有什么神奇之处。当请求受保护的资源时，该请求将被一系列不同的过滤器过滤。将检查身份验证主体和权限。如果请求会话还没有被认证，将抛出 [`AuthenticationException`](https://web.archive.org/web/20221004133411/https://github.com/spring-projects/spring-security/blob/master/core/src/main/java/org/springframework/security/core/AuthenticationException.java) 。

`AuthenticationException` 将在`[ExceptionTranslationFilter](https://web.archive.org/web/20221004133411/https://github.com/spring-projects/spring-security/blob/master/web/src/main/java/org/springframework/security/web/access/ExceptionTranslationFilter.java),` 中被捕获，其中认证过程将开始，导致重定向到登录页面。

```
public class ExceptionTranslationFilter extends GenericFilterBean {

    //...

    public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
      throws IOException, ServletException {
        //...

        handleSpringSecurityException(request, response, chain, ase);

        //...
    }

    private void handleSpringSecurityException(HttpServletRequest request,
      HttpServletResponse response, FilterChain chain, RuntimeException exception)
      throws IOException, ServletException {

        if (exception instanceof AuthenticationException) {

            sendStartAuthentication(request, response, chain,
              (AuthenticationException) exception);

        }

        //...
    }

    protected void sendStartAuthentication(HttpServletRequest request,
      HttpServletResponse response, FilterChain chain,
      AuthenticationException reason) throws ServletException, IOException {

       SecurityContextHolder.getContext().setAuthentication(null);
       requestCache.saveRequest(request, response);
       authenticationEntryPoint.commence(request, response, reason);
    }

    //... 

}
```

登录后，我们可以在`AuthenticationSuccessHandler`中自定义行为，如上图所示。

## 4。结论

在这个`Spring Security`例子中，我们讨论了登录后重定向的常见实践，并解释了使用 Spring Security 的实现。

注意**如果没有应用验证或额外的方法控制，我们提到的所有实现都容易受到某些攻击**。此类攻击可能会将用户重定向到恶意站点。

`OWASP`提供了一个[备忘单](https://web.archive.org/web/20221004133411/https://cheatsheetseries.owasp.org/cheatsheets/Unvalidated_Redirects_and_Forwards_Cheat_Sheet.html)来帮助我们处理未验证的重定向和转发。如果我们需要自己构建实现，这将非常有帮助。

这篇文章的完整实现代码可以在 Github 上找到[。](https://web.archive.org/web/20221004133411/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-web-login)