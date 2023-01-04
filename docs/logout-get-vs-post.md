# 注销应该是 GET 还是 POST？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/logout-get-vs-post>

## 1.概观

在传统的 web 应用程序中，登录通常需要向服务器发送用户名和密码进行身份验证。虽然理论上这些元素可以是 GET 请求中的 URL 参数，但显然将它们封装到 POST 请求中要好得多。

但是，既然不需要发送任何敏感信息，那么注销应该可以通过 GET 请求实现吗？

在本教程中，我们将研究这种设计考虑的各个方面。

## 2.服务器端会话

当我们管理服务器端会话时，我们必须公开一个端点来销毁这些会话。由于 GET 方法的简单性，我们可能会倾向于使用它。当然，这在技术上可行，但可能会导致一些不良行为。

有一些像 [web 加速器](https://web.archive.org/web/20221009140904/https://www.nginx.com/resources/glossary/web-acceleration/)这样的进程会为用户预取 GET 链接。预取的目的是在用户点击链接时立即提供内容，从而减少页面加载时间。这些过程假设[GET 链接严格用于返回内容](https://web.archive.org/web/20221009140904/https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods)，而不是改变任何东西的状态。

如果我们将注销公开为一个 GET 请求，并以一个链接的形式呈现，这些进程可能会在试图预取页面上的链接时无意中让用户注销。

如果我们的注销 URL 不是静态可用的，例如由 javascript 确定，这可能不是问题。然而， [HTTP/1.1 RFC](https://web.archive.org/web/20221009140904/https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html) 明确声明 GET 方法应该只用于返回内容，用户不能对 GET 请求的任何副作用负责。我们应该尽可能遵循这一建议。

相反，RFC 将 POST 方法描述为可以向数据处理过程(注销)提交数据(我们的会话或会话 ID)的方法。这是对我们试图实现的目标的更恰当的描述。

### 2.1.春天安全

默认情况下，Spring Security 要求注销请求的类型是 POST。然而，当我们禁用 [CSRF 保护](/web/20221009140904/https://www.baeldung.com/spring-security-csrf)时，我们可以让 Spring 使用 GET logout 请求:

```java
protected void configure(HttpSecurity http) throws Exception {
    http.csrf().disable()
    ...
}
```

## 3.无状态 REST

当我们管理无状态 REST 会话时,“注销”的概念发生了变化。在无状态环境中，每个请求都包括整个会话。因此，可以通过简单地使用 Javascript 丢弃会话而不是发送任何类型的请求来“注销”。

然而，出于安全原因，仍然应该通知服务器注销操作，以便将被撤销的 JWT 列入黑名单。这可以防止会话在“注销”后被使用

即使在无状态环境中，我们仍然必须在注销时向服务器发送请求。因为这种请求的目的不是检索内容，所以不应该通过 GET 发出。相反，应该将会话发送到服务器，并明确表示要注销。

## 4.结论

在这个简短的讨论中，我们简要地看了常见的设计问题，即注销应该是 GET 还是 POST。

我们考虑了这个问题的各个方面:

*   语义上，GET 请求不应该有任何有状态副作用
*   用户可能会在浏览器中运行一些包含预取链接的进程。如果注销发生在 GET 之上，预取过程可能会在用户登录后无意中将其注销
*   即使是无状态会话也应该向服务器报告注销事件，这应该通过 POST 请求来完成