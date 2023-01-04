# Servlet 和 Servlet 容器简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-servlets-containers-intro>

## 1.概观

在本教程中，**我们将从概念上理解什么是 servlet 和 servlet 容器，以及它们如何工作**。

我们还会在请求、响应、会话对象、共享变量和多线程的上下文中看到它们。

## 2。什么是 Servlets 及其容器

Servlets 是用于 web 开发的 JEE 框架的一个组件。它们基本上是运行在容器边界内的 Java 程序。总的来说，**它们负责接受请求，处理请求，并发回响应**。[Java servlets 简介](/web/20220627074056/https://www.baeldung.com/intro-to-servlets)提供了对这个主题的基本理解。

要使用它们，servlets 需要先在[注册](/web/20220627074056/https://www.baeldung.com/register-servlet)，这样无论是基于 JEE 还是基于 Spring 的容器都可以在启动时获取它们。一开始，容器通过调用它的`init()`方法来实例化一个 servlet。

一旦初始化完成，servlet 就可以接受传入的请求。随后，容器引导这些请求在 servlet 的`service()`方法中进行处理。之后，它根据 HTTP 请求类型进一步将请求委托给适当的方法，比如`doGet()`或`doPost()`。

使用`destroy()`，容器拆除 servlet，并且不再接受传入的请求。**我们称这个`init-service-destroy`周期为一个 servlet** 的生命周期。

现在让我们从容器的角度来看这个问题，比如 [Apache Tomcat](/web/20220627074056/https://www.baeldung.com/tomcat) 或 [Jetty](/web/20220627074056/https://www.baeldung.com/deploy-to-jetty) 。在启动时，它创建一个 [`ServletContext`](/web/20220627074056/https://www.baeldung.com/context-servlet-initialization-param) 的对象。`ServletContext`的工作是充当服务器或容器的内存，并记住所有与 web 应用程序相关联的 servlets、过滤器和监听器，如其`web.xml`或等效注释中所述。在我们停止或终止容器之前，`ServletContext`会一直使用它。

然而， **servlet 的`load-on-startup`参数在这里发挥了重要作用**。如果该参数的值大于零，则服务器仅在启动时对其进行初始化。如果没有指定这个参数，那么当一个请求第一次命中它时，就会调用 servlet 的`init()`。

## 3.请求、响应和会话

在上一节中，我们讨论了发送请求和接收响应，这基本上是任何客户机-服务器应用程序的基础。现在，让我们从 servlets 的角度来详细看看它们。

在这种情况下，**请求会用`[HttpServletRequest](https://web.archive.org/web/20220627074056/https://javaee.github.io/javaee-spec/javadocs/javax/servlet/http/HttpServletRequest.html)`表示，响应用 [`HttpServletResponse`](https://web.archive.org/web/20220627074056/https://javaee.github.io/javaee-spec/javadocs/javax/servlet/http/HttpServletResponse.html)** 表示。

每当客户端(如浏览器或 curl 命令)发送请求时，容器就会创建一个新的`HttpServletRequest`和`HttpServletResponse`对象。然后，它将这些新对象传递给 servlet 的`service`方法。基于`HttpServletRequest`的方法属性，该方法确定应该调用哪个`doXXX`方法。

除了关于方法的信息，请求对象还携带其他信息，如头、参数和主体。类似地，`HttpServletResponse`对象也携带头、参数和主体——我们可以在 servlet 的`doXXX`方法中设置它们。

**这些物体都是短命的**。当客户端获得响应时，服务器将请求和响应对象标记为垃圾收集。

那么，我们如何在后续的客户端请求或连接之间维护状态呢？ [`HttpSession`](https://web.archive.org/web/20220627074056/https://javaee.github.io/javaee-spec/javadocs/javax/servlet/http/HttpSession.html) 就是这个谜题的答案。

这基本上是将对象绑定到一个用户会话，因此与特定用户相关的信息可以跨多个请求持久化。这通常使用 cookies 的概念来实现，使用 [`JSESSIONID`](/web/20220627074056/https://www.baeldung.com/java-servlet-cookies-session#httpsession-object) 作为给定会话的唯一标识符。我们可以在`web.xml`中指定会话的超时时间:

```java
<session-config>
    <session-timeout>10</session-timeout>
</session-config> 
```

这意味着如果我们的会话空闲了 10 分钟，服务器将会丢弃它。任何后续请求都会创建一个新的会话。

## 4。Servlets 如何共享数据

基于所需的范围，servlets 可以通过多种方式共享数据。

正如我们在前面几节中看到的，不同的对象有不同的生存期。`HttpServletRequest`和`HttpServletResponse`对象只存在于一个 servlet 调用之间。`HttpSession`只要它处于活动状态并且没有超时，它就一直存在。

**`ServletContext`的寿命最长。**它与网络应用程序一起诞生，只有当应用程序本身关闭时才会被破坏。由于 servlet、过滤器和监听器实例与上下文相关联，因此只要 web 应用程序启动并运行，它们就存在。

因此，如果我们的需求是在所有 servlets 之间共享数据，比方说，如果我们想要统计我们站点的访问者数量，那么我们应该将变量放在`ServletContext`中。如果我们需要在一个会话中共享数据，那么我们将把它保存在会话范围内。在这种情况下，用户名就是一个例子。

最后，还有与单个请求的数据相关的请求范围，比如请求负载。

## 5.处理多线程

多个`HttpServletRequest`对象在彼此之间共享 servlet，这样每个请求都用它自己的 servlet 实例线程来操作。

就线程安全而言，这实际上意味着**我们不应该将请求或会话范围的数据作为 servlet** 的实例变量。

例如，让我们考虑这个片段:

```java
public class ExampleThree extends HttpServlet {

    private String instanceMessage;

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) 
      throws ServletException, IOException {
        String message = request.getParameter("message");
        instanceMessage = request.getParameter("message");
        request.setAttribute("text", message);
        request.setAttribute("unsafeText", instanceMessage);
        request.getRequestDispatcher("/jsp/ExampleThree.jsp").forward(request, response);
    }
}
```

在这种情况下，会话中的所有请求共享`instanceMessage`，而`message`对于给定的请求对象是唯一的。因此，在并发请求的情况下，`instanceMessage`中的数据可能会不一致。

## 6.结论

在本教程中，**我们看了一些关于 servlets 的概念，它们的容器，以及它们围绕着**的一些基本对象。我们还看到了 servlets 如何共享数据，以及多线程如何影响它们。

和往常一样，源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220627074056/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-xml)