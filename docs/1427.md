# Java Servlets 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/intro-to-servlets>

## 1。概述

在本文中，我们将了解 Java web 开发的一个核心方面——servlet。

## 2。Servlet 和容器

简单地说，Servlet 是一个处理请求、处理请求并回复响应的类。

例如，我们可以使用 Servlet 通过 HTML 表单收集用户的输入，从数据库中查询记录，并动态地创建网页。

Servlet 由另一个名为 **`Servlet Container.`** 的 Java 应用程序控制。当运行在 web 服务器上的应用程序收到请求`,` 时，服务器将请求传递给 Servlet 容器，Servlet 容器再将请求传递给目标 Servlet。

## 3。 **美芬依存**

为了在我们的 web 应用程序中添加 Servlet 支持，`javax`。`servlet-api`需要依赖关系:

```
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>3.1.0</version>
</dependency>
```

最新的 maven 依赖可以在[这里](https://web.archive.org/web/20221126234027/https://mvnrepository.com/artifact/javax.servlet/servlet-api)找到。

当然，我们还必须配置一个 Servlet 容器来部署我们的应用程序；这是一个开始如何对 Tomcat 发动战争的好地方。

## 4。Servlet 生命周期

让我们来看一下定义 Servlet 生命周期的一组方法。

### 4.1。`init()`

`init`方法被设计成只被调用一次。如果 servlet 的实例不存在，则 web 容器:

1.  加载 servlet 类
2.  创建 servlet 类的实例
3.  通过调用`init`方法初始化它

在 servlet 可以接收任何请求之前，`init`方法必须成功完成。如果`init`方法抛出一个`ServletException`或者在 Web 服务器定义的时间段内没有返回，servlet 容器就不能将 servlet 投入使用。

```
public void init() throws ServletException {
    // Initialization code like set up database etc....
}
```

### 4.2。`service()`

只有在 servlet 的`init()`方法成功完成`.`之后，才会调用该方法

容器调用`service()`方法来处理来自客户端的请求，解释 HTTP 请求类型(`GET`、`POST`、`PUT`、`DELETE`等)。)并调用`doGet`、`doPost`、`doPut`、`doDelete`等。适当的方法。

```
public void service(ServletRequest request, ServletResponse response) 
  throws ServletException, IOException {
    // ...
}
```

### 4.3。`destroy()`

由 Servlet 容器调用，使 Servlet 停止服务。

只有当 servlet 的`service`方法中的所有线程都退出时，或者超时时间过去后，才会调用该方法。容器调用这个方法后，不会在 Servlet 上再次调用`service`方法。

```
public void destroy() {
    // 
}
```

## 5。示例 Servlet

首先，[将上下文根](/web/20221126234027/https://www.baeldung.com/tomcat-root-application)从 `javax-servlets-1.0-SNAPSHOT`更改为/ add:

```
<Context path="/" docBase="javax-servlets-1.0-SNAPSHOT"></Context>
```

在`$CATALINA_HOME\conf\server.xml.`中的`Host`标签下

**现在让我们建立一个使用表单处理信息的完整示例**。

首先，让我们定义一个带有映射`/calculateServlet`的 servlet，它将捕获表单发布的信息，并使用 [RequestDispatcher](https://web.archive.org/web/20221126234027/https://docs.oracle.com/javaee/6/api/javax/servlet/RequestDispatcher.html) 返回结果:

```
@WebServlet(name = "FormServlet", urlPatterns = "/calculateServlet")
public class FormServlet extends HttpServlet {

    @Override
    protected void doPost(HttpServletRequest request, 
      HttpServletResponse response)
      throws ServletException, IOException {

        String height = request.getParameter("height");
        String weight = request.getParameter("weight");

        try {
            double bmi = calculateBMI(
              Double.parseDouble(weight), 
              Double.parseDouble(height));

            request.setAttribute("bmi", bmi);
            response.setHeader("Test", "Success");
            response.setHeader("BMI", String.valueOf(bmi));

            request.getRequestDispatcher("/WEB-INF/jsp/index.jsp").forward(request, response);
        } catch (Exception e) {
           request.getRequestDispatcher("/WEB-INF/jsp/index.jsp").forward(request, response);
        }
    }

    private Double calculateBMI(Double weight, Double height) {
        return weight / (height * height);
    }
}
```

如上图，用 [`@WebServlet`](https://web.archive.org/web/20221126234027/https://docs.oracle.com/javaee/6/api/javax/servlet/annotation/WebServlet.html) 标注的类必须扩展 [`javax.servlet.http.HttpServlet`](https://web.archive.org/web/20221126234027/https://docs.oracle.com/javaee/7/api/javax/servlet/http/HttpServlet.html) 类。需要注意的是`[@WebServlet](https://web.archive.org/web/20221126234027/https://docs.oracle.com/javaee/6/api/javax/servlet/annotation/WebServlet.html)`注释只在 Java EE 6 以后才可用。

`[@WebServlet](https://web.archive.org/web/20221126234027/https://docs.oracle.com/javaee/6/api/javax/servlet/annotation/WebServlet.html)` 注释在部署时由容器处理，相应的 servlet 在指定的 URL 模式下可用。值得注意的是，通过使用注释来定义 URL 模式，我们可以避免使用名为`web.xml`的 XML 部署描述符来进行 Servlet 映射。

如果我们希望映射不带注释的 Servlet，我们可以使用传统的`web.xml`来代替:

```
<web-app ...>

    <servlet>
       <servlet-name>FormServlet</servlet-name>
       <servlet-class>com.root.FormServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>FormServlet</servlet-name>
        <url-pattern>/calculateServlet</url-pattern>
    </servlet-mapping>

</web-app>
```

接下来，让我们创建一个基本的 HTML `form`:

```
<form name="bmiForm" action="calculateServlet" method="POST">
    <table>
        <tr>
            <td>Your Weight (kg) :</td>
            <td><input type="text" name="weight"/></td>
        </tr>
        <tr>
            <td>Your Height (m) :</td>
            <td><input type="text" name="height"/></td>
        </tr>
        <th><input type="submit" value="Submit" name="find"/></th>
        <th><input type="reset" value="Reset" name="reset" /></th>
    </table>
    <h2>${bmi}</h2>
</form>
```

最后，为了确保一切按预期运行，让我们也编写一个快速测试:

```
public class FormServletLiveTest {

    @Test
    public void whenPostRequestUsingHttpClient_thenCorrect() 
      throws Exception {

        HttpClient client = new DefaultHttpClient();
        HttpPost method = new HttpPost(
          "http://localhost:8080/calculateServlet");

        List<BasicNameValuePair> nvps = new ArrayList<>();
        nvps.add(new BasicNameValuePair("height", String.valueOf(2)));
        nvps.add(new BasicNameValuePair("weight", String.valueOf(80)));

        method.setEntity(new UrlEncodedFormEntity(nvps));
        HttpResponse httpResponse = client.execute(method);

        assertEquals("Success", httpResponse
          .getHeaders("Test")[0].getValue());
        assertEquals("20.0", httpResponse
          .getHeaders("BMI")[0].getValue());
    }
}
```

## 6。Servlet、HttpServlet 和 JSP

理解 Servlet 技术并不局限于 HTTP 协议是很重要的。

实际上几乎总是这样，但是`[Servlet](https://web.archive.org/web/20221126234027/https://docs.oracle.com/javaee/7/api/javax/servlet/Servlet.html)`是一个通用接口，而`[HttpServlet](https://web.archive.org/web/20221126234027/https://docs.oracle.com/javaee/7/api/javax/servlet/http/HttpServlet.html)`是该接口的扩展——增加了 HTTP 特定的支持——例如`doGet`和`doPost`等。

最后，Servlet 技术也是许多其他 web 技术的主要驱动力，例如[JSP–Java server Pages](/web/20221126234027/https://www.baeldung.com/jsp)，Spring MVC 等等。

## 7。结论

在这篇简短的文章中，我们介绍了 Java web 应用程序中 Servlets 的基础。

示例项目可以像 GitHub 项目一样被下载和运行。