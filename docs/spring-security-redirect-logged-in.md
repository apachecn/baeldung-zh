# 使用 Spring Security 重定向登录用户

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-redirect-logged-in>

## 1.概观

当用户已经登录时，网站通常会阻止他们访问登录。一种常见的方法是将用户重定向到另一个页面，通常是登录后应用程序的起点。

在本教程中，我们将探索使用 Spring Security 实现这个解决方案的多种方法。

此外，要了解更多关于如何快速实现登录的信息，我们可以从[这篇文章](/web/20220628235827/https://www.baeldung.com/spring-security-login)开始。

## 2.认证验证

首先，我们需要一种方法来验证身份验证。

换句话说，**我们需要从`SecurityContext`获取认证细节，并验证用户是否登录了**:

```
private boolean isAuthenticated() {
    Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
    if (authentication == null || AnonymousAuthenticationToken.class.
      isAssignableFrom(authentication.getClass())) {
        return false;
    }
    return authentication.isAuthenticated();
}
```

我们将在以下所有负责重定向的组件中使用它。

## 3.从登录控制器重定向

实现我们目标的最简单方法是在控制器中为登录页面定义一个端点。

如果用户通过了身份验证，我们还需要返回特定的页面，否则返回登录页面:

```
@GetMapping("/loginUser")
public String getUserLoginPage() {
    if (isAuthenticated()) {
        return "redirect:userMainPage";
    }
    return "loginUser";
}
```

## 4.使用拦截器

另一种重定向用户的方法是通过登录页面 URI 上的拦截器。

拦截器将在请求到达控制器之前拦截它。因此，我们可以根据身份验证来决定是让它继续下去，还是阻止它并返回一个重定向响应。

如果用户通过了身份验证，我们需要修改响应中的两件事:

*   将状态代码设置为`HttpStatus.SC_TEMPORARY_REDIRECT`
*   添加带有重定向 URL 的`Location`头

最后，我们将通过返回`false`来中断执行链:

```
public class LoginPageInterceptor implements HandlerInterceptor {
    UrlPathHelper urlPathHelper = new UrlPathHelper();
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        if ("/loginUser".equals(urlPathHelper.getLookupPathForRequest(request)) && isAuthenticated()) {
            String encodedRedirectURL = response.encodeRedirectURL(
              request.getContextPath() + "/userMainPage");
            response.setStatus(HttpStatus.SC_TEMPORARY_REDIRECT);
            response.setHeader("Location", encodedRedirectURL);

            return false;
        } else {
            return true;
        }
    }

    // isAuthenticated method 
}
```

**我们还需要将拦截器添加到 Spring MVC 生命周期中**:

```
@Configuration
public class LoginRedirectMvcConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LoginPageInterceptor());
    }
} 
```

我们可以使用 Spring 的基于 XML 模式的配置来达到同样的目的:

```
<mvc:interceptors>
    <mvc:interceptor>
        <mvc:mapping path="/loginUser"/>
        <bean class="com.baeldung.loginredirect.LoginPageInterceptor"/>
    </mvc:interceptor>
</mvc:interceptors>
```

## 5.使用过滤器

类似地，我们可以实现一个弹簧过滤器。

可以使用 Spring Security 的过滤器链将过滤器直接应用于`SecurityContext`。因此，它可以在创建身份验证后立即拦截请求。

让我们扩展`GenericFilterBean,`覆盖`doFilter`方法，并验证认证:

```
public class LoginPageFilter extends GenericFilterBean {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
      throws IOException, ServletException {
        HttpServletRequest servletRequest = (HttpServletRequest) request;
        HttpServletResponse servletResponse = (HttpServletResponse) response;

        if (isAuthenticated() && "/loginUser".equals(servletRequest.getRequestURI())) {

            String encodedRedirectURL = ((HttpServletResponse) response).encodeRedirectURL(
              servletRequest.getContextPath() + "/userMainPage");

            servletResponse.setStatus(HttpStatus.SC_TEMPORARY_REDIRECT);
            servletResponse.setHeader("Location", encodedRedirectURL);
        }

        chain.doFilter(servletRequest, servletResponse);
    }
    // isAuthenticated method 
}
```

**我们需要在滤波器链中的`UsernamePasswordAuthenticationFilter `之后添加滤波器。**

此外，我们需要授权对登录页面 URI 的请求，以便为它启用过滤器链:

```
@Configuration
@EnableWebSecurity
public class LoginRedirectSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
          .addFilterAfter(new LoginPageFilter(), UsernamePasswordAuthenticationFilter.class)
          .authorizeRequests().antMatchers("/loginUser").permitAll()

        // Other security configuration
    }
} 
```

最后，如果我们选择使用 XML 配置，我们可以为过滤器定义 bean，并将其添加到安全`HTTP`标签中的过滤器链中:

```
<beans:bean id="loginPageFilter" class="com.baeldung.loginredirect.LoginPageFilter"/>

<security:http pattern="/**" use-expressions="true" auto-config="true">
    <security:intercept-url pattern="/loginUser" access="permitAll"/>
    <security:custom-filter after="BASIC_AUTH_FILTER" ref="loginPageFilter"/>
</security:http>
```

关于如何为 Spring 安全创建自定义过滤器的快速教程可以在[这里](/web/20220628235827/https://www.baeldung.com/spring-security-custom-filter)找到。

## 6.结论

在本教程中，我们探索了使用 Spring Security 从登录页面重定向已经登录的用户的多种方法。

和往常一样，本教程中使用的完整源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220628235827/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-web-boot-2)

另一个可能感兴趣的教程是[使用 Spring Security](/web/20220628235827/https://www.baeldung.com/spring_redirect_after_login) 登录后重定向到不同的页面，在这个教程中，我们学习了如何将不同类型的用户重定向到特定的页面。