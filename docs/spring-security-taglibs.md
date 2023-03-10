# Spring 安全标记库简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-taglibs>

## 1。概述

在本教程中，我们将看一看 [Spring Security Taglibs](https://web.archive.org/web/20221129000745/https://docs.spring.io/spring-security/site/docs/5.0.7.RELEASE/reference/htmlsingle/#taglibs) ，它为访问安全信息和在 JSP 中应用安全约束提供了基本支持。

## 2。Maven 依赖关系

首先，让我们将 [spring-security-taglibs](https://web.archive.org/web/20221129000745/https://search.maven.org/search?q=g:org.springframework.security%20AND%20a:spring-security-taglibs&core=gav) 依赖项添加到我们的`pom.xml`:

```java
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-taglibs</artifactId>
    <version>5.2.2.RELEASE</version>
</dependency>
```

## 3。声明标记库

现在，在使用标记之前，我们需要在 JSP 文件的顶部导入标记库:

```java
<%@ taglib prefix="sec" uri="http://www.springframework.org/security/tags" %>
```

添加这个之后，我们将能够用前缀`sec `指定 Spring Security 的标签。

## 4。`authorize`标签

### 4.1。`access`表情

在我们的应用程序中，我们可能会有只为某些角色或用户显示的信息。

在这种情况下，我们可以使用`authorize`标签:

```java
<sec:authorize access="!isAuthenticated()">
  Login
</sec:authorize>
<sec:authorize access="isAuthenticated()">
  Logout
</sec:authorize>
```

此外，我们可以检查经过身份验证的用户是否有特定的角色:

```java
<sec:authorize access="hasRole('ADMIN')">
    Manage Users
</sec:authorize>
```

我们可以使用任何一个弹簧安全表达式作为`access`的值:

*   如果当前用户具有任何列出的角色，则`hasAnyRole(‘ADMIN','USER')`返回`true`
*   如果当前主体是匿名用户，则`isAnonymous()`返回`true`
*   如果当前主体是“记住我”用户，则`isRememberMe()`返回`true`
*   如果用户已经过身份验证，并且既不是匿名用户也不是“记住我”用户，则`isFullyAuthenticated()`返回`true`

### 4.2。`url`

除此之外，我们可以检查被授权向特定 URL 发送请求的用户:

```java
<sec:authorize url="/userManagement">
    <a href="/userManagement">Manage Users</a>
</sec:authorize>
```

### 4.3。调试

可能有些情况下，我们希望对 UI 有更多的控制，例如在测试场景中。我们可以在比如说我们的`application.properties`文件中设置`spring.security.disableUISecurity` = `true`,而不是让 Spring Security 跳过呈现这些未经授权的部分。

**当我们这样做的时候，`authorize`标签不会隐藏它的内容。**相反，它会用`<span class=”securityHiddenUI”>… </span>`标签来包装内容。然后，我们可以用一些 CSS 自定义渲染。

请记住，通过 CSS 隐藏内容是不安全的！用户只需查看源文件就可以看到未经授权的内容。

## 5。`authentication`标签

在其他时候，我们希望显示登录用户的详细信息，比如说“欢迎回来，Carol！”在网站上。

为此，我们使用了`authentication `标签:

```java
<sec:authorize access="isAuthenticated()">
    Welcome Back, <sec:authentication property="name"/>
</sec:authorize>
```

## 6。`csrfInput`标签

希望我们的应用程序中启用了 Spring Security 的 CSRF 防御功能！

如果我们这样做了，那么 Spring Security 已经为我们在`<form:form>`标签中插入了一个 CSRF 隐藏表单输入。

但是如果我们想使用`<form>`，**，我们可以使用`csrfInput` :** 手动指示 Spring Security 应该在哪里放置这个隐藏的输入字段

```java
<form method="post" action="/do/something">
    <sec:csrfInput />
    Text Field:<br />
    <input type="text" name="textField" />
</form>
```

如果 CSRF 保护未启用，此标签不输出任何内容。

## 7。`csrfMetaTags`标签

或者，**如果我们想要在 Javascript 中访问 CSRF 令牌，**我们可能想要将令牌作为元标签插入。

我们可以用`csrfMetaTags `标签来做到这一点:

```java
<html>
    <head>
        <title>JavaScript with CSRF Protection</title>
        <sec:csrfMetaTags />
        <script type="text/javascript" language="javascript">
            var csrfParameter = $("meta[name='_csrf_parameter']").attr("content");
            var csrfHeader = $("meta[name='_csrf_header']").attr("content");
            var csrfToken = $("meta[name='_csrf']").attr("content");
        </script>
    </head>
    <body>
    ...
    </body>
</html>
```

同样，如果 CSRF 保护没有启用，这个标签不会输出任何东西。

## 8。结论

在这篇简短的文章中，我们关注了一些常见的 Spring 安全 taglib 用例。

正如我们所知，它们对于呈现认证和授权感知的 JSP 内容非常有用。

所有的例子都可以在 Github 上找到[。](https://web.archive.org/web/20221129000745/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-security)