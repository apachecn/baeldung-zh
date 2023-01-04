# Jakarta EE Servlet 异常处理

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/servlet-exceptions>

## 1.介绍

在本教程中，我们将在一个 [Jakarta EE Servlet](/web/20220627085836/https://www.baeldung.com/intro-to-servlets) 应用程序中处理异常——以便在发生错误时提供一个优雅的预期结果。

## 2.雅加达 EE Servlet 异常

首先，我们将使用 [API 注释](https://web.archive.org/web/20220627085836/https://tomcat.apache.org/tomcat-9.0-doc/servletapi/index.html)定义一个 Servlet(查看 [Servlets Intro](/web/20220627085836/https://www.baeldung.com/intro-to-servlets) 了解更多细节),默认的`GET`处理器将抛出一个异常:

```java
@WebServlet(urlPatterns = "/randomError")
public class RandomErrorServlet extends HttpServlet {

    @Override
    protected void doGet(
      HttpServletRequest req, 
      HttpServletResponse resp) {
        throw new IllegalStateException("Random error");
    }
}
```

## 3.默认错误处理

现在让我们简单地将应用程序部署到我们的 servlet 容器中(我们将假设应用程序在`http://localhost:8080/javax-servlets`下运行)。

当我们访问地址`http://localhost:8080/javax-servlets/randomError` **，**时，我们会看到默认的 servlet 错误处理已经就绪:

[![servlet](img/6147298711f559ceb714c80f2ef1bae0.png)](/web/20220627085836/https://www.baeldung.com/wp-content/uploads/2018/06/servlet.png)

默认的错误处理由 servlet 容器提供，可以在[容器](https://web.archive.org/web/20220627085836/https://tomcat.apache.org/tomcat-7.0-doc/config/valve.html#Error_Report_Valve)或应用程序级别定制。

## 4.自定义错误处理

我们可以使用一个`web.xml`文件描述符来定义自定义错误处理，其中我们可以定义以下类型的策略:

*   **状态代码错误处理**–它允许我们将 HTTP 错误代码([客户端](https://web.archive.org/web/20220627085836/https://en.wikipedia.org/wiki/List_of_HTTP_status_codes#4xx_Client_errors)和[服务器](https://web.archive.org/web/20220627085836/https://en.wikipedia.org/wiki/List_of_HTTP_status_codes#5xx_Server_errors))映射到静态 HTML 错误页面或错误处理 servlet
*   **异常类型错误处理**–它允许我们将异常类型映射到静态 HTML 错误页面或错误处理 servlet

### 4.1.用 HTML 页面处理状态代码错误

我们可以在`web.xml`中为 HTTP 404 错误设置自定义错误处理策略:

```java
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 

  xsi:schemaLocation="http://java.sun.com/xml/ns/javaee 
    http://java.sun.com/xml/ns/javaee/web-app_3_1.xsd"
  version="3.1">

    <error-page>
        <error-code>404</error-code>
        <location>/error-404.html</location> <!-- /src/main/webapp/error-404.html-->
    </error-page>

</web-app> 
```

现在，从浏览器访问`http://localhost:8080/javax-servlets/invalid.html`,获得静态 HTML 错误页面。

### 4.2.用 Servlet 处理异常类型错误

我们可以在`web.xml`中为`java.lang.Exception`设置自定义错误处理策略:

```java
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 

  xsi:schemaLocation="http://java.sun.com/xml/ns/javaee 
    http://java.sun.com/xml/ns/javaee/web-app_3_1.xsd"
  version="3.1">
    <error-page> 
        <exception-type>java.lang.Exception</exception-type> 
        <location>/errorHandler</location> 
    </error-page>
</web-app>
```

在`ErrorHandlerServlet`中，我们可以使用请求中提供的[错误属性](https://web.archive.org/web/20220627085836/https://tomcat.apache.org/tomcat-7.0-doc/servletapi/constant-values.html)来访问错误细节:

```java
@WebServlet(urlPatterns = "/errorHandler")
public class ErrorHandlerServlet extends HttpServlet {

    @Override
    protected void doGet(
      HttpServletRequest req, 
      HttpServletResponse resp) throws IOException {

        resp.setContentType("text/html; charset=utf-8");
        try (PrintWriter writer = resp.getWriter()) {
            writer.write("<html><head><title>Error description</title></head><body>");
            writer.write("<h2>Error description</h2>");
            writer.write("<ul>");
            Arrays.asList(
              ERROR_STATUS_CODE, 
              ERROR_EXCEPTION_TYPE, 
              ERROR_MESSAGE)
              .forEach(e ->
                writer.write("<li>" + e + ":" + req.getAttribute(e) + " </li>")
            );
            writer.write("</ul>");
            writer.write("</html></body>");
        }
    }
} 
```

现在，我们可以访问`http://localhost:8080/javax-servlets/randomError`来查看定制错误 servlet 的工作情况。

**注意**:我们在`web.xml`中定义的异常类型太宽泛，我们应该更详细地指定我们想要处理的所有异常。

我们还可以在我们的`ErrorHandlerServlet`组件中使用[容器提供的 servlet 记录器](https://web.archive.org/web/20220627085836/https://javaee.github.io/javaee-spec/javadocs/javax/servlet/ServletContext.html#log-java.lang.String-)来记录额外的细节:

```java
Exception exception = (Exception) req.getAttribute(ERROR_EXCEPTION);
if (IllegalArgumentException.class.isInstance(exception)) {
    getServletContext()
      .log("Error on an application argument", exception);
}
```

了解 servlet 提供的日志机制之外还有什么是值得的，查看 slf4j 上的[指南了解更多细节。](/web/20220627085836/https://www.baeldung.com/slf4j-with-log4j2-logback)

## 5.结论

在这篇简短的文章中，我们看到了 servlet 应用程序中的默认错误处理和指定的自定义错误处理，而没有添加外部组件或库。

和往常一样，您可以在[servlet 教程库](https://web.archive.org/web/20220627085836/https://github.com/eugenp/tutorials/tree/master/javax-servlets)上找到源代码。