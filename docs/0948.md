# 使用 Spring Security 手动注销

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-manual-logout>

## 1.介绍

Spring Security 是保护基于 Spring 的应用程序的标准。它有几个功能来管理用户的身份验证，包括登录和注销。

在本教程中，我们将重点介绍使用 Spring Security 手动注销 **。**

我们先假设读者已经了解了标准的 [春季安全注销](/web/20220630140457/https://www.baeldung.com/spring-security-logout)流程。

## 2.基本注销

当 **用户试图注销时，会对其当前会话状态** 产生若干后果。我们需要用两个步骤来销毁会话:

1.  使 HTTP 会话信息无效。
2.  清除`SecurityContext`，因为它包含认证信息。

这两个动作由`SecurityContextLogoutHandler.` 执行

让我们看看实际情况:

```
@Configuration
public class DefaultLogoutConfiguration extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
          .logout(logout -> logout
            .logoutUrl("/basic/basiclogout")
            .addLogoutHandler(new SecurityContextLogoutHandler())
          );
    }
}
```

请注意，`SecurityContextLogoutHandler`是由 Spring Security 默认添加的——我们只是为了清晰起见才在这里显示它。

## 3.Cookie 清除注销

通常，注销还需要我们清除部分或全部用户的 cookies。

为此，我们可以创建自己的`LogoutHandler` 来遍历所有 cookies，并在注销时使它们过期:

```
@Configuration
public class AllCookieClearingLogoutConfiguration extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
          .logout(logout -> logout
            .logoutUrl("/cookies/cookielogout")
            .addLogoutHandler((request, response, auth) -> {
                for (Cookie cookie : request.getCookies()) {
                    String cookieName = cookie.getName();
                    Cookie cookieToDelete = new Cookie(cookieName, null);
                    cookieToDelete.setMaxAge(0);
                    response.addCookie(cookieToDelete);
                }
            })
          );
    }
}
```

事实上，Spring Security 提供了`CookieClearingLogoutHandler` ，这是一个现成的注销处理程序，用于清除 cookie。

## 4.`Clear-Site-Data`表头注销

同样，我们可以使用一个特殊的 HTTP 响应头来实现同样的事情；这就是 [**`Clear-Site-Data`报头**](/web/20220630140457/https://www.baeldung.com/spring-security-clear-site-data-header) 发挥作用的地方。

基本上，`Clear-Data-Site`头清除了与请求网站相关的浏览数据(cookies、存储、缓存):

```
@Configuration
public class ClearSiteDataHeaderLogoutConfiguration extends WebSecurityConfigurerAdapter {

    private static final ClearSiteDataHeaderWriter.Directive[] SOURCE = 
      {CACHE, COOKIES, STORAGE, EXECUTION_CONTEXTS};

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
          .logout(logout -> logout
            .logoutUrl("/csd/csdlogout")
            .addLogoutHandler(new HeaderWriterLogoutHandler(new ClearSiteDataHeaderWriter(SOURCE)))
          );
    }
}
```

然而，当我们仅清除一种类型的存储时，存储清理可能会破坏应用程序状态。因此，由于[不完全清除](https://web.archive.org/web/20220630140457/https://w3c.github.io/webappsec-clear-site-data/#incomplete)，只有在请求是安全的情况下才会应用报头。

## 5.请求注销

同样，我们可以使用 [`HttpServletRequest.logout()`](https://web.archive.org/web/20220630140457/https://docs.spring.io/spring-security/site/docs/current/reference/html5/#servletapi-logout) 的方法来注销用户。

首先，让我们添加必要的配置，以便根据请求手动调用`logout()`:

```
@Configuration
public static class LogoutOnRequestConfiguration extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.antMatcher("/request/**")
            .authorizeRequests(authz -> authz.anyRequest()
                .permitAll())
            .logout(logout -> logout.logoutUrl("/request/logout")
                .addLogoutHandler((request, response, auth) -> {
                    try {
                        request.logout();
                    } catch (ServletException e) {
                        logger.error(e.getMessage());
                    }
                }));
    }
}
```

最后，让我们创建一个测试用例来确认一切都按预期运行:

```
@Test
public void givenLoggedUserWhenUserLogoutOnRequestThenSessionCleared() throws Exception {

    this.mockMvc.perform(post("/request/logout").secure(true)
        .with(csrf()))
        .andExpect(status().is3xxRedirection())
        .andExpect(unauthenticated())
        .andReturn();
}
```

## 6.结论

总之，Spring Security 有许多处理认证场景的内置特性。掌握如何以编程方式使用这些特性总是很方便的。

和往常一样，这些例子的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220630140457/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-5-security)