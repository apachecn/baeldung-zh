# 如何禁用 Spring 安全注销重定向

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-disable-logout-redirects>

## 1.概观

在这个简短的教程中，我们将仔细看看**如何在 Spring Security** 中禁用注销重定向。

我们将首先从注销流在 Spring Security 中如何工作的简单背景开始。然后，我们将通过一个实际的例子来说明如何在成功注销后避免用户重定向。

## 2.Spring Security 中的注销

简而言之，Spring Security 通过`logout()` DSL 方法为[注销机制提供了开箱即用的支持。基本上， **Spring Security 触发器在用户点击默认注销 URL`/logout`**`**.**`时注销](/web/20220706105134/https://www.baeldung.com/spring-security-logout#config)

值得一提的是，在[春安 4](https://web.archive.org/web/20220706105134/https://docs.spring.io/spring-security/site/migrate/current/3-to-4/html5/migrate-3-to-4-xml.html#m3to4-xmlnamespace-logout) `.`之前，注销网址的默认值为`/j_spring_security_logout`

Spring Security 提供了在注销后将用户重定向到特定 URL 的可能性。但是，有些场合我们要避免这种行为。

所以，事不宜迟，**让我们看看如何在 Spring Security** 中实现禁用注销重定向的逻辑。

## 3.禁用 Spring 安全注销重定向

默认情况下，Spring Security 会在成功注销后将用户重定向到`/login?logout` 。因此，在这一节中，我们将重点讨论如何防止用户在注销后重定向到登录页面。

注意，我们可以在 [`logoutSuccessUrl()` DSL 方法](/web/20220706105134/https://www.baeldung.com/spring-security-logout#1-logoutsuccessurl) `.`的帮助下覆盖默认的重定向 URL

**这里的要点是展示当从 REST 客户端调用`/logout` URL 时如何避免重定向。**

事实上， [`Log` `outSuccessHandler`](/web/20220706105134/https://www.baeldung.com/spring-security-logout#4-logoutsuccesshandler) 界面提供了一种灵活的方式来在注销过程成功执行时执行自定义逻辑。

所以在这里，我们将**使用一个自定义`LogoutSuccessHandler`来返回一个干净的 200 状态码**。这样，它就不会将我们重定向到任何页面。

现在，让我们实现禁用注销重定向所需的必要的 Spring 安全配置:

```
@Configuration
@EnableWebSecurity
public class SpringSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests(authz -> authz
                .mvcMatchers("/login").permitAll()
                .anyRequest().authenticated()
            )
            .logout(logout -> logout
                .permitAll()
                .logoutSuccessHandler((request, response, authentication) -> {
                    response.setStatus(HttpServletResponse.SC_OK);
                }
            );
    }

}
```

上述配置中需要注意的重要部分是`logoutSuccessHandler()`方法。正如我们所看到的，[我们使用一个 lambda 表达式](/web/20220706105134/https://www.baeldung.com/java-8-lambda-expressions-tips)来定义我们的定制注销成功处理程序。

请记住，我们还可以创建一个简单的`LogoutSuccessHandler` 接口的[实现类，并使用 DSL 将其传递给`logoutSuccessHandler()`方法。](/web/20220706105134/https://www.baeldung.com/spring-security-track-logged-in-users#2-implementing-logoutsuccesshandler)

## 4.测试

既然我们把所有的部分都放在了一起，让我们测试一下`/logout`端点，以确认一切都按预期工作。

注意，我们将在测试中使用 [`MockMvc`](/web/20220706105134/https://www.baeldung.com/spring-security-integration-tests#testing-controllers-with-webmvctest) 来发送`/logout`请求。

**首先，让我们创建一个简单的测试类，并在其中注入`MockMvc`对象:**

```
public class LogoutApplicationUnitTest {

    @Autowired
    private MockMvc mockMvc;

    // test case

}
```

现在，**让我们写一个方法来测试我们的`/logout`端点:**

```
@Test
public void whenLogout_thenDisableRedirect() throws Exception {

    this.mockMvc.perform(post("/logout").with(csrf()))
        .andExpect(status().isOk())
        .andExpect(jsonPath("$").doesNotExist())
        .andExpect(unauthenticated())
        .andReturn();
}
```

最后，让我们尝试分解我们的测试代码:

*   `perform(post(“/logout”))` 调用`/logout`端点作为一个简单的 POST 请求
*   [`with(csrf())`](/web/20220706105134/https://www.baeldung.com/spring-security-csrf#enabled) 向查询添加预期的`_csrf`参数
*   `status()` 返回 HTTP 响应的状态代码
*   `jsonPath()` 允许访问和检查 HTTP 响应的主体

## 5.结论

总之，我们已经解释并举例说明了如何在 Spring Security 和 Spring Boot 中解决禁用注销重定向的挑战。

像往常一样，本文的完整源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220706105134/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-5-security)