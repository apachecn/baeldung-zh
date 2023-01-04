# 返回用户的自定义登录页面

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/custom-login-page-for-returning-user>

 ![](img/dbe454d4e624cc7723ad851551100ad4.png)

It’s just plain hard to get true, **real-time visibility into a running auth flow.**

Parts of the process can be completely hidden from us; if the complete authorization process requires a redirect from a remote OAuth production server, then every debugging effort must go through the production server.

It’s practically unfeasible to debug this locally. There’s no way to reproduce the exact state and no way to inspect what is actually happening under the hood. Not ideal.

Knowing these types of challenges, we built Lightrun - a real-time production debugging tool - to allow you to understand complicated flows with code-level information. Add logs, take snapshots (virtual breakpoints), and instrument metrics without a remote debugger, without stopping the running service, and, most importantly - **in real-time and without side effects**.

**Learn more with this 5-minute tutorial** focused on debugging these kinds of scenarios using Lightrun:

[>> Debugging Authentication and Authorization Using Lightrun](/web/20220524030013/https://www.baeldung.com/lightrun-n-security)

## 1。简介

本文是我们正在进行的春季安全系列 的 **[注册的延续。](/web/20220524030013/https://www.baeldung.com/spring-security-registration)**

在本文中，我们将看看如何为返回到我们应用程序的用户开发一个定制的登录页面。用户将收到一条标准的“欢迎…”消息。

## 2。添加长期 Cookie

**识别用户是否再次访问我们网站的一种方法是在用户成功登录后添加一个长期 cookie(例如 30 天)。**为了开发这个逻辑，我们需要实现我们的`AuthenticationSuccessHandler`，它在成功认证后添加 cookie。

让我们创建我们的自定义`MyCustomLoginAuthenticationSuccessHandler`并实现`onAuthenticationSuccess()`方法:

```java
@Override
public void onAuthenticationSuccess(final HttpServletRequest request,
  final HttpServletResponse response, final Authentication authentication)
  throws IOException {
    addWelcomeCookie(gerUserName(authentication), response);
    redirectStrategy.sendRedirect(request, response,
    "/homepage.html?user=" + authentication.getName());
}
```

这里的重点是对`addWelcomeCookie()`方法的调用。

现在，让我们看看添加 cookie 的代码:

```java
private String gerUserName(Authentication authentication) {
    return ((User) authentication.getPrincipal()).getFirstName();
}

private void addWelcomeCookie(String user, HttpServletResponse response) {
    Cookie welcomeCookie = getWelcomeCookie(user);
    response.addCookie(welcomeCookie);
}

private Cookie getWelcomeCookie(String user) {
    Cookie welcomeCookie = new Cookie("welcome", user);
    welcomeCookie.setMaxAge(60 * 60 * 24 * 30);
    return welcomeCookie;
}
```

我们已经用键`“welcome”`和当前用户的值`firstName`设置了一个 cookie。cookie 设置为 30 天后过期。

## 3。读取登录表单上的 Cookie

最后一步是每当登录表单加载时读取 cookie，如果存在，获取显示问候消息的值。我们可以用`Javascript.` 轻松做到这一点

首先，让我们添加占位符`“welcometext”`以在登录页面上显示我们的消息:

```java
<form name='f' action="login" method='POST' onsubmit="return validate();">
    <span id="welcometext"> </span>

    <br /><br />
    <label class="col-sm-4" th:text="#{label.form.loginEmail}">Email</label>
    <span class="col-sm-8">
      <input class="form-control" type='text' name='username' value=''/>
    </span>
    ...
</form>
```

现在，我们来看看相应的`Javascript`:

```java
function getCookie(name) {
    return document.cookie.split('; ').reduce((r, v) => {
        const parts = v.split('=')
        return parts[0] === name ? decodeURIComponent(parts[1]) : r
    }, '')
}

function display_username() {
    var username = getCookie('welcome');
    if (username) {
        document.getElementById("welcometext").innerHTML = "Welcome " + username + "!";
    }
}
```

第一个函数只是读取用户登录时设置的 cookie。如果 cookie 存在，第二个函数操纵 HTML 文档来设置欢迎消息。

函数`display_username()`在`HTML <body>`标签的`onload`事件上被调用:

```java
<body onload="display_username()">
```

## 4。结论

在这篇简短的文章中，我们看到了通过修改 Spring 中的默认认证流来定制用户体验是多么简单。基于这个简单的设置，可以完成许多复杂的事情。

本例的登录页面可以通过`/customLogin` URL 访问。这篇文章的完整代码可以在 GitHub 上找到。