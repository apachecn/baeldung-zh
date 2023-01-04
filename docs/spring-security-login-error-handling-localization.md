# 登录 Spring Web 应用程序——错误处理和本地化

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-login-error-handling-localization>

## 1。概述

在本文中，我们将展示如何使用 Spring MVC 为一个在后端使用 Spring Security 处理身份验证的应用程序实现简单的登录页面。

关于如何使用 Spring Security 处理登录的完整细节，[这里有一篇文章](/web/20220628092148/https://www.baeldung.com/spring-security-login)深入探讨了它的配置和实现。

## 2。登录页面

让我们从定义一个非常简单的登录页面开始:

```java
<html>
<head></head>
<body>
   <h1>Login</h1>
   <form name='f' action="login" method='POST'>
      <table>
         <tr>
            <td>User:</td>
            <td><input type='text' name='username' value=''></td>
         </tr>
         <tr>
            <td>Password:</td>
            <td><input type='password' name='password' /></td>
         </tr>
         <tr>
            <td><input name="submit" type="submit" value="submit" /></td>
         </tr>
      </table>
  </form>
</body>
</html>
```

现在，让我们包含一个客户端检查，以确保在提交表单之前已经输入了`username`和`password`。对于这个例子，我们将使用普通的 Javascript，但是 JQuery 也是一个很好的选择:

```java
<script type="text/javascript">
function validate() {
    if (document.f.username.value == "" && document.f.password.value == "") {
        alert("Username and password are required");
        document.f.username.focus();
        return false;
    }
    if (document.f.username.value == "") {
        alert("Username is required");
        document.f.username.focus();
        return false;
    }
    if (document.f.password.value == "") {
	alert("Password is required");
	document.f.password.focus();
        return false;
    }
}
</script>
```

如您所见，我们简单地检查了`username`或`password`字段是否为空；如果是——会弹出一个 javascript 消息框，显示相应的消息。

## 3。消息本地化

接下来，让我们本地化我们在前端使用的消息。此类消息有多种类型，每种类型都以不同的方式本地化:

1.  在表单被 Spring 的控制器或处理程序处理之前**生成的消息。这些消息可以在 JSP 页面中引用，并使用 **Jsp/Jslt 本地化**进行本地化(参见第 4.3 节)。)**
2.  页面提交给 Spring 处理后本地化的消息(在提交`login`表单之后)；这些消息使用 **Spring MVC 本地化**进行本地化(参见 4.2 节)。)

### 3.1。`message.properties`文件

无论哪种情况，我们都需要为我们想要支持的每种语言创建一个`message.properties`文件；文件的名字应该遵循这个惯例:`messages_[localeCode].properties`。

例如，如果我们想要支持英语和西班牙语的错误消息，我们将拥有文件: **`messages_en.properties`** 和 **`messages_es_ES.properties`** 。注意，对于英语来说——`messages.properties`也是有效的。

我们将把这两个文件放在项目的类路径(`src/main/resources`)中。这些文件只包含我们需要以不同语言显示的错误代码和消息，例如:

```java
message.username=Username required
message.password=Password required
message.unauth=Unauthorized access!!
message.badCredentials=Invalid username or password
message.sessionExpired=Session timed out
message.logoutError=Sorry, error login out
message.logoutSucc=You logged out successfully
```

### 3.2。配置 Spring MVC 本地化

Spring MVC 提供了一个 `LocaleResolver`,它与它的 `LocaleChangeInterceptor` API 一起工作，使得根据地区设置以不同的语言显示消息成为可能。为了配置本地化——我们需要在 MVC 配置中定义以下 beans】

```java
@Override
public void addInterceptors(InterceptorRegistry registry) {
    LocaleChangeInterceptor localeChangeInterceptor = new LocaleChangeInterceptor();
    localeChangeInterceptor.setParamName("lang");
    registry.addInterceptor(localeChangeInterceptor);
}

@Bean
public LocaleResolver localeResolver() {
    CookieLocaleResolver cookieLocaleResolver = new CookieLocaleResolver();
    return cookieLocaleResolver;
} 
```

默认情况下，区域设置解析器将从 HTTP 头中获取区域设置代码。为了强制使用默认的语言环境，我们需要在`localeResolver()`上设置它:

```java
@Bean
public LocaleResolver localeResolver() {
    CookieLocaleResolver cookieLocaleResolver = new CookieLocaleResolver();
    cookieLocaleResolver.setDefaultLocale(Locale.ENGLISH);
    return cookieLocaleResolver;
}
```

这个语言环境解析器是一个`CookieLocaleResolver`，这意味着它将语言环境信息存储在客户端 cookie 中；因此——它会在用户每次登录时以及整个访问过程中记住用户的语言环境。

或者，有一个 `SessionLocaleResolver`，它会在整个会话过程中记住语言环境。要使用这个`LocaleResolver`来代替，我们需要用下面的方法替换上面的方法:

```java
@Bean
public LocaleResolver localeResolver() {
    SessionLocaleResolver sessionLocaleResolver = new SessionLocaleResolver();
    return sessionLocaleResolver;
}
```

最后，请注意，`LocaleChangeInterceptor`将根据通过简单链接与登录页面一起发送的`lang`参数的值来更改区域设置:

```java
<a href="?lang=en">English</a> |
<a href="?lang=es_ES">Spanish</a> 
```

### 3.3。JSP/JSLT 本地化

JSP/JSLT API 将用于显示 JSP 页面本身捕获的本地化消息。要使用 jsp 本地化库，我们应该向`pom.xml`添加以下依赖项:

```java
<dependency>
    <groupId>javax.servlet.jsp</groupId>
    <artifactId>javax.servlet.jsp-api</artifactId>
    <version>2.3.2-b01</version>
</dependency>
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>jstl</artifactId>
    <version>1.2</version>
</dependency>
```

## 4。显示错误信息

### 4.1。登录验证错误

为了使用 JSP/JSTL 支持并在`login.jsp`中显示本地化消息，让我们在页面中实现以下更改:

1.将以下标记库元素添加到 `login.jsp`:

```java
<%@ taglib uri="http://java.sun.com/jsp/jstl/fmt" prefix="fmt"%>
```

2.添加指向`messages.properties`文件的 jsp/jslt 元素:

```java
<fmt:setBundle basename="messages" />
```

3.添加以下`<fmt:…>`元素来存储 jsp 变量上的消息:

```java
<fmt:message key="message.password" var="noPass" />
<fmt:message key="message.username" var="noUser" />
```

4。修改我们在第 3 节中看到的登录验证脚本，以便本地化错误消息:

```java
<script type="text/javascript">
function validate() {
    if (document.f.username.value == "" && document.f.password.value == "") {
        alert("${noUser} and ${noPass}");
	document.f.username.focus();
	return false;
    }
    if (document.f.username.value == "") {
	alert("${noUser}");
	document.f.username.focus();
	return false;
     }
     if (document.f.password.value == "") {
	alert("${noPass}");
	document.f.password.focus();
	return false;
     }
}
</script>
```

### 4.2。预登录错误

有时，如果前面的操作失败，登录页面会被传递一个错误参数。例如，注册表单提交按钮将加载登录页面。如果注册成功，那么最好在登录表单中显示一条成功消息，如果相反，则显示一条错误消息。

在下面的示例`login` 表单中，我们通过截取 `regSucc`和`regError`参数来实现这一点，并根据它们的值显示本地化的消息。

```java
<c:if test="${param.regSucc == true}">
    <div id="status">
	<spring:message code="message.regSucc">    
        </spring:message>
    </div>
</c:if>
<c:if test="${param.regError == true}">
    <div id="error">
        <spring:message code="message.regError">   
        </spring:message>
    </div>
</c:if>
```

### 4.3。登录安全错误

如果登录过程由于某种原因失败，Spring Security 将重定向到一个登录错误 URL，我们已经定义为`/login.html?error=true`。

因此，与我们在页面中显示注册状态的方式类似，我们需要在出现登录问题时做同样的事情:

```java
<c:if test="${param.error != null}">
    <div id="error">
        <spring:message code="message.badCredentials">   
        </spring:message>
    </div>
</c:if>
```

注意，我们使用了一个 `<spring:message …>`元素。这意味着错误消息是在 Spring MVC 处理过程中生成的。

完整的登录页面——包括 js 验证和这些附加的状态消息可以在 [github 项目](https://web.archive.org/web/20220628092148/https://github.com/Baeldung/spring-security-registration)中找到。

### 4.4。注销错误

在下面的例子中， `logout.html`页面中的 jsp 代码`<c:if test=”${not empty SPRING_SECURITY_LAST_EXCEPTION}”>` 将检查注销过程中是否有错误。

例如，如果自定义注销处理程序在重定向到注销页面之前试图存储用户数据时出现持久性异常。虽然这些错误很少发生，但是我们也应该尽可能干净利落地处理它们。

让我们来看看完整的`logout.jsp`:

```java
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<%@ taglib prefix="sec"
    uri="http://www.springframework.org/security/tags"%>
<%@taglib uri="http://www.springframework.org/tags" prefix="spring"%>
<c:if test="${not empty SPRING_SECURITY_LAST_EXCEPTION}">
    <div id="error">
        <spring:message code="message.logoutError">    
        </spring:message>
    </div>
</c:if>
<c:if test="${param.logSucc == true}">
    <div id="success">
	<spring:message code="message.logoutSucc">    
        </spring:message>
    </div>
</c:if>
<html>
<head>
<title>Logged Out</title>
</head>
<body>	
    <a href="login.html">Login</a>
</body>
</html>
```

请注意，注销页面还读取查询字符串 param `logSucc`，如果其值等于`true`，将显示本地化成功消息。

## 5。春季安全配置

本文的重点是登录过程的前端，而不是后端——所以我们将只简要地看一下安全配置的要点；对于完整的配置，你应该阅读[以前的文章](/web/20220628092148/https://www.baeldung.com/spring-security-login)。

### 5.1。重定向到登录错误 URL

`<form-login…/>`元素中的以下指令将应用程序流定向到将处理登录错误的 url:

```java
authentication-failure-url="/login.html?error=true"
```

### 5.2。注销成功重定向

```java
<logout 
  invalidate-session="false" 
  logout-success-url="/logout.html?logSucc=true" 
  delete-cookies="JSESSIONID" />
```

`logout-success-url`属性简单地重定向到注销页面，带有一个确认注销成功的参数。

## 6。结论

在本文中，我们展示了如何为 Spring 安全支持的应用程序实现登录页面——处理登录验证、显示认证错误和消息本地化。

我们将在下一篇文章中研究完整的注册实现——目标是为生产做好登录和注册过程的完整实现。