# Spring Security 自定义认证失败处理程序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-custom-authentication-failure-handler>

## 1.概观

在这个快速教程中，我们将演示如何在 Spring Boot 应用程序中定制 Spring Security 的认证失败处理。目标是使用`form login` 方法`.`认证用户

关于`Spring Boot`中`Spring Security` 和`Form Login `的介绍，请分别参见[本](/web/20220902174603/https://www.baeldung.com/spring-boot-security-autoconfiguration)和[本](/web/20220902174603/https://www.baeldung.com/spring-security-login)篇。

## 2.认证和授权

`Authentication`和`Authorization`经常结合使用，因为在授予系统访问权限时，它们扮演着重要的角色。

但是，在验证请求时，它们具有不同的含义并应用不同的约束:

*   `**Authentication**`–位于`Authorization;`之前，用于验证收到的凭证；在这里，我们验证用户名和密码是否与我们的应用程序识别的相匹配
*   **`Authorization`** `–`验证成功认证的用户是否有权限访问应用的某项功能

我们可以定制`authentication`和`authorization`故障处理，但是，在这个应用程序中，我们将重点关注认证故障。

## 3.春天安全的`AuthenticationFailureHandler`

`Spring Security`默认为我们提供一个处理认证失败的组件。

然而，我们经常会发现默认行为不足以满足需求。

如果是这样，我们可以通过实现 **[`AuthenticationFailureHandler`](https://web.archive.org/web/20220902174603/https://docs.spring.io/spring-security/site/docs/4.2.6.RELEASE/apidocs/org/springframework/security/web/authentication/AuthenticationFailureHandler.html)** 接口来创建我们自己的组件并提供我们想要的自定义行为:

```java
public class CustomAuthenticationFailureHandler 
  implements AuthenticationFailureHandler {

    private ObjectMapper objectMapper = new ObjectMapper();

    @Override
    public void onAuthenticationFailure(
      HttpServletRequest request,
      HttpServletResponse response,
      AuthenticationException exception) 
      throws IOException, ServletException {

        response.setStatus(HttpStatus.UNAUTHORIZED.value());
        Map<String, Object> data = new HashMap<>();
        data.put(
          "timestamp", 
          Calendar.getInstance().getTime());
        data.put(
          "exception", 
          exception.getMessage());

        response.getOutputStream()
          .println(objectMapper.writeValueAsString(data));
    }
}
```

默认情况下，`Spring` `redirects`用户回到带有`request parameter`的登录页面，其中包含有关错误的信息。

在这个应用程序中，我们将返回一个 401 响应，其中包含有关错误的信息，以及错误发生的时间戳。

除了默认组件之外，`Spring`还有其他现成的组件，我们可以根据自己的需要利用这些组件:

*   **`DelegatingAuthenticationFailureHandler`** 将`[AuthenticationException](https://web.archive.org/web/20220902174603/https://docs.spring.io/spring-security/site/docs/4.2.4.RELEASE/apidocs/org/springframework/security/core/AuthenticationException.html)`子类委托给不同的`AuthenticationFailureHandlers`，这意味着我们可以为`AuthenticationException`的不同实例创建不同的行为
*   `**ExceptionMappingAuthenticationFailureHandler**`根据`AuthenticationException's `完整的类名将用户重定向到特定的 URL
*   **`ForwardAuthenticationFailureHandler`** 会将用户转发到指定的 URL，而不考虑`AuthenticationException`的类型
*   **`SimpleUrlAuthenticationFailureHandler`** 是默认使用的组件，如果指定，它会将用户重定向到`failureUrl, `；否则，它将简单地返回一个 401 响应

既然我们已经创建了自定义的`AuthenticationFailureHandler`，让我们配置我们的应用程序并覆盖默认的`Spring's`处理程序:

```java
@Configuration
@EnableWebSecurity
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(AuthenticationManagerBuilder auth) 
      throws Exception {
        auth.inMemoryAuthentication()
          .withUser("user1").password(passwordEncoder.encode("user1Pass")).roles("USER");
    }

    @Override
    protected void configure(HttpSecurity http) 
      throws Exception {
        http
          .authorizeRequests()
          .anyRequest()
          .authenticated()
          .and()
          .formLogin()
          .failureHandler(authenticationFailureHandler());
    }

    @Bean
    public AuthenticationFailureHandler authenticationFailureHandler() {
        return new CustomAuthenticationFailureHandler();
    }
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
} 
```

**注意`failureHandler()` 调用——在这里我们可以告诉`Spring`使用我们的定制组件，而不是使用默认组件。**

## 4.结论

在这个例子中，我们利用`Spring's AuthenticationFailureHandler`接口定制了应用程序的认证失败处理程序。

这个例子的实现可以在[Github 项目](https://web.archive.org/web/20220902174603/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-web-login)中找到。

在本地运行时，您可以在`localhost:8080`访问和测试应用程序