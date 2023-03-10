# JavaServer 页面指南(JSP)

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jsp>

**目录**

*   [**1。**概述](#overview)
*   [**2。** JavaServer 页面](#jsp)
*   [**2.1。** JSP 语法](#syntax)
*   [**2.2。**静态和动态内容](#contents)
*   [**2.3。**隐含对象](#implicit)
*   [**2.4。**其他隐含对象](#implicit)
*   [**2.5。**指令](#directives)
*   [**2.6。**页面指令](#page)
*   [**3.0。**三个例子](#threeExamples)
*   [**3.1。**在 Servlet 中呈现的 HTML】](#htmlRendered)
*   [**3.2。**JSP 中的 Java 静态内容](#staticJava)
*   [**3.3。**带转发的 JSP](#forwarding)
*   [**3.4。**试试看！](#tryIt)
*   [**4。**结论](#conclusion)

## 1。概述

**JavaServer Pages (JSP)允许使用 Java 和 Java servlet**将`dynamic`内容注入到`static`内容中。我们可以向一个 **Java Servlet 发出请求，执行相关的逻辑，并呈现一个特定的视图服务器端供客户端使用**。本文将提供使用 Java 8 和 Jave 7 EE 的 JavaServer 页面的全面概述。

我们将从探索一些与 JSP 相关的关键概念开始:也就是说，`dynamic`和`static`内容之间的区别，JSP 生命周期，JSP 语法以及指令和编译时创建的隐式对象！

## 2。JavaServer 页面

JavaServer Pages (JSP)支持将特定于 Java 的数据传递到或放置在. JSP 视图中，并在客户端使用。

**JSP 文件本质上是。带有一些额外语法的 html 文件**,以及一些微小的初始差异:

1.  后缀`.html`被替换为。jsp(它被认为是. jsp 文件类型)和
2.  下面的标记被添加到。html 标记元素:

```java
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
```

让我们回顾一下 JSP 中的一些关键概念。

### 2.1。JSP 语法

有两种方法可以将 Java 代码添加到. jsp 中。首先，我们可以使用基本的 Java Scriptlet 语法，这涉及到将 Java 代码块放在两个 Scriptlet 标记中:

```java
<% Java code here %>
```

第二种方法是特定于 XML 的:

```java
<jsp:scriptlet>
    Java code here
</jsp:scriptlet>
```

重要的是，通过使用`if`、`then`和`else`子句，然后用这些括号包装相关的标记块，可以在 JSP 中使用条件逻辑客户端。

```java
<% if (doodad) {%>
    <div>Doodad!</div>
<% } else { %>
    <p>Hello!</p>
<% } %>
```

例如，如果`doodad`为真，我们将显示第一个`div`元素，否则我们将显示第二个`p`元素！

### 2.2。`Static`和`Dynamic`内容

web 内容是独立于 RESTful、SOAP、HTTP、HTTPS 请求或其他用户提交信息的固定资产。

然而，内容是固定的，不会被用户输入修改。`Dynamic` web 内容是那些根据用户操作或信息做出响应、修改或更改的资产！

JSP 技术允许清晰地分离`dynamic`和`static`内容之间的责任。

服务器(servlet)管理`dynamic`内容，客户机(实际的。jsp 页面)是动态内容注入其中的`static`上下文。

让我们看看 JSP 创建的`implicit objects`，它允许您访问 JSP 相关的服务器端数据！

### 2.3。隐含对象

`Implicit objects`由 JSP 引擎在`compilation`期间自动生成。

`Implicit objects`包含`HttpRequest`和`HttpResponse`对象，并公开各种服务器端功能，以便在您的 servlet 中使用以及与您的进行交互。jsp！下面是创建的`implicit objects`列表:

**请求**
`request`属于`javax.servlet.http.HttpServletRequest`类。`request`对象公开所有用户输入数据，并使其在服务器端可用。

**响应**
`response`属于`javax.servlet.http.HttpServletResponse`类，决定在`request`产生后什么被传回客户端。

让我们仔细看看`request`和`response`隐式对象，因为它们是最重要和最常用的。

下面的例子演示了一个非常简单、不完整的 servlet 方法来处理 GET 请求。我省略了大部分细节，这样我们可以专注于如何使用`request`和`response`对象:

```java
protected void doGet(HttpServletRequest request, 
  HttpServletResponse response) throws ServletException, IOException {
    String message = request.getParameter("message");
    response.setContentType("text/html");
    . . .
}
```

首先，我们看到`request`和`response`对象作为参数传入方法，使它们在其作用域内可用。

我们可以使用`.getParameter()`函数访问请求参数。上面，我们截取了`message`参数并初始化了一个字符串变量，这样我们就可以在我们的服务器端逻辑中使用它。我们还可以访问`response`对象，它决定了传递给视图的数据是什么以及如何传递。

上面我们设置了它的内容类型。我们不需要返回`response`对象来让它的有效载荷在渲染时显示在 JSP 页面上！

**out**
属于`javax.servlet.jsp.JspWriter`类，用于向客户端写入内容。

至少有两种方法可以打印到 JSP 页面，这两种方法都值得在这里讨论。`out`是自动创建的，允许您写入内存，然后写入`response`对象:

```java
out.print(“hello”);
out.println(“world”);
```

就是这样！

第二种方法性能更好，因为它允许您直接写入`response`对象！这里，我们使用`PrintWriter`:

```java
PrintWriter out = response.getWriter();
out.println("Hello World");
```

### 2.4。其他隐含对象

这里有一些其他的`Implicit objects`也很好知道！

**会话**
`session`属于类`javax.servlet.http.HttpSession`在会话期间维护用户数据。

**应用**
`application`属于`javax.servlet.ServletContext`类，存储初始化时设置的应用范围参数或需要在应用范围内访问的参数。

**异常**
`exception`属于`javax.servlet.jsp.JspException`类，用于在带有`<%@ page isErrorPage=”true” %>`标签的 JSP 页面上显示错误消息。

**页面**
`page`属于类`java.lang.Object`允许一个人访问或引用当前的 servlet 信息。

**pageContext**
`pageContext`属于`javax.servlet.jsp.PageContext`类，默认为`page`范围，但可以用来访问`request`、`application`、`session attributes`。

**config**
属于类`javax.servlet.ServletConfig`是 servlet 配置对象，允许获取 servlet 上下文、名称和配置参数。

既然我们已经讨论了 JSP 提供的`implicit objects` ，让我们转向允许。jsp 页面来(间接)访问其中的一些对象。

### 2.5。指令

JSP 提供了现成的指令，可用于指定 JSP 文件的核心功能。JSP 指令有两个部分:(1)指令本身和(2)被赋值的指令的属性。

可以使用指令标签引用的三种指令是`<%@ page … %>`，它定义 JSP 的依赖项和属性，包括 `content type`和`language`、`<%@ include … %>`，它指定要使用的导入或文件，以及`<%@ taglib …%>`，它指定一个标签库，定义页面要使用的定制动作。

因此，作为一个例子，一个页面指令将使用 JSP 标签以如下方式指定:`<%@ page attribute=”value” %>`

而且，我们可以使用 XML 来实现，如下所示:`<jsp:directive.page attribute=”value” />`

### 2.6。页面指令属性

有许多属性可以在页面指令中声明:

**自动刷新** `<%@ page autoFlush=”false” %>`

`autoFlush`控制缓冲输出，当达到缓冲大小时清除。默认值为`true`。

**缓冲** `<%@ page buffer=”19kb” %>`

设置 JSP 页面使用的缓冲区的大小。默认值为 8kb。

**错误页面** `<%@ page errorPage=”errHandler.jsp” %>`

`errorPage`将 JSP 页面指定为错误页面。

**延伸** `<%@ page extends=”org.apache.jasper.runtime.HttpJspBase” %>`

`extends`指定对应 servlet 代码的超类。

**信息** `<%@ page info=”This is my JSP!” %>` 

`info`用于为 JSP 设置基于文本的描述。

**is aligned**

`isELIgnored`声明页面是否会忽略 JSP 中的`Expression Language` (EL)。EL 使表示层能够与 Java 托管 beans 进行通信，并使用`${…}`语法进行通信，虽然我们不会在这里深入 EL 的本质，但下面的几个示例足以构建我们的示例 JSP 应用程序！`isELIgnored`的默认值为`false`。

**isErrorPage**

`isErrorPage`表示页面是否为错误页面。如果我们在应用程序中为我们的页面创建一个错误处理程序，我们必须指定一个错误页面。

**是安全的**吗`<%@ page isThreadSafe=”false” %>`

`isThreadSafe`的默认值为`true`。`isThreadSafe`决定 JSP 是否可以使用 Servlet 多线程。一般来说，你不会希望
关闭这个功能。

**语言** `<%@ page language=”java” %>`

`language`决定在 JSP 中使用什么脚本语言。默认值为`Java`。

**会期** `<%@ page session=”value”%>`

`session`确定是否维持 HTTP 会话。它默认为真，并接受值`true`或`false`。

**trim directive white spaces**

在编译时，去除 JSP 页面中的空白，将代码压缩成更紧凑的块。将这个值设置为`true`可能有助于减少 JSP 代码的大小。默认值为`false`。

## 3。三个例子

现在我们已经回顾了 JSP 的核心概念，让我们将这些概念应用到一些基本的例子中，这些例子将帮助您启动并运行您的第一个 JSP 服务 servlet！

将 Java 注入. jsp 有三种主要方式，下面我们将使用 Java 8 和 Jakarta EE 中的原生功能来探索每一种方式。

首先，我们将把标记服务器端呈现给客户端显示。其次，我们将看看如何将 Java 代码直接添加到我们的。独立于`javax.servlet.http`的`request`和`response`对象的 jsp 文件。

第三，我们将演示如何将一个`HttpServletRequest`转发到一个特定的。jsp 并将服务器端处理过的 Java 绑定到它。

让我们使用托管在 Tomcat 中的`File/New/Project/Web/Dynamic web project/`类型在 Eclipse 中设置我们的项目！创建项目后，您应该会看到:

```java
|-project
  |- WebContent
    |- META-INF
      |- MANIFEST.MF
    |- WEB-INF
      |- lib
      |- src
```

我们将向应用程序结构中添加一些文件，这样我们最终会得到:

```java
|-project
  |- WebContent
    |- META-INF
      |- MANIFEST.MF
    |- WEB-INF
      |-lib
      *-web.xml
        |- ExampleTree.jsp
        |- ExampleTwo.jsp
        *- index.jsp
      |- src
        |- com
          |- baeldung
            *- ExampleOne.java
            *- ExampleThree.java
```

让我们设置`index.jsp`,当我们在 Tomcat 8 中访问 URL 上下文时将显示它:

```java
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
    <head>
        <title>JSP Examples</title>
    </head>
    <body>
        <h1>Simple JSP Examples</h1>
        <p>Invoke HTML rendered by Servlet: <a href="ExampleOne" target="_blank">here</a></p>
        <p>Java in static page: <a href="ExampleTwo.jsp" target="_blank">here</a></p>
        <p>Java injected by Servlet: <a href="ExampleThree?message=hello!" target="_blank">here</a></p>
    </body>
</html>
```

有三个`a`，每个都链接到我们将在下面的 4.1 到 4.4 节中讨论的一个例子。

我们还需要确保我们已经设置好了`web.xml`:

```java
<welcome-file-list>
    <welcome-file>index.html</welcome-file>
    <welcome-file>index.htm</welcome-file>
    <welcome-file>index.jsp</welcome-file>
</welcome-file-list>
<servlet>
    <servlet-name>ExampleOne</servlet-name>
    <servlet-class>com.baeldung.ExampleOne</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>ExampleOne</servlet-name>
    <url-pattern>/ExampleOne</url-pattern>
</servlet-mapping>
```

这里需要注意的是——如何正确地将我们的每个 servlet 映射到一个特定的 servlet 映射。这样做可以将每个 servlet 与一个可以使用它的特定端点关联起来！现在，我们将浏览下面的每个其他文件！

### 3.1。Servlet 中呈现的 HTML】

在这个例子中，我们实际上将跳过构建. jsp 文件！

相反，我们将创建我们的标记的字符串表示，然后在 ExampleOne Servlet 收到 GET 请求后，用`PrintWriter`将它写入 GET 响应:

```java
public class ExampleOne extends HttpServlet {
  @Override
  protected void doGet(HttpServletRequest request, 
    HttpServletResponse response) throws ServletException, IOException {
      response.setContentType("text/html");
      PrintWriter out = response.getWriter();
      out.println(
	"<!DOCTYPE html><html>" +
	"<head>" +
	"<meta charset=\"UTF-8\" />" +
	"<title>HTML Rendered by Servlet</title>" +
	"</head>" +
	"<body>" +
	"<h1>HTML Rendered by Servlet</h1></br>" +
	"<p>This page was rendered by the ExampleOne Servlet!</p>" +
	"</body>" +
	"</html>"
     );
   }
}
```

我们在这里做的是通过 servlet 请求处理直接注入我们的标记。代替 JSP 标记，我们生成 HTML，以及任何和所有要插入的特定于 Java 的数据，完全是服务器端的，没有静态 JSP！

前面，我们回顾了`out`对象，它是`JspWriter`的一个特性。

在上面，我使用了直接写入`response`对象的`PrintWriter`对象。

`JspWriter`实际缓冲要写入内存的字符串，然后在内存缓冲区被刷新后写入`response`对象。

`PrintWriter`已经附加到`response`对象。出于这些原因，我更喜欢在上面和下面的例子中直接写`response`对象。

### 3.2。JSP 中的 Java 静态内容

这里我们创建一个名为`ExampleTwo.jsp`的 JSP 文件，带有一个 JSP 标签。如上所述，这允许 Java 直接添加到我们的标记中。这里，我们随机打印一个`String[]`的元素:

```java
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<!DOCTYPE html>
    <head>
        <title>Java in Static Page Example</title>
    </head>
    <body>
        <h1>Java in Static Page Example</h1>
	    <% 
              String[] arr = {"What's up?", "Hello", "It's a nice day today!"}; 
	      String greetings = arr[(int)(Math.random() * arr.length)];	
            %>
        <p><%= greetings %></p>
    </body>
</html>
```

上面，你会看到 JSP 标签对象中的变量声明:`type` `variableName`和一个`initialization`，就像普通的 Java 一样。

我在上面的例子中演示了如何在不求助于特定 servlet 的情况下向静态页面添加 Java。在这里，Java 只是简单地添加到页面中，JSP 生命周期负责剩下的工作。

### 3.3。带转发的 JSP

现在，我们的最后和最复杂的例子！这里，我们将在 ExampleThree 上使用`@WebServlet`注释，这消除了对`server.xml`中 servlet 映射的需要。

```java
@WebServlet(
  name = "ExampleThree",
  description = "JSP Servlet With Annotations",
  urlPatterns = {"/ExampleThree"}
)
public class ExampleThree extends HttpServlet {
  @Override
  protected void doGet(HttpServletRequest request, HttpServletResponse response) 
    throws ServletException, IOException {
      String message = request.getParameter("message");
      request.setAttribute("text", message);
      request.getRequestDispatcher("/ExampleThree.jsp").forward(request, response);
   }
}
```

示例三将一个 URL 参数作为`message`传入，将该参数绑定到`request`对象，然后将该`request`对象重定向到`ExampleThree.jsp`。

因此，我们不仅实现了真正的`dynamic web`体验，而且还在包含多个。jsp 文件。

`getRequestDispatcher().forward()`是确保正确的简单方法。呈现 jsp 页面。

所有绑定到`request`对象的数据都发送了它的(。jsp 文件的)方式，然后将显示！下面是我们如何处理最后一部分:

```java
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<!DOCTYPE html>
    <head>
        <title>Java Binding Example</title>
    </head>
    <body>
        <h1>Bound Value</h1>
	    <p>You said: ${text}</p>
    </body>
</html>
```

注意添加到`ExampleThree.jsp`顶部的 JSP 标记。您会注意到我在这里交换了 JSP 标签。我在用表达式语言(之前提到过)渲染我们的 set 参数(绑定为`${text}`)！

### 3.4。试试吧！

现在，我们将把我们的应用程序导出到一个. war 文件中，该文件将在 Tomcat 8 中启动和托管！找到您的`server.xml`，我们会将我们的`Context`更新为:

```java
<Context path="/spring-mvc-xml" docBase="${catalina.home}/webapps/spring-mvc-xml">
</Context>
```

这将允许我们在`localhost:8080/spring-mvc-xml/jsp/index.jsp`上访问我们的 servlets 和 JSP！到 [GitHub](https://web.archive.org/web/20220710163703/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-xml) 拿一份工作副本。恭喜你。

### 4。结论

我们已经走过了相当多的地方！我们已经了解了什么是 JavaServer Pages，引入它们的目的是什么，它们的生命周期，如何创建它们，以及最后实现它们的几种不同方法！

JSP 介绍到此结束！保重，继续努力！