# Servlet 重定向与转发

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/servlet-redirect-forward>

## 1.概观

有时，我们的 Java Servlet 中的初始 HTTP 请求处理程序需要将请求委托给另一个资源。在这些情况下，我们可以进一步转发请求，或者将其重定向到不同的资源。

我们将使用这两种机制，并讨论每种机制的差异和最佳实践。

## 2.Maven 依赖性

首先，让我们添加 Servlet Maven 依赖项:

```
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>4.0.0</version>
</dependency>
```

最新版本可以在这里找到[。](https://web.archive.org/web/20220626085710/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22javax.servlet%22%20AND%20a%3A%22javax.servlet-api%22)

## 3.向前

现在让我们直接开始，看看如何做一个简单的前进:

```
protected void doGet(HttpServletRequest req, HttpServletResponse resp) {
    RequestDispatcher dispatcher = getServletContext()
      .getRequestDispatcher("/forwarded");
    dispatcher.forward(req, resp);
}
```

**我们从父 Servlet 获得`RequestDispatcher`引用，并将其指向另一个服务器资源。**

简单来说，这将转发请求。

当客户端向`http://localhost:8081/hello?name=Dennis`提交请求时，该逻辑将运行，请求将被转发到`/forwarded`。

## 4.再直接的

现在我们已经了解了转发的概念，让我们来看看重定向的一个快速片段:

```
protected void doGet(HttpServletRequest req, HttpServletResponse resp){
    resp.sendRedirect(req.getContextPath() + "/redirected");
} 
```

**我们使用原始响应对象将该请求重定向到另一个 URL**::`/redirected”.`

当客户端向`http://localhost:8081/welcome?name=Dennis`提交请求时，该请求将被重定向到`http://localhost:8081/redirected.`

要了解更多关于在 Spring 环境中进行重定向的信息，请看我们的专用文章。

## 5.差异

我们在两种情况下都传递了带有值的参数“`name`”。**简单地说，转发的请求仍然带有这个值，但是重定向的请求没有。**

这是因为，使用重定向时，请求对象不同于原始对象。如果我们还想使用这个参数，我们需要把它保存在`HttpSession`对象中。

下面列出了 servlet 转发和重定向之间的主要区别:

**前进**:

*   该请求将在服务器端进一步处理
*   客户端不受转发的影响，浏览器中的 URL 保持不变
*   转发后，请求和响应对象将保持不变。请求范围对象仍然可用

**重定向**:

*   该请求被重定向到不同的资源
*   重定向后，客户端将看到 URL 发生了变化
*   创建了一个新请求
*   重定向通常用在 [Post/Redirect/Get](https://web.archive.org/web/20220626085710/https://en.wikipedia.org/wiki/Post/Redirect/Get) web 开发模式中

## 6.结论

转发和重定向都是将用户发送到不同的资源，尽管它们的语义非常不同。

在这两者之间进行选择很简单。如果需要前一个作用域，或者不需要通知用户，但是应用程序也想执行一个内部动作**，那么使用转发**。

要放弃该范围，或者如果新内容与原始请求不相关联——比如重定向到登录页面或完成表单提交—**,则使用重定向**。

和往常一样，示例代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220626085710/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-basics-2)