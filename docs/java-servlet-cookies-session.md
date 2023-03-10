# 在 Java Servlet 中处理 Cookies 和会话

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-servlet-cookies-session>

## 1。概述

在本教程中，我们将使用 servlet 讲述**在 Java 中处理 cookies 和会话。**

此外，我们将简要描述什么是 cookie，并探索它的一些示例用例。

## 2。Cookie 基础知识

简单地说，**cookie 是存储在客户端的一小段数据，服务器在与客户端通信时使用这些数据**。

它们用于在发送后续请求时识别客户端。它们还可以用于将一些数据从一个 servlet 传递到另一个 servlet。

更多详情请参考本文的[。](https://web.archive.org/web/20220708154643/https://pl.wikipedia.org/wiki/HTTP_cookie)

### 2.1。创建一个 Cookie

在 `**javax.servlet.http**`包中定义了`Cookie` 类。

要将它发送给客户端，我们需要**创建一个并将其添加到响应**:

```java
Cookie uiColorCookie = new Cookie("color", "red");
response.addCookie(uiColorCookie); 
```

然而，它的 API 要宽泛得多——让我们来探索一下。

### 2.2。设置 Cookie 到期日期

我们可以设置最大年龄(使用方法`maxAge(int)`)，它定义了给定 cookie 的有效秒数:

```java
uiColorCookie.setMaxAge(60*60); 
```

我们将最大年龄设置为一小时。过了这段时间，客户端(浏览器)在发送请求时就不能使用 cookie 了，也应该从浏览器缓存中删除它。

### 2.3。设置 Cookie 域

`Cookie` API 中另一个有用的方法是`setDomain(String)`。

这允许我们指定域名，它应该由客户交付。这也取决于我们是否明确指定域名。

让我们为 cookie 设置域:

```java
uiColorCookie.setDomain("example.com");
```

cookie 将被发送到由`example.com`及其子域发出的每个请求。

**如果我们不明确指定一个域，它将被设置为创建 cookie** 的域名 **。**

例如，如果我们从`example.com`创建一个 cookie，并保留域名为空，那么它将被传送到`www.example.com`(没有子域)。

除了域名，我们还可以指定路径。接下来让我们来看看。

### 2.4。设置 Cookie 路径

该路径指定了 cookie 将被传送到的位置。

**如果我们显式地指定一个路径，那么一个`Cookie`将被传递到给定的 URL 及其所有子目录:**

```java
uiColorCookie.setPath("/welcomeUser");
```

隐式地，它将被设置为创建 cookie 及其所有子目录的 URL。

现在让我们关注如何在`Servlet`中检索它们的值。

### 2.5。读取 Servlet 中的 Cookies

客户端将 Cookies 添加到请求中。客户端检查其参数，并决定是否可以将它传递到当前 URL。

我们可以通过对传递给`Servlet`的请求(`HttpServletRequest`)调用`getCookies()`来获得所有的 cookies。

我们可以遍历这个数组并搜索我们需要的数组，例如，通过比较它们的名称:

```java
public Optional<String> readCookie(String key) {
    return Arrays.stream(request.getCookies())
      .filter(c -> key.equals(c.getName()))
      .map(Cookie::getValue)
      .findAny();
}
```

### 2.6。移除一个 Cookie

**为了** **从浏览器中删除一个 cookie，我们必须向响应中添加一个同名的新 cookie，但是将`maxAge`值设置为 0** :

```java
Cookie userNameCookieRemove = new Cookie("userName", "");
userNameCookieRemove.setMaxAge(0);
response.addCookie(userNameCookieRemove);
```

删除 cookies 的一个示例用例是用户注销操作——我们可能需要删除为活动用户会话存储的一些数据。

现在我们知道了如何在`Servlet`中处理 cookies。

接下来，我们将讨论另一个重要的对象，我们经常从一个`Servlet`—`Session`对象访问它。

## 3。`HttpSession`对象

`HttpSession` 是跨不同请求存储用户相关数据的另一个选项。会话是保存上下文数据的服务器端存储。

不同会话对象之间不共享数据(客户端只能从其会话中访问数据)。它还包含键-值对，但是与 cookie 相比，会话可以包含 object 作为值。存储实现机制是依赖于服务器的。

会话通过 cookie 或请求参数与客户端匹配。更多信息可在这里找到[。](https://web.archive.org/web/20220708154643/https://en.wikipedia.org/wiki/Session_(computer_science))

### 3.1。获取会话

我们可以直接从请求中获得`HttpSession`:

```java
HttpSession session = request.getSession(); 
```

上面的代码将创建一个新的会话，以防它不存在。我们可以通过拨打以下电话达到同样的目的:

```java
request.getSession(true)
```

如果我们只想获得现有会话，而不想创建新会话，我们需要使用:

```java
request.getSession(false) 
```

如果我们第一次访问 JSP 页面，那么默认情况下会创建一个新的会话。我们可以通过将`session`属性设置为`false:`来禁用这种行为

```java
<%@ page contentType="text/html;charset=UTF-8" session="false" %>
```

在大多数情况下，web 服务器使用 cookies 进行会话管理。创建会话对象时，服务器会创建一个带有`JSESSIONID`键和值的 cookie 来标识会话。

### 3.2。`Session`属性

session 对象提供了一系列访问(创建、读取、修改、删除)为给定用户会话创建的属性的方法:

*   `setAttribute(String, Object)`用一个键和一个新值创建或替换一个会话属性
*   `getAttribute(String)`读取具有给定名称(键)的属性值
*   `removeAttribute(String)`删除给定名称的属性

我们还可以通过调用`getAttributeNames()`轻松检查已经存在的会话属性。

正如我们已经提到的，我们可以从请求中检索会话对象。当我们已经有了它，我们就可以快速执行上面提到的方法。

我们可以创建一个属性:

```java
HttpSession session = request.getSession();
session.setAttribute("attributeKey", "Sample Value"); 
```

属性值可以通过其键(名称)获得:

```java
session.getAttribute("attributeKey"); 
```

我们可以删除不再需要的属性:

```java
session.removeAttribute("attributeKey"); 
```

用户会话的一个众所周知的用例是，当用户从我们的网站注销时，它存储的所有数据都将失效。会话对象为它提供了一个解决方案:

```java
session.invalidate(); 
```

这个方法从 web 服务器中删除了整个会话，所以我们不能再从它那里访问属性。

object 有更多的方法，但我们提到的这个是最常见的。

## 4。结论

在本文中，我们讨论了两种机制，这两种机制允许我们在对服务器的后续请求之间存储用户数据 cookie 和会话。

请记住，HTTP 协议是无状态的，因此跨请求维护状态是必须的。

和往常一样，代码片段可以在 Github 上找到[。](https://web.archive.org/web/20220708154643/https://github.com/eugenp/tutorials/tree/master/javax-servlets)