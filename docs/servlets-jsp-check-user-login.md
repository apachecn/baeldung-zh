# 检查用户是否使用 Servlets 和 JSP 登录

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/servlets-jsp-check-user-login>

## 1.概观

在本教程中，我们将学习如何检查用户的登录，并确保用户已经用有效的凭证填写了登录表单并开始了会话。然而，我们将在不使用 [Spring Security](/web/20220706232948/https://www.baeldung.com/spring-security-login) 和仅使用 JSP 和 [servlets](/web/20220706232948/https://www.baeldung.com/register-servlet) 的情况下做到这一点。因此，我们需要一个 servlet 容器来支持它，比如 Tomcat 9。

到最后，我们将会很好地理解事情是如何在引擎盖下工作的。

## 2.持久性策略

首先，我们需要用户。为了简单起见，我们将使用预加载的地图。让我们用`User`来定义它:

```java
public class User {
    static HashMap<String, User> DB = new HashMap<>();
    static {
        DB.put("user", new User("user", "pass"));
        // ...
    }

    private String name;
    private String password;

    // getters and setters
}
```

## 3.过滤请求

我们将首先创建一个[过滤器](/web/20220706232948/https://www.baeldung.com/intercepting-filter-pattern-in-java)来检查无会话请求，阻止对我们的 servlets 的直接访问:

```java
@WebFilter("/*")
public class UserCheckFilter implements Filter {

    public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain) {
        // ...
        request.setAttribute("origin", request.getRequestURI());

        if (!request.getRequestURI().contains("login") && request.getSession(false) == null) {
            forward(request, response, "/login.jsp");
            return;
        }

        chain.doFilter(request, response);
    }
}
```

**在这里，通过在`@WebFilter`上将`/*`定义为我们的 URL 模式，所有请求将首先通过我们的过滤器。**然后，如果还没有会话，我们将请求重定向到我们的登录页面，存储`origin`供以后使用。最后，我们提前返回，防止 servlet 在没有正确会话的情况下进行处理。

## 4.用 JSP 创建登录表单

为了构建我们的登录表单，我们需要从 [JSTL](/web/20220706232948/https://www.baeldung.com/jstl) 导入核心 Taglib。同样，让我们在`page`指令中将`session`属性设置为`false`。因此，不会自动创建新会话，我们可以完全控制:

```java
<%@ page session="false"%>
<%@ taglib uri="http://java.sun.com/jstl/core_rt" prefix="c"%>

<form action="login" method="POST">
    ...
</form> 
```

然后，在我们的`form`中，我们将有一个隐藏的输入来保存`origin`:

```java
<input type="hidden" name="origin" value="${origin}">
```

接下来，我们将包含一个条件元素来输出错误:

```java
<c:if test="${not empty error}">
    * error: ${error} 
</c:if>
```

最后，让我们添加一些`input`标签，以便用户可以输入和提交凭证:

```java
<input type="text" name="name">
<input type="password" name="password"> 
<input type="submit">
```

## 5.设置我们的登录 Servlet

在我们的 servlet 中，如果请求是一个`GET`，我们将把它转发到我们的登录表单。**最重要的是，如果是`POST`** ，我们会验证登录:

```java
@WebServlet("/login")
public class UserCheckLoginServlet extends HttpServlet {
    // ...
} 
```

因此，在我们的`doGet()`方法中，我们将只重定向到我们的登录 JSP，向前传递`origin`:

```java
protected void doGet(HttpServletRequest request, HttpServletResponse response) {
    String referer = (String) request.getAttribute("origin");
    request.setAttribute("origin", referer);
    forward(request, response, "/login.jsp");
}
```

在我们的`doPost()`中，我们验证凭证并创建一个会话，将`User`对象向前传递并重定向到`origin`:

```java
protected void doPost(HttpServletRequest request, HttpServletResponse response) {
    String key = request.getParameter("name");
    String pass = request.getParameter("password");

    User user = User.DB.get(key);
    if (!user.getPassword().equals(pass)) {
        request.setAttribute("error", "invalid login");
        forward(request, response, "/login.jsp");
        return;
    }

    HttpSession session = request.getSession();
    session.setAttribute("user", user);

    response.sendRedirect(request.getParameter("origin"));
}
```

**在无效凭证的情况下，我们在我们的`error`变量**中设置一条消息。否则，我们用我们的`User`对象更新会话。

## 6.正在检查登录信息

最后，让我们创建我们的主页。它只显示会话信息，并有一个注销链接:

```java
<body>
    current session info: ${user.name}

    <a href="logout">logout</a>
</body> 
```

我们的 home servlet 所做的就是将`User`转发到主页:

```java
@WebServlet("/home")
public class UserCheckServlet extends HttpServlet {

    protected void doGet(HttpServletRequest request, HttpServletResponse response) {
        User user = (User) session.getAttribute("user");
        request.setAttribute("user", user);

        forward(request, response, "/home.jsp");
    }
}
```

这是它看起来的样子:

[![](img/d6a3e6ff137e554cff924f575199ace3.png)](/web/20220706232948/https://www.baeldung.com/wp-content/uploads/2022/02/login-success.png)

## 7.注销

**要注销，我们只需使当前会话无效并重定向到主页**。之后，我们的`UserCheckFilter`将检测到一个无会话请求，并将我们重定向回登录页面，重新开始这个过程:

```java
@WebServlet("/logout")
public class UserCheckLogoutServlet extends HttpServlet {

    protected void doGet(HttpServletRequest request, HttpServletResponse response) {
        request.getSession().invalidate();

        response.sendRedirect("./");
    }
}
```

## 8.结论

在本文中，我们经历了完整登录周期的创建。我们看到了如何使用单个过滤器完全控制对 servlets 的访问。简而言之，使用这种方法，我们可以始终确保在我们需要的地方有一个有效的会话。类似地，我们可以扩展该机制来实现更好的访问控制。

和往常一样，源代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220706232948/https://github.com/eugenp/tutorials/tree/master/javax-servlets-2)