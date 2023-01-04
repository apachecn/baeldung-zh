# 在 Spring URLs 中使用斜杠字符

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-slash-character-in-url>

## 1。简介

当我们开发 web 服务时，**我们可能需要处理可能包含斜线**的复杂或意外的 URL 路径。因此，我们可能会遇到我们正在使用的 web 服务器或框架的问题。

由于 Spring 提供的默认配置，它在这方面可能有点棘手。

在本教程中，**我们将展示一些在 Spring** 中处理带斜线的 URL 的常见解决方案和建议。我们还将了解为什么我们不应该使用一些常见的方法来解决这些问题。继续阅读，了解更多！

## 2。手动解析请求

在我们的 web 服务中，有时我们需要将某个路径下的所有请求映射到同一个端点。更糟糕的是，我们可能不知道剩下的路会是什么样子。我们可能还需要以某种方式接收这个路径作为参数，以便以后使用它。

假设我们可以通过`/mypaths`下的任何路径接收请求:

```
http://localhost:8080/mypaths/any/custom/path
```

假设我们想将所有这些不同的路径存储在一个数据库中，以了解我们正在接收哪些请求。

我们想到的第一个解决方案可能是将路径的动态部分捕获到一个`PathVariable`:

```
@GetMapping("mypaths/{anything}")
public String pathVariable(@PathVariable("anything") String anything) {
    return anything;
}
```

不幸的是，我们很快发现如果`PathVariable`包含斜杠，那么**将返回一个`404`。斜杠字符是 [URI 标准](https://web.archive.org/web/20220714113225/https://tools.ietf.org/html/rfc3986)路径分隔符，它之后的所有字符都被视为路径层次结构中的新级别。不出所料，Spring 遵循了这个标准。**

我们可以通过使用`**`通配符为特定路径下的所有请求创建一个回退来轻松解决这个问题:

```
@GetMapping("all/**")
public String allDirectories(HttpServletRequest request) {
    return request.getRequestURI()
        .split(request.getContextPath() + "/all/")[1];
}
```

然后，我们必须自己解析 URI，以获得我们感兴趣的路径部分。

当使用类似 URL 的参数时，这种解决方案非常方便，但是正如我们将在下一节中看到的，它对于其他一些情况是不够的。

## 3。使用查询参数

与我们之前的例子相反，在其他情况下，我们不仅映射不同的路径，还在 URL 中接收任何`String`作为参数。

让我们想象一下，在前面的例子中，我们用包含连续斜线的路径参数发出**请求:**

```
http://localhost:8080/all/http://myurl.com
```

起初，我们可能认为这应该可行，但我们很快意识到我们的控制器返回了`http:/myurl.com.` 这是因为 **Spring Security 规范了 URL，并用单个**替换了任何双斜杠。

Spring 也规范了 URL 中的其他序列，比如路径遍历。它采取这些预防措施**来防止恶意 URL 绕过已定义的安全约束**，正如在[官方 Spring 安全文档](https://web.archive.org/web/20220714113225/https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#request-matching)中解释的那样。

在这些情况下，强烈建议使用查询参数:

```
@GetMapping("all")
public String queryParameter(@RequestParam("param") String param) {
    return param;
}
```

这样，我们可以接收任何`String`参数而没有这些安全限制，我们的 web 服务将更加健壮和安全。

## 4。避免变通办法

我们给出的解决方案可能意味着我们的映射设计中的一些变化。这可能会诱使我们使用一些常见的变通办法，使我们的原始端点在接收 URL 中的斜杠时工作。

最常见的解决方法可能是在路径参数中编码斜线。然而，**过去曾报道过一些[安全漏洞](https://web.archive.org/web/20220714113225/https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2007-0450)，大多数网络和应用服务器[对此的反应](https://web.archive.org/web/20220714113225/https://tomcat.apache.org/security-6.html#Fixed_in_Apache_Tomcat_6.0.10)是默认禁止编码斜线**。仍然可以通过改变相应的设置来改变这种行为，就像在 [Tomcat](https://web.archive.org/web/20220714113225/https://tomcat.apache.org/tomcat-9.0-doc/config/systemprops.html#Security) 中一样。

其他人如 [Apache Server](https://web.archive.org/web/20220714113225/https://httpd.apache.org/docs/current/mod/core.html#allowencodedslashes) 走得更远，引入了一个选项，允许编码斜杠而不解码它们，这样它们就不会被解释为路径分隔符。在任何情况下，**这都是不推荐的，它会引入潜在的安全风险。**

另一方面，web 框架也采取了一些预防措施。正如我们之前所见， **Spring 增加了一些机制来防范不太严格的 servlet 容器**。因此，如果我们在服务器中允许编码斜线，我们仍然必须在 Spring 中允许它们。

最后，还有其他种类的解决方法，比如改变 Spring 默认提供的 URI 规范化。和以前一样，如果我们改变这些默认值，我们应该非常谨慎。

## 5。结论

在这篇短文中，我们展示了一些在 Spring 中处理 URL 中斜杠的解决方案。我们还介绍了一些安全性问题，如果我们改变服务器或框架(如 Spring)的默认配置，就会出现这些问题。

根据经验，查询参数通常是处理 URL 中斜杠的最佳解决方案。

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20220714113225/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-web-url)