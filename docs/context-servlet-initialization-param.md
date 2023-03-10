# 上下文和 Servlet 初始化参数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/context-servlet-initialization-param>

## 1。概述

servlet 是运行在 servlet 容器中的普通 Java 类。

HTTP servlets(一种特定类型的 servlet)是 Java web 应用程序中的一等公民。HTTP servlets 的 API**旨在通过典型的请求-处理-响应周期处理 HTTP 请求，在客户端-服务器协议**中实现。

此外，servlets 可以使用请求/响应参数形式的键值对来控制客户机(通常是 web 浏览器)和服务器之间的交互。

这些参数可以被初始化并绑定到应用程序范围(上下文参数)和特定于 servlet 的范围(servlet 参数)。

在本教程中，我们将学习**如何定义和访问上下文和 servlet 初始化参数**。

## 2。初始化 Servlet 参数

我们可以使用注释和标准部署描述符——`“web.xml”`文件来定义和初始化 servlet 参数。值得注意的是，这两个选项并不相互排斥。

让我们深入探讨一下这些选项。

### 2.1。使用注释

**用注释初始化 servlets 参数允许我们将配置和源代码放在同一个地方**。

在这一节中，我们将演示如何使用注释定义和访问绑定到特定 servlet 的初始化参数。

为此，我们将实现一个简单的 `UserServlet`类，通过普通的 HTML 表单收集用户数据。

首先，让我们看看呈现表单的 JSP 文件:

```java
<!DOCTYPE html>
<html>
    <head>
        <title>Context and Initialization Servlet Parameters</title>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    </head>
    <body>
        <h2>Please fill the form below:</h2>
        <form action="${pageContext.request.contextPath}/userServlet" method="post">
            <label for="name"><strong>Name:</strong></label>
            <input type="text" name="name" id="name">
            <label for="email"><strong>Email:</strong></label>
            <input type="text" name="email" id="email">
            <input type="submit" value="Send">
        </form>
    </body>
</html> 
```

注意，我们已经通过使用 [EL](https://web.archive.org/web/20221126234222/https://mvnrepository.com/artifact/javax.el/el-api/2.2) (表达式语言)对表单的`action`属性进行了编码。这允许它总是指向`“/userServlet”`路径，而不管应用程序文件在服务器中的位置。

`“${pageContext.request.contextPath}”`表达式**为表单设置一个动态 URL，它总是相对于应用程序的上下文路径**。

下面是我们最初的 servlet 实现:

```java
@WebServlet(name = "UserServlet", urlPatterns = "/userServlet", initParams={
@WebInitParam(name="name", value="Not provided"), 
@WebInitParam(name="email", value="Not provided")}))
public class UserServlet extends HttpServlet {
    // ...    

    @Override
    protected void doGet(
      HttpServletRequest request, 
      HttpServletResponse response)
      throws ServletException, IOException {
        processRequest(request, response);
        forwardRequest(request, response, "/WEB-INF/jsp/result.jsp");
    }

    protected void processRequest(
      HttpServletRequest request, 
      HttpServletResponse response)
      throws ServletException, IOException {

        request.setAttribute("name", getRequestParameter(request, "name"));
        request.setAttribute("email", getRequestParameter(request, "email"));
    }

    protected void forwardRequest(
      HttpServletRequest request, 
      HttpServletResponse response, 
      String path)
      throws ServletException, IOException { 
        request.getRequestDispatcher(path).forward(request, response); 
    }

    protected String getRequestParameter(
      HttpServletRequest request, 
      String name) {
        String param = request.getParameter(name);
        return !param.isEmpty() ? param : getInitParameter(name);
    }
} 
```

在这种情况下，我们已经通过**使用`initParams`和`@WebInitParam`注释**定义了两个 servlet 初始化参数`name`和 `email`。

请注意，我们使用 HttpServletRequest 的`getParameter()`方法从 HTML 表单中检索数据，使用`getInitParameter()`方法访问 servlet 初始化参数。

`getRequestParameter()`方法检查`name`和`email`请求参数是否为空字符串。

如果它们是空字符串，那么它们被赋予匹配的初始化参数的默认值。

`doPost()`方法首先检索用户在 HTML 表单中输入的姓名和电子邮件(如果有的话)。然后它处理请求参数并将请求转发到一个`“result.jsp”`文件:

```java
<!DOCTYPE html>
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
        <title>User Data</title>
    </head>
    <body>
        <h2>User Information</h2>
        <p><strong>Name:</strong> ${name}</p>
        <p><strong>Email:</strong> ${email}</p>
    </body>
</html>
```

如果我们将示例 web 应用程序部署到一个应用服务器，比如 [Apache Tomcat、](https://web.archive.org/web/20221126234222/https://tomcat.apache.org/) [Oracle GlassFish](https://web.archive.org/web/20221126234222/http://www.oracle.com/technetwork/middleware/glassfish/overview/index.html) 或[JBoss widfly](https://web.archive.org/web/20221126234222/http://www.wildfly.org/)，并运行它，它应该首先显示 HTML 表单页面。

一旦用户填写了`name`和`email`字段并提交了表单，它将输出数据:

```java
User Information
Name: the user's name
Email: the user's email 
```

如果表单是空白的，它将显示 servlet 初始化参数:

```java
User Information 
Name: Not provided 
Email: Not provided 
```

在这个例子中，我们已经向**展示了如何使用注释定义 servlet 初始化参数，以及如何使用`g` `etInitParameter()`方法**访问它们。

### 2.2。使用标准部署描述符

**这种方法不同于使用注释的方法，因为它允许我们将配置和源代码相互隔离**。

为了展示如何用`“web.xml”`文件定义初始化 servlet 参数，让我们首先从`UserServlet`类中移除`initParam`和`@WebInitParam`注释:

```java
@WebServlet(name = "UserServlet", urlPatterns = {"/userServlet"}) 
public class UserServlet extends HttpServlet { ... } 
```

接下来，让我们在`“web.xml”`文件中定义 servlet 初始化参数:

```java
<?xml version="1.0" encoding="UTF-8"?>
<web-app  

  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd" version="3.1">
    <servlet>
        <display-name>UserServlet</display-name>
        <servlet-name>UserServlet</servlet-name>
        <init-param>
            <param-name>name</param-name>
            <param-value>Not provided</param-value>
        </init-param>
        <init-param>
            <param-name>email</param-name>
            <param-value>Not provided</param-value>
        </init-param>
    </servlet>
</web-app> 
```

如上所示，使用`“web.xml”`文件定义 servlet 初始化参数可以归结为使用`<init-param>,` `<param-name>`和`<param-value>`标签。

此外，只要我们坚持上述标准结构，就可以根据需要定义任意多的 servlet 参数。

当我们将应用程序重新部署到服务器并重新运行它时，它的行为应该与使用注释的版本相同。

## 3。初始化上下文参数

有时我们需要定义一些不可变的数据，这些数据必须在 web 应用程序中全局共享和访问。

由于数据的全局性质，我们应该**使用应用程序范围的上下文初始化参数来存储数据，而不是求助于 servlet 对应物**。

尽管不可能使用注释定义上下文初始化参数，但我们可以在`“web.xml”`文件中这样做。

假设我们想为应用程序运行所在的国家和省份提供一些默认的全局值。

我们可以通过几个上下文参数来实现这一点。

让我们相应地重构`“web.xml”`文件:

```java
<web-app 

  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee 
  http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd" version="3.1">
    <context-param>
        <param-name>province</param-name>
        <param-value>Mendoza</param-value>
    </context-param>
    <context-param>
        <param-name>country</param-name>
        <param-value>Argentina</param-value>
    </context-param>
    <!-- Servlet initialization parameters -->
</web-app>
```

这一次，我们使用了`<context-param>, <param-name>,`和`<param-value>`标签来定义`province`和`country`上下文参数。

当然，我们需要重构`UserServlet`类，以便它可以获取这些参数并将它们传递给结果页面。

以下是 servlet 的相关部分:

```java
@WebServlet(name = "UserServlet", urlPatterns = {"/userServlet"})
public class UserServlet extends HttpServlet {
    // ...

    protected void processRequest(
      HttpServletRequest request, 
      HttpServletResponse response)
      throws ServletException, IOException {

        request.setAttribute("name", getRequestParameter(request, "name"));
        request.setAttribute("email", getRequestParameter(request, "email"));
        request.setAttribute("province", getContextParameter("province"));
        request.setAttribute("country", getContextParameter("country"));
    }

    protected String getContextParameter(String name) {-
        return getServletContext().getInitParameter(name);
    }
} 
```

请注意`getContextParameter()`方法的实现，其中**首先通过`getServletContext(),`获取 servlet 上下文，然后用`getInitParameter()`方法**获取上下文参数。

接下来，我们需要重构`“result.jsp”`文件，以便它可以显示上下文参数以及特定于 servlet 的参数:

```java
<p><strong>Name:</strong> ${name}</p>
<p><strong>Email:</strong> ${email}</p>
<p><strong>Province:</strong> ${province}</p>
<p><strong>Country:</strong> ${country}</p> 
```

最后，我们可以重新部署应用程序并再次执行它。

如果用户在 HTML 表单中填写姓名和电子邮件，那么它将显示这些数据以及上下文参数:

```java
User Information 
Name: the user's name 
Email: the user's email
Province: Mendoza
Country: Argentina 
```

否则，它将输出 servlet 和上下文初始化参数:

```java
User Information 
Name: Not provided 
Email: Not provided
Province: Mendoza 
Country: Argentina 
```

虽然这个例子是虚构的，但是它向**展示了如何使用上下文初始化参数来存储不可变的全局数据**。

由于数据被绑定到应用程序上下文，而不是特定的 servlet，我们可以使用`getServletContext()`和`getInitParameter()`方法从一个或多个 servlet 访问它们。

## 4。结论

在本文中，**我们学习了上下文和 servlet 初始化参数的关键概念，以及如何使用注释和`“web.xml”`文件**来定义和访问它们。

很长一段时间以来，Java 中有一种强烈的趋势，即尽可能摆脱 XML 配置文件并迁移到注释。

仅举几个例子，CDI 、[春天](https://web.archive.org/web/20221126234222/https://spring.io/)、[冬眠](https://web.archive.org/web/20221126234222/http://hibernate.org/)，就是这方面的突出例子。

然而，使用`“web.xml”`文件来定义上下文和 servlet 初始化参数并没有什么本质上的错误。

尽管 [Servlet API](https://web.archive.org/web/20221126234222/https://search.maven.org/classic/#search%7Cga%7C1%7Cservlet-api) 已经朝着这个趋势快速发展，**我们仍然需要使用部署描述符来定义上下文初始化参数**。

像往常一样，本文中显示的所有代码示例都可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221126234222/https://github.com/eugenp/tutorials/tree/master/web-modules/javax-servlets)