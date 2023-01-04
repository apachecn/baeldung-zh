# 没有 web.xml 的 Java Web 应用程序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-web-app-without-web-xml>

## 1.概观

在本教程中，我们将使用 [Servlet 3.0+](https://web.archive.org/web/20221004093651/https://tomcat.apache.org/tomcat-7.0-doc/servletapi/index.html) 创建一个 Java web 应用程序。

我们将看看三个注释——`@WebServlet`、`@WebFilter`和`@WebListener`——它们可以帮助我们删除`web.xml`文件。

## 2.Maven 依赖性

为了使用这些新注释，我们需要包含`[javax.servlet-api](https://web.archive.org/web/20221004093651/https://search.maven.org/search?q=g:javax.servlet%20a:javax.servlet-api)`依赖项:

```java
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>4.0.1</version>
</dependency>
```

## 3.基于 XML 的配置

在 Servlet 3.0 之前，我们在一个`web.xml`文件中配置一个 Java web 应用程序:

```java
<web-app 
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://java.sun.com/xml/ns/javaee 
  http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
  version="2.5">
    <listener>
        <listener-class>com.baeldung.servlets3.web.listeners.RequestListener</listener-class>
    </listener>
    <servlet>
        <servlet-name>uppercaseServlet</servlet-name>
        <servlet-class>com.baeldung.servlets3.web.servlets.UppercaseServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>uppercaseServlet</servlet-name>
        <url-pattern>/uppercase</url-pattern>
    </servlet-mapping>
    <filter>
        <filter-name>emptyParamFilter</filter-name>
        <filter-class>com.baeldung.servlets3.web.filters.EmptyParamFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>emptyParamFilter</filter-name>
        <url-pattern>/uppercase</url-pattern>
    </filter-mapping>
</web-app>
```

让我们开始用 Servlet 3.0 中引入的相应注释替换每个配置部分。

## 4.servlet

JEE 6 附带了 Servlet 3.0，它使我们能够为 Servlet 定义使用注释，最大限度地减少了 web 应用程序对`web.xml`文件的使用。

**例如，我们可以定义一个 servlet 并用`@WebServlet`注释**来公开它

让我们为 URL 模式`/uppercase`定义一个 servlet。它将把 `input`请求参数的值转换成大写:

```java
@WebServlet(urlPatterns = "/uppercase", name = "uppercaseServlet")
public class UppercaseServlet extends HttpServlet {
    public void doGet(HttpServletRequest request, HttpServletResponse response) 
      throws IOException {
        String inputString = request.getParameter("input").toUpperCase();

        PrintWriter out = response.getWriter();
        out.println(inputString);
    }
}
```

**注意，我们为 servlet 定义了一个名称(`uppercaseServlet)`，现在我们可以引用它了。**我们将在下一节利用这一点。

使用`@WebServlet`注释，我们将替换`web.xml`文件中的`servlet`和`servlet-mapping`部分。

## 5.过滤

`Filter`是一个对象，用于拦截请求或响应，执行预处理或后处理任务。

**我们可以用`@WebFilter`注释定义一个过滤器。**

让我们创建一个过滤器来检查`input`请求参数是否存在:

```java
@WebFilter(urlPatterns = "/uppercase")
public class EmptyParamFilter implements Filter {
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse,
      FilterChain filterChain) throws IOException, ServletException {
        String inputString = servletRequest.getParameter("input");

        if (inputString != null && inputString.matches("[A-Za-z0-9]+")) {
            filterChain.doFilter(servletRequest, servletResponse);
        } else {
            servletResponse.getWriter().println("Missing input parameter");
        }
    }

    // implementations for other methods
}
```

使用`@WebFilter`注释，我们将替换`web.xml`文件中的`filter`和`filter-mapping`部分。

## 6.听众

我们经常需要基于某些事件来触发动作。这就是听众来帮忙的地方。这些对象将监听事件并执行我们指定的行为。

**像前面一样，我们可以用`@WebListener`注释定义一个监听器。**

让我们创建一个侦听器，它在每次执行对服务器的请求时进行计数。我们将实现`ServletRequestListener`，监听`ServletRequestEvent` s:

```java
@WebListener
public class RequestListener implements ServletRequestListener {
    @Override
    public void requestDestroyed(ServletRequestEvent event) {
        HttpServletRequest request = (HttpServletRequest)event.getServletRequest();
        if (!request.getServletPath().equals("/counter")) {
            ServletContext context = event.getServletContext();
            context.setAttribute("counter", (int) context.getAttribute("counter") + 1);
        }
    }

    // implementations for other methods
}
```

注意，我们排除了对 URL 模式`/counter`的请求。

使用`@WebListener`注释，我们替换了`web.xml`文件中的`listener`部分。

## 7.构建并运行

请注意，为了进行测试，我们为端点`/counter`添加了第二个 servlet，它只返回`counter` servlet 上下文属性。

所以，让我们用`Tomcat`作为应用服务器。

如果我们使用的是 3.1.0 之前的`maven-war-plugin`版本，我们需要将属性 [`failOnMissingWebXml`设置为`false`](/web/20221004093651/https://www.baeldung.com/eclipse-error-web-xml-missing) 。

现在，我们可以[将我们的`.war`文件部署到`Tomcat`](/web/20221004093651/https://www.baeldung.com/tomcat-deploy-war) ，并访问我们的 servlets。

让我们试试我们的`/uppercase` 端点:

```java
curl http://localhost:8080/spring-mvc-java/uppercase?input=texttouppercase

TEXTTOUPPERCASE
```

我们还应该看看我们的错误处理看起来如何:

```java
curl http://localhost:8080/spring-mvc-java/uppercase

Missing input parameter
```

最后，快速测试一下我们的听众:

```java
curl http://localhost:8080/spring-mvc-java/counter

Request counter: 2
```

## 8.仍然需要 XML

甚至，尽管 Servlet 3.0 中引入了所有的特性，但仍有一些用例需要一个`web.xml`文件，其中包括:

*   **我们不能用注释**来定义过滤器的顺序——如果我们有多个需要以特定顺序应用的过滤器，我们仍然需要`<filter-mapping>`部分
*   为了定义一个[会话超时](/web/20221004093651/https://www.baeldung.com/servlet-session-timeout)，我们仍然需要使用`<session-config>`部分
*   对于基于容器的授权，我们仍然需要`<security-role>`元素
*   为了指定欢迎文件，我们仍然需要一个`<welcome-file-list>`部分

或者，Servlet 3.0 也通过`ServletContainerInitializer` 引入了一些**编程支持，这也可以填补一些空白。**

## 9.结论

在本教程中，我们配置了一个 Java web 应用程序，但没有使用`web.xml`文件，而是使用了等效的注释。

一如既往，本教程的源代码可以在 [GitHub](https://web.archive.org/web/20221004093651/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-java) 上找到。此外，使用传统 web.xml 文件的应用程序也可以在 [GitHub](https://web.archive.org/web/20221004093651/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-xml) 上找到。

对于基于 Spring 的方法，请阅读我们的教程 [web.xml 与 Spring 初始化器。](/web/20221004093651/https://www.baeldung.com/spring-xml-vs-java-config)