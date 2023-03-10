# 调试 Spring MVC 404“没有为 HTTP 请求找到映射”错误

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-mvc-404-error>

## 1.介绍

Spring MVC 是使用前端控制器模式构建的传统应用程序。`[DispatcherServlet](/web/20220926184643/https://www.baeldung.com/spring-dispatcherservlet),` 充当前端控制器，负责路由和请求处理。

与任何 web 应用程序或网站一样，当无法找到请求的资源时，Spring MVC 会返回 HTTP 404 响应代码。在本教程中，我们将看看 Spring MVC 中 404 错误的**常见原因。**

## 2.404 响应的可能原因

### 2.1.错误的 URI

假设我们有一个映射到`/greeting`并呈现`greeting.jsp`的`GreetingController `:

```java
@Controller
public class GreetingController {

    @RequestMapping(value = "/greeting", method = RequestMethod.GET)
    public String get(ModelMap model) {
        model.addAttribute("message", "Hello, World!");
        return "greeting";
    }
}
```

相应的视图呈现了`message`变量的值:

```java
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
    <head>
        <title>Greeting</title>
    </head>
    <body>
        <h2>${message}</h2>
    </body>
</html>
```

正如所料，向`/greeting`发出一个 GET 请求是可行的:

```java
curl http://localhost:8080/greeting
```

我们将看到一个 HTML 页面，显示消息“Hello World”:

```java
<html>
    <head>
        <title>Greeting</title>
    </head>
    <body>
        <h2>Hello, World!</h2>
    </body>
</html>
```

看到 404 最常见的原因之一是使用了不正确的 URI。例如，向`/greetings`而不是`/greeting`发出 GET 请求是错误的:

```java
curl http://localhost:8080/greetings
```

在这种情况下，我们会在服务器日志中看到一条警告消息:

```java
[http-nio-8080-exec-6] WARN  o.s.web.servlet.PageNotFound - 
  No mapping found for HTTP request with URI [/greetings] in DispatcherServlet with name 'mvc'
```

客户端会看到一个错误页面:

```java
<html>
    <head>
        <title>Home</title>
    </head>
    <body>
        <h1>Http Error Code : 404\. Resource not found</h1>
    </body>
</html>
```

为了避免这种情况，我们需要确保我们已经正确地进入了 URI。

### 2.2.不正确的 Servlet 映射

如前所述，`DispatcherServlet`是 Spring MVC 中的前端控制器。因此，就像在标准的基于 servlet 的应用程序中一样，我们需要使用`web.xml`文件为 servlet 创建一个映射。

我们在`servlet `标签中定义 servlet，并将其映射到`servlet-mapping`标签中的 URI。我们需要确保`url-pattern`的值是正确的，因为经常会看到 servlet 映射到“/*”的建议**——注意后面的星号**:

```java
<?xml version="1.0" encoding="UTF-8"?>
<web-app ...>
    <!-- Additional config omitted -->
    <servlet>
        <servlet-name>mvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>mvc</servlet-name>
        <url-pattern>/*</url-pattern>
    </servlet-mapping>
    <!-- Additional config omitted -->
</web-app>
```

现在，如果我们请求`/greeting`**，我们会在服务器日志中看到一条警告:**

```java
curl http://localhost:8080/greeting
```

```java
WARN  o.s.web.servlet.PageNotFound - No mapping found for HTTP request with URI 
  [/WEB-INF/view/greeting.jsp] in DispatcherServlet with name 'mvc'
```

这一次错误声明没有找到`greeting.jsp`，用户看到一个空白页。

**为了修复这个错误，我们需要将`DispatcherServlet `映射到“/”(后面没有星号)来代替:**

```java
<servlet-mapping>
    <servlet-name>mvc</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

修复映射后，一切都应该正常工作。请求`/greeting`现在显示消息“你好，世界！”：

```java
curl http://localhost:8080/greeting
```

```java
<html>
    <head>
        <title>Greeting</title>
    </head>
    <body>
        <h2>Hello, World!</h2>
    </body>
</html>
```

问题背后的原因是，如果我们将`DispatcherServlet`映射到/* **，**，那么我们是在告诉应用程序，到达我们应用程序的每个请求都将由`DispatcherServlet`来服务。然而，这不是一个正确的方法，因为`DispatcherServlet`没有能力做到这一点。相反，Spring MVC 期望一个`ViewResolver`的实现来服务 JSP 文件之类的视图。

## 3.结论

在这篇简短的文章中，我们解释了如何在 Spring MVC 中调试 404 错误。我们讨论了从 Spring 应用程序收到 404 响应的两个最常见的原因。第一个错误是在发出请求时使用了不正确的 URI。第二个是将`DispatcherServlet`映射到`web.xml`中错误的`url-pattern`。

一如既往，本教程的完整实现可以在 Github 上找到[。](https://web.archive.org/web/20220926184643/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-xml-2)**