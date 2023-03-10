# 防止 Spring 应用程序中的跨站点脚本(XSS)

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-prevent-xss>

## 1.概观

在构建 Spring web 应用程序时，关注安全性非常重要。[跨站脚本(XSS)](https://web.archive.org/web/20220617075711/https://en.wikipedia.org/wiki/Cross-site_scripting) 是网络安全最关键的攻击之一。

在 Spring 应用程序中，防止 XSS 攻击是一个挑战。Spring 提供了内置的帮助来提供完整的保护。

在本教程中，我们将使用可用的 Spring 安全特性。

## 2.什么是跨站点脚本(XSS)攻击？

### 2.1.问题的定义

XSS 是一种常见的注入式攻击。在 XSS，攻击者试图在 web 应用程序中执行恶意代码。他们通过网络浏览器或 HTTP 客户端工具如 [Postman](/web/20220617075711/https://www.baeldung.com/postman-testing-collections) 与它互动。

有两种类型的 XSS 攻击:

*   反射或非持久 XSS
*   存储的或持久的 XSS

在反射或非持久 XSS 中，不可信的用户数据被提交给 web 应用程序，它会立即在响应中返回，从而向页面添加不可信的内容。web 浏览器假定代码来自 web 服务器并执行它。这可能会让黑客向您发送一个链接，当跟踪该链接时，会导致您的浏览器从您使用的网站检索您的私人数据，然后让您的浏览器将其转发到黑客的服务器。

在存储或持久 XSS 中，攻击者的输入由 web 服务器存储。随后，任何未来的访问者都可能执行该恶意代码。

### 2.2.抵御攻击

防止 XSS 攻击的主要策略是清除用户输入。

在 Spring web 应用程序中，用户的输入是一个 HTTP 请求。为了防止攻击，我们应该检查 HTTP 请求的内容，并删除任何可能被服务器或浏览器执行的内容。

对于一个通过网络浏览器访问的常规网络应用程序，我们可以使用 [Spring Security](/web/20220617075711/https://www.baeldung.com/security-spring) 的内置特性(反映了 XSS)。

## 3.使用 Spring Security 使应用程序 XSS 安全

默认情况下，Spring Security 提供了几个安全头。它包括 [`X-XSS-Protection`](https://web.archive.org/web/20220617075711/https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-XSS-Protection) 报头。`X-XSS-Protection` 告诉浏览器屏蔽看起来像 XSS 的内容。Spring Security 可以自动将这个安全头添加到响应中。为了激活它，我们在 Spring 安全配置类中配置了 XSS 支持。

使用此功能，浏览器在检测到 XSS 尝试时不会呈现。然而，一些网络浏览器还没有实现 XSS 审计器。在这种情况下，他们不使用`X-XSS-Protection` 报头`.` 来克服这个问题，我们也可以使用[内容安全策略(CSP)](https://web.archive.org/web/20220617075711/https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy) 功能。

CSP 是一个附加的安全层，有助于缓解 XSS 和数据注入攻击。要启用它，我们需要配置我们的应用程序，通过提供一个`WebSecurityConfigurerAdapter` bean 来返回一个`Content-Security-Policy`头:

```java
@Configuration
public class SecurityConf extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
          .headers()
          .xssProtection()
          .and()
          .contentSecurityPolicy("script-src 'self'");
    }
}
```

## 4.结论

在本文中，我们看到了如何使用 Spring Security 的`xssProtection`特性来防止 XSS 攻击。

和往常一样，源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220617075711/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-5-security)