# 用@ExceptionHandler 处理 Spring 安全异常

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-exceptionhandler>

## 1.概观

在本教程中，我们将学习**如何用`@ExceptionHandler`和`@ControllerAdvice.` 全局处理 Spring 安全异常** **[控制器建议](/web/20221128045922/https://www.baeldung.com/exception-handling-for-rest-with-spring)是一个拦截器，它允许我们在应用**中使用相同的异常处理。

## 2.Spring 安全异常

Spring 安全核心异常如`AuthenticationException`和`AccessDeniedException`是运行时异常。由于这些**异常是由`DispatcherServlet`后面的认证过滤器抛出的，在调用控制器方法**之前，`@ControllerAdvice`将无法捕捉这些异常。

[Spring 安全异常](/web/20221128045922/https://www.baeldung.com/spring-security-exceptions)可以通过添加自定义过滤器和构造响应体来直接处理。为了通过`@ExceptionHandler`和`@ControllerAdvice,`在全局级别处理这些异常，我们需要一个 [`AuthenticationEntryPoint`](/web/20221128045922/https://www.baeldung.com/spring-security-basic-authentication) 的定制实现。 **`AuthenticationEntryPoint`用于发送 HTTP 响应，向客户端**请求凭证。尽管安全入口点有多个内置实现，但我们需要编写一个自定义实现来发送自定义响应消息。

首先，让我们看看如何在不使用`@ExceptionHandler`的情况下全局处理安全异常。

## 3.不带`@ExceptionHandler`

春季安全异常在`AuthenticationEntryPoint`开始。让我们为`AuthenticationEntryPoint`编写一个拦截安全异常的实现。

### 3.1.配置`AuthenticationEntryPoint`

让我们实现`AuthenticationEntryPoint`并覆盖`commence()`方法:

```java
@Component("customAuthenticationEntryPoint")
public class CustomAuthenticationEntryPoint implements AuthenticationEntryPoint {

    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException authException) 
      throws IOException, ServletException {

        RestError re = new RestError(HttpStatus.UNAUTHORIZED.toString(), "Authentication failed");
        response.setContentType(MediaType.APPLICATION_JSON_VALUE);
        response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
        OutputStream responseStream = response.getOutputStream();
        ObjectMapper mapper = new ObjectMapper();
        mapper.writeValue(responseStream, re);
        responseStream.flush();
    }
}
```

这里，我们使用`ObjectMapper`作为响应体的消息转换器。

### 3.2.配置`SecurityConfig`

接下来，让我们配置`SecurityConfig`来截取认证路径。这里我们将配置'`/login`'作为上述实现的路径。此外，我们将为“管理员”用户配置“管理员”角色:

```java
@Configuration
@EnableWebSecurity
public class CustomSecurityConfig {

    @Autowired
    @Qualifier("customAuthenticationEntryPoint")
    AuthenticationEntryPoint authEntryPoint;

    @Bean
    public UserDetailsService userDetailsService() {
        UserDetails admin = User.withUsername("admin")
            .password("password")
            .roles("ADMIN")
            .build();
        InMemoryUserDetailsManager userDetailsManager = new InMemoryUserDetailsManager();
        userDetailsManager.createUser(admin);
        return userDetailsManager;
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.requestMatchers()
            .antMatchers("/login")
            .and()
            .authorizeRequests()
            .anyRequest()
            .hasRole("ADMIN")
            .and()
            .httpBasic()
            .and()
            .exceptionHandling()
            .authenticationEntryPoint(authEntryPoint);
        return http.build();
    }
}
```

### 3.3.配置 Rest 控制器

现在，让我们编写一个监听这个端点'/login '的 rest 控制器:

```java
@PostMapping(value = "/login", produces = MediaType.APPLICATION_JSON_VALUE)
public ResponseEntity<RestResponse> login() {
    return ResponseEntity.ok(new RestResponse("Success"));
} 
```

### 3.4.测试

最后，让我们用模拟测试来测试这个端点。

首先，让我们编写一个成功认证的测试用例:

```java
@Test
@WithMockUser(username = "admin", roles = { "ADMIN" })
public void whenUserAccessLogin_shouldSucceed() throws Exception {
    mvc.perform(formLogin("/login").user("username", "admin")
      .password("password", "password")
      .acceptMediaType(MediaType.APPLICATION_JSON))
      .andExpect(status().isOk());
} 
```

接下来，我们来看一个身份验证失败的场景:

```java
@Test
public void whenUserAccessWithWrongCredentialsWithDelegatedEntryPoint_shouldFail() throws Exception {
    RestError re = new RestError(HttpStatus.UNAUTHORIZED.toString(), "Authentication failed");
    mvc.perform(formLogin("/login").user("username", "admin")
      .password("password", "wrong")
      .acceptMediaType(MediaType.APPLICATION_JSON))
      .andExpect(status().isUnauthorized())
      .andExpect(jsonPath("$.errorMessage", is(re.getErrorMessage())));
} 
```

现在，让我们看看如何用`@ControllerAdvice`和`@ExceptionHandler`实现同样的效果。

## 4.用`@ExceptionHandler`

这种方法允许我们使用完全相同的异常处理技术，但是在控制器建议中以更干净和更好的方式使用带有`@ExceptionHandler`注释的方法。

### 4.1.配置`AuthenticationEntryPoint`

类似于上面的方法，我们将实现`AuthenticationEntryPoint`，然后将异常处理程序委托给`HandlerExceptionResolver`:

```java
@Component("delegatedAuthenticationEntryPoint")
public class DelegatedAuthenticationEntryPoint implements AuthenticationEntryPoint {

    @Autowired
    @Qualifier("handlerExceptionResolver")
    private HandlerExceptionResolver resolver;

    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException authException) 
      throws IOException, ServletException {
        resolver.resolveException(request, response, null, authException);
    }
}
```

这里我们注入了`DefaultHandlerExceptionResolver`并将处理程序委托给这个解析器。这个安全异常现在可以用控制器通知和异常处理程序方法来处理。

### 4.2.配置`ExceptionHandler`

现在，对于[异常处理程序](/web/20221128045922/https://www.baeldung.com/exception-handling-for-rest-with-spring)的主配置，我们将扩展`ResponseEntityExceptionHandler`并用`@ControllerAdvice`注释这个类:

```java
@ControllerAdvice
public class DefaultExceptionHandler extends ResponseEntityExceptionHandler {

    @ExceptionHandler({ AuthenticationException.class })
    @ResponseBody
    public ResponseEntity<RestError> handleAuthenticationException(Exception ex) {

        RestError re = new RestError(HttpStatus.UNAUTHORIZED.toString(), 
          "Authentication failed at controller advice");
        return ResponseEntity.status(HttpStatus.UNAUTHORIZED).body(re);
    }
}
```

### 4.3.配置`SecurityConfig`

现在，让我们为这个委托身份验证入口点编写一个安全配置:

```java
@Configuration
@EnableWebSecurity
public class DelegatedSecurityConfig {

    @Autowired
    @Qualifier("delegatedAuthenticationEntryPoint")
    AuthenticationEntryPoint authEntryPoint;

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.requestMatchers()
            .antMatchers("/login-handler")
            .and()
            .authorizeRequests()
            .anyRequest()
            .hasRole("ADMIN")
            .and()
            .httpBasic()
            .and()
            .exceptionHandling()
            .authenticationEntryPoint(authEntryPoint);
        return http.build();
    }

    @Bean
    public InMemoryUserDetailsManager userDetailsService() {
        UserDetails admin = User.withUsername("admin")
            .password("password")
            .roles("ADMIN")
            .build();
        return new InMemoryUserDetailsManager(admin);
    }
}
```

对于'`/login-handler`'端点，我们已经用上面实现的`DelegatedAuthenticationEntryPoint`配置了异常处理程序。

### 4.4.配置 Rest 控制器

让我们为'`/login-handler`'端点配置 rest 控制器:

```java
@PostMapping(value = "/login-handler", produces = MediaType.APPLICATION_JSON_VALUE)
public ResponseEntity<RestResponse> loginWithExceptionHandler() {
    return ResponseEntity.ok(new RestResponse("Success"));
} 
```

### 4.5.试验

现在让我们测试这个端点:

```java
@Test
@WithMockUser(username = "admin", roles = { "ADMIN" })
public void whenUserAccessLogin_shouldSucceed() throws Exception {
    mvc.perform(formLogin("/login-handler").user("username", "admin")
      .password("password", "password")
      .acceptMediaType(MediaType.APPLICATION_JSON))
      .andExpect(status().isOk());
}

@Test
public void whenUserAccessWithWrongCredentialsWithDelegatedEntryPoint_shouldFail() throws Exception {
    RestError re = new RestError(HttpStatus.UNAUTHORIZED.toString(), "Authentication failed at controller advice");
    mvc.perform(formLogin("/login-handler").user("username", "admin")
      .password("password", "wrong")
      .acceptMediaType(MediaType.APPLICATION_JSON))
      .andExpect(status().isUnauthorized())
      .andExpect(jsonPath("$.errorMessage", is(re.getErrorMessage())));
} 
```

在成功测试中，我们已经使用预先配置的用户名和密码测试了端点。在失败测试中，我们已经验证了响应主体中的状态代码和错误消息的响应。

## 5.结论

在本文中，**我们学习了如何用`@ExceptionHandler`** 全局处理 **Spring 安全异常。此外，我们还创建了一个全功能示例，帮助我们理解所解释的概念。**

GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20221128045922/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-core-2)