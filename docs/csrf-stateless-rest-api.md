# 具有无状态 REST API 的 CSRF

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/csrf-stateless-rest-api>

## 1.概观

在我们的[上一篇文章](/web/20220617075711/https://www.baeldung.com/csrf-thymeleaf-with-spring-security)中，我们已经解释了 CSRF 攻击如何影响 Spring MVC 应用程序。

本文将通过不同的案例来确定无状态 REST API 是否容易受到 CSRF 攻击，如果是，如何保护它免受攻击。

## 2.REST API 需要 CSRF 保护吗？

首先，我们可以在我们的[专用指南](/web/20220617075711/https://www.baeldung.com/spring-security-csrf#example)中找到一个 CSRF 攻击的例子。

现在，在阅读本指南时，我们可能认为无状态 REST API 不会受到这种攻击的影响，因为在服务器端没有会话可窃取。

让我们举一个典型的例子:一个 Spring REST API 应用程序和一个 Javascript 客户端。客户端使用一个安全令牌作为凭证(比如 JSESSIONID 或 [JWT](https://web.archive.org/web/20220617075711/https://en.wikipedia.org/wiki/JSON_Web_Token) )，在用户成功登录后由 REST API 发布。

**CSRF 漏洞取决于客户端如何存储这些凭证并将其发送给 API** 。

让我们回顾一下不同的选项，以及它们将如何影响我们的应用程序漏洞。

我们举一个典型的例子:一个 Spring REST API 应用程序和一个 Javascript 客户端。客户端使用一个安全令牌作为凭证(比如 JSESSIONID 或 [JWT](https://web.archive.org/web/20220617075711/https://en.wikipedia.org/wiki/JSON_Web_Token) )，在用户成功登录后由 REST API 发布。

### 2.1.凭据不是持久的

一旦我们从 REST API 中检索到令牌，我们就可以将令牌设置为一个 JavaScript 全局变量。这将把令牌保存在浏览器的内存中，并且只能用于当前页面。

这是最安全的方式:CSRF 和 XSS 攻击总是导致在一个新的页面上打开客户端应用程序，这无法访问用于登录的初始页面的内存。

但是，我们的用户每次访问或刷新页面时都必须重新登录。

在移动浏览器上，即使浏览器进入后台，也会发生这种情况，因为系统会清除内存。

**这对用户的限制如此之大，以至于该选项很少实施**。

### 2.2.存储在浏览器存储中的凭据

我们可以将令牌保存在浏览器存储中，例如会话存储。然后，我们的 JavaScript 客户机可以从中读取令牌，并在所有 REST 请求中发送带有该令牌的授权头。

这是一种普遍使用的方式，例如，JWT: **它很容易实现，并防止攻击者使用 CSRF 攻击**。事实上，与 cookies 不同，浏览器存储变量不会自动发送到服务器。

**然而，这种实现容易受到 XSS 攻击**:恶意的 JavaScript 代码可以访问浏览器存储并随请求发送令牌。在这种情况下，我们必须[保护我们的应用](/web/20220617075711/https://www.baeldung.com/spring-prevent-xss)。

### 2.3.存储在 Cookies 中的凭据

另一种选择是使用 cookie 来保存凭证。然后，我们的应用程序的漏洞取决于我们的应用程序如何使用 cookie。

我们可以使用 cookie 只保存凭证，就像 JWT 一样，但不能验证用户。

我们的 JavaScript 客户机必须读取令牌，并在授权头中将它发送给 API。

**在这种情况下，我们的应用程序不容易受到 CSRF** 的攻击:即使 cookie 是通过恶意请求自动发送的，我们的 REST API 也会从授权头而不是 cookie 中读取凭证。然而，`HTTP-only` 标志必须转换为`false`才能让我们的客户端读取 cookie。

然而，这样做的话，我们的应用程序将很容易受到 XSS 的攻击，就像上一节一样。

另一种方法是验证来自会话 cookie 的请求，将`HTTP-only`标志设置为`true`。这通常是 Spring Security 通过 JSESSIONID cookie 提供的。当然，为了让我们的 API 保持无状态，我们绝不能在服务器端使用会话。

**在这种情况下，我们的应用程序像有状态应用程序一样容易受到 CSRF 的攻击**:由于 cookie 会随任何 REST 请求自动发送，所以单击恶意链接就可以执行认证操作。

### 2.4.其他 CSRF 易受攻击的配置

一些配置不使用安全令牌作为凭证，但也可能容易受到 CSRF 攻击。

**这是 [HTTP 基本认证](https://web.archive.org/web/20220617075711/https://en.wikipedia.org/wiki/Basic_access_authentication)、 [HTTP 摘要认证](https://web.archive.org/web/20220617075711/https://en.wikipedia.org/wiki/Digest_access_authentication)、 [mTLS](https://web.archive.org/web/20220617075711/https://en.wikipedia.org/wiki/Mutual_authentication) 的情况。**

它们并不常见，但都有相同的缺点:浏览器会在任何 HTTP 请求上自动发送凭证。在这些情况下，我们必须启用 CSRF 保护。

## 3.在 Spring Boot 禁用 CSRF 保护

从版本 4 开始，Spring Security 默认启用 CSRF 保护。

如果我们的项目不需要它，我们可以在自定义的`WebSecurityConfigurerAdapter`中禁用它:

```java
@Configuration
public class SpringBootSecurityConfiguration 
  extends WebSecurityConfigurerAdapter {
    @Override
    public void configure(HttpSecurity http) throws Exception {
        http.csrf().disable();
    }
}
```

## 4.使用 REST API 启用 CSRF 保护

### 4.1.弹簧配置

如果我们的项目需要 CSRF 保护，**我们可以通过在自定义的`WebSecurityConfigurerAdapter`** 中使用`CookieCsrfTokenRepository`来发送带有 cookie 的 CSRF 令牌。

我们必须将`HTTP-only`标志设置为`false`,以便能够从我们的 JavaScript 客户端检索它:

```java
@Configuration
public class SpringSecurityConfiguration extends WebSecurityConfigurerAdapter {
    @Override
    public void configure(HttpSecurity http) throws Exception {
        http.csrf().csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse());
    }
}
```

**重启应用后，我们的请求收到 HTTP 错误，这意味着 CSRF 保护已启用。**

我们可以通过将日志级别调整为 DEBUG 来确认这些错误是从`CsrfFilter`类发出的:

```java
<logger name="org.springframework.security.web.csrf" level="DEBUG" />
```

它将显示:

```java
Invalid CSRF token found for http://...
```

**此外，我们应该在浏览器中看到一个新的`XSRF-TOKEN` cookie。**

让我们在 REST 控制器中添加几行代码，将信息写入 API 日志:

```java
CsrfToken token = (CsrfToken) request.getAttribute("_csrf");
LOGGER.info("{}={}", token.getHeaderName(), token.getToken());
```

### 4.2.客户端配置

在客户端应用程序中，在第一次 API 访问之后设置`XSRF-TOKEN` cookie。我们可以使用 JavaScript 正则表达式来检索它:

```java
const csrfToken = document.cookie.replace(/(?:(?:^|.*;\s*)XSRF-TOKEN\s*\=\s*([^;]*).*$)|^.*$/, '$1');
```

然后，我们必须向每个修改 API 状态的 REST 请求发送令牌:POST、PUT、DELETE 和 PATCH。

**春天期待在`X-XSRF-TOKEN`头**收到它。我们可以简单地用 JavaScript `Fetch` API 来设置它:

```java
fetch(url, {
    method: 'POST',
    body: JSON.stringify({ /* data to send */ }),
    headers: { 'X-XSRF-TOKEN': csrfToken },
})
```

现在，我们可以看到我们的请求正在工作，REST API 日志中的`“Invalid CSRF token”`错误已经消失。

因此，**攻击者将不可能执行 CSRF 攻击**。例如，试图从诈骗网站执行相同请求的脚本将收到`“Invalid CSRF token”`错误。

事实上，如果用户没有首先访问实际的网站，cookie 将不会被设置，请求将失败。

## 5.结论

在本文中，我们回顾了针对 REST API 的 CSRF 攻击可能或不可能发生的不同环境。

然后，我们学习了如何使用 Spring Security 启用或禁用 CSRF 保护。