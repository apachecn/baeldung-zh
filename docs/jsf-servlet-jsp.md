# JSF、Servlet 和 JSP 之间的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jsf-servlet-jsp>

## 1.介绍

在开发任何应用程序时，选择正确的技术起着重要的作用。然而，决定并不总是直截了当的。

在本文中，我们将对三种流行的 Java 技术进行比较。在开始比较之前，我们将从探索每种技术的用途及其生命周期开始。那么，我们就来看看他们的突出特点是什么，在几个特点的基础上进行比较。

## 2.JSF

Jakarta Server Faces，原名 [JavaServer Faces](/web/20220628131255/https://www.baeldung.com/spring-jsf) ，是一个 web 框架，为 Java 应用程序构建基于组件的用户界面。像许多其他的一样，它也遵循了[的 MVC 方法](/web/20220628131255/https://www.baeldung.com/mvc-servlet-jsp)。MVC 的“视图”在可重用 UI 组件的帮助下简化了用户界面的创建。

JSF 有各种各样的标准 UI 组件，也提供了通过外部 API 定义新组件的灵活性。

任何应用程序的生命周期都是指从开始到结束的各个阶段。类似地，JSF 应用程序的生命周期在客户端发出 HTTP 请求时开始，在服务器发出响应时结束。JSF 生命周期是一个请求-响应生命周期，处理两种类型的请求:初始请求和回发。

**JSF 应用** **的生命周期由两个主要阶段组成:`execute`和`render`** 。

`execute`阶段又分为六个阶段:

*   还原视图:当 JSF 收到请求时启动
*   应用请求值:回发请求期间组件树的还原
*   过程验证:处理组件树上注册的所有验证器
*   更新模型值:遍历组件树并设置相应的服务器端对象属性
*   调用应用程序:处理应用程序级事件，比如提交表单
*   呈现响应:构建视图并呈现页面

在`render`阶段，系统将所请求的资源呈现为对客户端浏览器的响应。

JSF 2.0 是一个重要的版本，包括了 Facelets、复合组件、AJAX 和资源库。

在 Facelets 之前，JSP 是 JSF 应用程序的默认模板引擎。在 JSF 2.x 的旧版本中，引入了许多新特性，使得框架更加健壮和高效。这些特性包括对注释、HTML5、Restful 和无状态 JSF 等的支持。

## 3.小型应用程序

Jakarta Servlets，以前被称为 [Java Servlets](/web/20220628131255/https://www.baeldung.com/intro-to-servlets) ，扩展了服务器的功能。**通常，servlets 使用由容器实现的请求/响应机制与 web 客户端交互。**

servlet 容器是 web 服务器的重要组成部分。它管理 servlets 并根据用户请求创建动态内容。每当 web 服务器收到请求时，它会将请求发送到一个[注册的 servlet](/web/20220628131255/https://www.baeldung.com/register-servlet) 。

生命周期只包含三个阶段。首先，调用`init()` 方法来启动 servlet `.`，然后，容器将传入的请求发送给执行所有任务的`service()`方法。最后，`destroy()`方法清理了一些东西并拆除了 servlet。

Servlets 有许多重要的特性，包括对 Java 及其库的本地支持、web 服务器的标准 API 以及 HTTP/2 的强大功能。此外，它们允许异步请求，并为每个请求创建单独的线程。

## 4.JSP

Jakarta Server Pages 以前被称为 [JavaServer Pages](/web/20220628131255/https://www.baeldung.com/jsp) ，它使我们能够将动态内容注入到静态页面中。JSP 是 servlet 的高级抽象，因为它们在执行开始之前被转换成 servlet。

诸如变量声明和打印值、循环、条件格式和异常处理等常见任务通过[JSTL 库](/web/20220628131255/https://www.baeldung.com/jstl)来执行。

JSP 的生命周期类似于 servlet，只是多了一个步骤——编译步骤。当浏览器请求页面时，JSP 引擎首先检查是否需要编译页面。编译步骤包括三个阶段。

最初，引擎解析页面。然后，它将页面转换成 servlet。最后，生成的 servlet 编译成一个 Java 类。

JSP 有许多显著的特性，比如跟踪会话、良好的表单控制以及向/从服务器发送/接收数据。因为 JSP 是建立在 servlet 之上的，所以它可以访问所有重要的 Java APIs，比如 JDBC、JNDI 和 EJB。

## 5.主要差异

Servlet 技术是 J2EE web 应用程序开发的基础。然而，它没有视图技术，开发人员必须将标记标签与 Java 代码混合在一起。此外，它缺少用于构建标记、验证请求和启用安全特性等常见任务的实用程序。

JSP 填补了 servlet 的标记空白。在 JSTL 和 EL 的帮助下，我们可以定义任何定制的 HTML 标签来构建一个好的 UI。不幸的是，JSP 编译缓慢，调试困难，将基本的表单验证和类型转换留给了开发人员，并且缺乏对安全性的支持。

JSF 是一个合适的框架，它将数据源与可重用的 UI 组件连接起来，提供对多个库的支持，并减少构建和管理应用程序的工作量。由于是基于组件的，JSF 总是比 JSP 有更好的安全优势。尽管有这么多好处，JSF 还是很复杂，而且有一个陡峭的学习曲线。

根据 MVC 设计模式，servlet 充当控制器，JSP 充当视图，而 JSF 是一个完整的 MVC。

正如我们已经知道的，servlet 将需要 Java 代码中的手动 HTML 标记。出于同样的目的，JSP 使用 HTML，JSF 使用 Facelets。此外，两者都提供了对定制标签的支持。

servlet 和 JSP 中没有默认的错误处理支持。相比之下，JSF 提供了一堆预定义的验证器。

在通过 web 传输数据的应用程序中，安全性一直是一个问题。仅支持基于角色和基于表单的身份验证的 JSP 在这方面有所欠缺。

说到协议，JSP 只接受 HTTP，而 servlet 和 JSF 支持几种协议，包括 HTTP/HTTPS、SMTP 和 SIP。所有这些技术都提倡多线程，并且需要一个 web 容器来运行。

## 6.结论

在本教程中，我们比较了 Java 世界中三种流行的技术:JSF、Servlet 和 JSP。首先，我们看到了每种技术代表什么，以及它的生命周期是如何发展的。然后，我们讨论了每种技术的主要特性和局限性。最后，我们基于几个特征对它们进行了比较。

应该选择哪种技术，完全取决于具体情况。应用程序的性质应该是决定性因素。