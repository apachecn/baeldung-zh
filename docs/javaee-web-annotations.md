# Java EE Web 相关注释指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/javaee-web-annotations>

## 1。概述

Java EE 注释允许开发人员指定应用程序组件在容器中的行为方式，从而简化了开发人员的工作。这些是 XML 描述符的现代替代品，基本上可以避免样板代码。

在本文中，我们将关注 Java EE 7 中 Servlet API 3.1 引入的注释。我们将研究它们的用途和用法。

## 2。网页注释

Servlet API 3.1 引入了一组新的注释类型，可以在`Servlet`类中使用:

*   `@WebServlet`
*   `@WebInitParam`
*   `@WebFilter`
*   `@WebListener`
*   `@ServletSecurity`
*   `@HttpConstraint`
*   `@HttpMethodConstraint`
*   `@MultipartConfig`

我们将在下一节中详细研究它们。

## 3。`@WebServlet`

简单地说，这个注释允许我们将 Java 类声明为 servlet*:*

```java
@WebServlet("/account")
public class AccountServlet extends javax.servlet.http.HttpServlet {

    public void doGet(HttpServletRequest request, HttpServletResponse response) 
      throws IOException {
        // ...
    }

    public void doPost(HttpServletRequest request, HttpServletResponse response) 
      throws IOException {
        // ...
    }
}
```

### 3.1。使用属性`@WebServlet`标注

`@WebServlet`有一组允许我们定制 servlet 的属性:

*   `name`
*   `description`
*   `urlPatterns`
*   `initParams`

我们可以使用这些，如下例所示:

```java
@WebServlet(
  name = "BankAccountServlet", 
  description = "Represents a Bank Account and it's transactions", 
  urlPatterns = {"/account", "/bankAccount" }, 
  initParams = { @WebInitParam(name = "type", value = "savings")})
public class AccountServlet extends javax.servlet.http.HttpServlet {

    String accountType = null;

    public void init(ServletConfig config) throws ServletException {
        // ...
    }

    public void doGet(HttpServletRequest request, HttpServletResponse response) 
      throws IOException {
        // ... 
    }

    public void doPost(HttpServletRequest request, HttpServletResponse response) 
      throws IOException {
        // ...  
    }
}
```

`name`属性覆盖了缺省 servlet 名称，缺省情况下它是完全限定的类名。如果我们想描述 servlet 做什么，我们可以使用`description`属性。

`urlPatterns`属性用于指定 servlet 可用的 URL(如代码示例所示，可以为该属性提供多个值)。

## 4。`@WebInitParam`

该注释与`@WebServlet`注释的`initParams`属性和 servlet 的初始化参数一起使用。

在本例中，我们将 servlet 初始化参数`type`设置为‘savings’值:

```java
@WebServlet(
  name = "BankAccountServlet", 
  description = "Represents a Bank Account and it's transactions", 
  urlPatterns = {"/account", "/bankAccount" }, 
  initParams = { @WebInitParam(name = "type", value = "savings")})
public class AccountServlet extends javax.servlet.http.HttpServlet {

    String accountType = null;

    public void init(ServletConfig config) throws ServletException {
        accountType = config.getInitParameter("type");
    }

    public void doPost(HttpServletRequest request, HttpServletResponse response) 
      throws IOException {
        // ...
    }
}
```

## 5.`@WebFilter`

如果我们想改变 servlet 的请求和响应而不触及其内部逻辑，我们可以使用`WebFilter`注释。通过指定 URL 模式，我们可以将过滤器与一个 servlet 或一组 servlet 和静态内容相关联。

在下面的例子中，我们使用 `@WebFilter`注释将任何未授权的访问重定向到登录页面:

```java
@WebFilter(
  urlPatterns = "/account/*",
  filterName = "LoggingFilter",
  description = "Filter all account transaction URLs")
public class LogInFilter implements javax.servlet.Filter {

    public void init(FilterConfig filterConfig) throws ServletException {
    }

    public void doFilter(
        ServletRequest request, ServletResponse response, FilterChain chain) 
          throws IOException, ServletException {
        HttpServletRequest req = (HttpServletRequest) request;
        HttpServletResponse res = (HttpServletResponse) response;

        res.sendRedirect(req.getContextPath() + "/login.jsp");
        chain.doFilter(request, response);
    }

    public void destroy() {
    }

}
```

## 6。`@WebListener`

如果我们想要了解或控制 servlet 及其请求如何以及何时被初始化或更改，我们可以使用`@WebListener`注释。

要编写 web 侦听器，我们需要扩展以下一个或多个接口:

*   `ServletContextListener – for` 关于`ServletContext` 生命周期的通知
*   `ServletContextAttributeListener – for` 当`ServletContext` 属性改变时的通知
*   每当发出资源请求时的通知
*   `ServletRequestAttributeListener – for` 在`ServletRequest`中添加、删除或更改属性时的通知
*   `HttpSessionListener – for` 创建和销毁新会话时的通知
*   `HttpSessionAttributeListener – for` 在会话中添加或删除新属性时的通知

下面是一个我们如何使用`ServletContextListener` 来配置 web 应用程序的例子:

```java
@WebListener
public class BankAppServletContextListener 
  implements ServletContextListener {

    public void contextInitialized(ServletContextEvent sce) { 
        sce.getServletContext().setAttribute("ATTR_DEFAULT_LANGUAGE", "english"); 
    } 

    public void contextDestroyed(ServletContextEvent sce) { 
        // ... 
    } 
}
```

## 7。`@ServletSecurity`

当我们想要为我们的 servlet 指定安全模型时，包括角色、访问控制和认证需求，我们使用注释`@ServletSecurity`。

在这个例子中，我们将使用`@ServletSecurity` 注释来限制对我们的`AccountServlet` 的访问:

```java
@WebServlet(
  name = "BankAccountServlet", 
  description = "Represents a Bank Account and it's transactions", 
  urlPatterns = {"/account", "/bankAccount" }, 
  initParams = { @WebInitParam(name = "type", value = "savings")})
@ServletSecurity(
  value = @HttpConstraint(rolesAllowed = {"Member"}),
  httpMethodConstraints = {@HttpMethodConstraint(value = "POST", rolesAllowed = {"Admin"})})
public class AccountServlet extends javax.servlet.http.HttpServlet {

    String accountType = null;

    public void init(ServletConfig config) throws ServletException {
        // ...
    }

    public void doGet(HttpServletRequest request, HttpServletResponse response) 
      throws IOException {
       // ...
    }

    public void doPost(HttpServletRequest request, HttpServletResponse response) 
      throws IOException {        
        double accountBalance = 1000d;

        String paramDepositAmt = request.getParameter("dep");
        double depositAmt = Double.parseDouble(paramDepositAmt);
        accountBalance = accountBalance + depositAmt;

        PrintWriter writer = response.getWriter();
        writer.println("<html> Balance of " + accountType + " account is: " + accountBalance 
        + "</html>");
        writer.flush();
    }
}
```

在这种情况下，当调用`AccountServlet,` 时，浏览器弹出一个登录屏幕，让用户输入有效的用户名和密码。

我们可以使用`@HttpConstraint`和`@HttpMethodConstraint`注释来指定`@ServletSecurity`注释的属性`value` 和`httpMethodConstraints,` 的值。

`@HttpConstraint` 注释适用于所有 HTTP 方法。换句话说，它指定了默认的安全约束。

`@HttpConstraint` 有三个属性:

*   `value`
*   `rolesAllowed`
*   `transportGuarantee`

在这些属性中，最常用的属性是`rolesAllowed`。在上面的示例代码片段中，属于角色`Member` 的用户被允许调用所有 HTTP 方法。

注释允许我们指定特定 HTTP 方法的安全约束。

`@HttpMethodConstraint` 具有以下属性:

*   `value`
*   `emptyRoleSemantic`
*   `rolesAllowed`
*   `transportGuarantee`

在上面的示例代码片段中，它显示了如何将`doPost`方法限制为仅用于属于`Admin`角色的用户，从而允许存款功能仅由`Admin`用户完成。

## 8。`@MultipartConfig`

当我们需要注释一个 servlet 来处理`multipart/form-data`请求时，就会用到这个注释(通常用于文件上传 servlet)。

这将暴露出`HttpServletRequest` 的`getParts()` 和`getPart(name)` 方法可用于访问所有部件以及单个部件。

上传的文件可以通过调用零件对象的`write(fileName)`写入磁盘。

现在我们将看一个示例 servlet `UploadCustomerDocumentsServlet` 来演示它的用法:

```java
@WebServlet(urlPatterns = { "/uploadCustDocs" })
@MultipartConfig(
  fileSizeThreshold = 1024 * 1024 * 20,
  maxFileSize = 1024 * 1024 * 20,
  maxRequestSize = 1024 * 1024 * 25,
  location = "./custDocs")
public class UploadCustomerDocumentsServlet extends HttpServlet {

    protected void doPost(HttpServletRequest request, HttpServletResponse response) 
      throws ServletException, IOException {
        for (Part part : request.getParts()) {
            part.write("myFile");
        }
    }

}
```

`@MultipartConfig`有四个属性:

*   `fileSizeThreshold` –这是临时保存上传文件时的大小阈值。如果上传文件的大小大于此阈值，它将被存储在磁盘上。否则，文件存储在内存中(以字节为单位)
*   `maxFileSize` –这是上传文件的最大大小(以字节为单位)
*   `maxRequestSize` –这是请求的最大大小，包括上传的文件和其他表单数据(字节大小)
*   `location` –是存储上传文件的目录

## 9。结论

在本文中，我们研究了 Servlet API 3.1 中引入的一些 Java EE 注释及其用途和用法。

与本文相关的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221128121617/https://github.com/eugenp/tutorials/tree/master/web-modules/jee-7)