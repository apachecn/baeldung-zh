# Spring 安全中的 Clear-Site-Data 头

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-clear-site-data-header>

## 1.概观

为了网络优化，一些网站允许浏览器在本地存储中缓存 CSS 或 JS 之类的资源。这允许浏览器为每个请求节省一次网络往返。

因此，缓存资源对于改善网页的加载时间至关重要。同样重要的是，一旦不需要缓存的数据，就将其清除。**例如，** **如果用户从网站注销，浏览器应该从缓存中删除所有会话数据。**

浏览器缓存数据的时间超过要求有两个主要问题:

*   现代网站使用一组丰富的 CSS 和 JS 文件，这些文件消耗了大量的浏览器内存
*   缓存会话 cookies 等敏感数据的网站容易受到网络钓鱼攻击

在本教程中，我们将看到 HTTP 的`Clear-Site-Data`响应头如何帮助网站从浏览器中清除本地存储的数据。

## 2.`Clear-Site-Data`表头

就像`[Cache-Control](/web/20220905102449/https://www.baeldung.com/spring-mvc-cache-headers)`头一样，`Clear-Site-Data`也是一个 HTTP 响应头。网站可以使用此标题来指示浏览器删除缓存在本地存储中的数据。

对于需要认证的网站，`Cache-Control `头一般包含在`/login `响应中，允许浏览器缓存用户数据。类似地，网站在`/logout `响应中包含`Clear-Site-Data` 头，以清除属于该用户的任何缓存数据。

此时，理解浏览器通常将本地存储分为不同的类型是很重要的:

*   局部存储器
*   会话存储
*   饼干

**由于网站可以存储这些类型中的任何一种数据，`Clear-Site-Data` 允许我们在标题中指定目标存储:**

*   `cache`–删除本地缓存数据，包括私有和共享浏览器缓存
*   `cookies`–删除存储在浏览器 cookies 中的数据
*   `storage`–清除浏览器的本地和会话存储
*   `executionContexts`–此开关告知浏览器重新加载该 URL 的浏览器标签
*   `*`(星号)–从上述所有存储区域删除数据

因此，`Clear-Site-Data `头必须包括至少一种存储类型:

```
Clear-Site-Data: "cache", "cookies", "storage", "executionContexts"
```

在接下来的小节中，我们将在 Spring Security 中实现一个`/logout `服务，并在响应中包含一个`Clear-Site-Data `头。

## 3.Maven 依赖性

在我们编写一些代码以在 Spring 中添加`Clear-Site-Data `头之前，让我们将 [`spring-security-web`](https://web.archive.org/web/20220905102449/https://search.maven.org/artifact/org.springframework.security/spring-security-web) 和 [`spring-security-config`](https://web.archive.org/web/20220905102449/https://search.maven.org/artifact/org.springframework.security/spring-security-config) 依赖项添加到项目中:

```
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-web</artifactId>
    <version>5.2.2.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-config</artifactId>
    <version>5.2.2.RELEASE</version>
</dependency>
```

## 4.`ClearSiteDataHeaderWriter `春天里的安全

我们之前讨论过，Spring 提供了一个`[CacheControl](/web/20220905102449/https://www.baeldung.com/spring-security-cache-control-headers) `实用程序类来在响应中写入`Cache-Control `头。**同样，Spring Security 提供了一个`ClearSiteDataHeaderWriter `类来轻松地在 HTTP 响应中添加报头**:

```
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SpringSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf()
          .disable()
          .formLogin()
          .loginPage("/login.html")
          .loginProcessingUrl("/perform_login")
          .defaultSuccessUrl("/homepage.html", true)
          .and()
          .logout().logoutUrl("/baeldung/logout")
          .addLogoutHandler(new HeaderWriterLogoutHandler(
            new ClearSiteDataHeaderWriter(
              ClearSiteDataHeaderWriter.Directive.CACHE,
              ClearSiteDataHeaderWriter.Directive.COOKIES,
              ClearSiteDataHeaderWriter.Directive.STORAGE)));
    }
}
```

这里，我们用 Spring Security 实现了一个登录和注销页面。因此，Spring 将添加一个`Clear-Site-Data` 头来响应所有的`/baeldung/logout` 请求`:`

```
Clear-Site-Data: "cache", "cookies", "storage"
```

如果我们现在使用`curl `并向`https://localhost:8080/baeldung/logout`发送一个请求，我们将得到以下响应头:

```
{ [5 bytes data]
< HTTP/1.1 302
< Clear-Site-Data: "cache", "cookies", "storage"
< X-Content-Type-Options: nosniff
< X-XSS-Protection: 1; mode=block
< Cache-Control: no-cache, no-store, max-age=0, must-revalidate
< Pragma: no-cache
< Expires: 0
< Strict-Transport-Security: max-age=31536000 ; includeSubDomains
< X-Frame-Options: DENY
< Location: https://localhost:8080/login.html?logout
< Content-Length: 0
< Date: Tue, 17 Mar 2020 17:12:23 GMT
```

## 5.结论

在本文中，我们研究了浏览器缓存关键用户数据的影响，即使这并不是必需的。例如，浏览器不应该在用户退出网站后缓存数据。

然后我们看到 HTTP 的`Clear-Site-Data `响应头是如何允许网站强制浏览器清除本地缓存数据的。

**最后，我们用`ClearSiteDataHeaderWriter `在 Spring Security 中实现了一个注销页面，将这个头添加到控制器的响应中。**

和往常一样，代码可以在 GitHub 的[处获得。](https://web.archive.org/web/20220905102449/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-web-mvc)