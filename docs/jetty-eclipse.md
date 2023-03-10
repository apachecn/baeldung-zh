# Eclipse 中的 Jetty 配置

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jetty-eclipse>

## 1.概观

Web 应用程序是 Java 最流行的用例之一。Web 服务器和 Servlet 容器为部署应用程序提供了运行时。

不幸的是，在 web 服务器上部署 Web 应用程序和排除故障有时很复杂。幸运的是，ide 对大多数应用程序都有很好的调试支持。然而，为了调试 web 应用程序，我们需要在 IDE 中嵌入一个 Web 服务器。

在本教程中，**我们将在 Eclipse 中嵌入** **Jetty，并在其上运行和调试应用程序**。

## 2.Eclipse Jetty 插件

将 Jetty 连接到 Eclipse 的最简单方法是使用 Eclipse Jetty 插件。

该插件将一个托管的 Jetty 服务器添加到 Eclipse 中。因此，它允许我们无缝地部署、测试或调试应用程序。此外，该插件提供了一个接口，可以轻松配置服务器。

安装插件最快的方法是通过市场。在 eclipse 中，Marketplace 使我们只需点击几下鼠标就能安装插件:

[![marketplace](img/e2a23e140c8e2a03fed21988f56fb750.png)](/web/20221129013521/https://www.baeldung.com/wp-content/uploads/2018/08/marketplace.jpg)

## 3.示例应用程序

现在让我们开发一个简单的 web 应用程序。

首先，让我们将`web.xml `添加到我们项目的`/src/main/webapp/WEB-INF `文件夹中:

```java
<web-app 
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee/webapp_4_0.xsd"
	version="4.0">

	<welcome-file-list>
		<welcome-file>helloworld.html</welcome-file>
	</welcome-file-list>

</web-app>
```

让我们添加一个简单的静态文件。在我们的例子中，`helloworld.html `看起来像这样:

```java
<!DOCTYPE html>
<html>
   <head>
      <meta charset="ISO-8859-1">
      <title>Hello World</title>
   </head>
   <body>Hello World!</body>
</html>
```

注意，我们还没有向我们的`web.xml`添加任何 servlet 映射。

相反，我们将为我们的 Servlet 使用 Servlet 3 注释:

```java
@WebServlet("/helloworld")
public class HelloWorldServlet extends HttpServlet
```

注释使我们的 servlet 类能够被扫描并部署在容器上。

我们应该记住，Jetty 不支持基本 HTTP 模块的注释。因此**我们需要添加注释支持模块来使其工作**。

我们将在下面几节中看到如何做到这一点。

## 4.在 Jetty 上运行应用程序

在服务器上部署 web 应用程序因供应商而异。Eclipse Jetty 插件为我们处理这个过程。同样，它与我们的 IDE 调试器集成，改善了开发体验。

有时我们需要用一些配置来运行应用程序。Eclipse 允许我们使用启动配置来做到这一点。

这是它在 Jetty 上运行应用程序的样子:

[![Launch config](img/41c2b1fc2651ec46a2ea4fdb4227d695.png)](/web/20221129013521/https://www.baeldung.com/wp-content/uploads/2018/08/Launch-config.jpg)

我们可以为我们的应用程序配置以下参数:

*   上下文路径——应用程序 URL 的前缀
*   HTTP 端口–部署应用程序的端口，默认为 8080
*   启用 HTTPS–用于与 HTTP 一起部署在 HTTPS 上
*   HTTPS 港-默认为 8443

与常规 Jetty 一样，Eclipse Jetty 插件允许我们在部署之前管理应用程序的依赖关系。对于 maven 应用程序，如果我们希望从服务器提供类路径，我们可以选择依赖范围作为类路径。

## 5.Jetty 服务器选项

Jetty 是一个高度可配置的 Servlet 容器。我们可以指定各种参数，如`Thread Pool Size`、`Shutdown Interval`等。

除此之外，Jetty 允许我们在基本的 HTTP 模块之上添加各种模块。以下是我们可以添加的一些常见模块:

*   注释支持——支持特定于 Servlet 的注释
*   JNDI 支持–允许通过码头管理 JNDI 资源
*   Websocket 支持——支持 Websocket 服务器和客户端实现
*   JMX 支持——允许使用任何 MBeans 浏览器监控 Jetty
*   JSP 支持——支持在 Jetty 中编译和部署 JSP

这些配置在 Eclipse Jetty 中也是可能的。因此，我们可以从启动配置中配置服务器参数和模块。

最后，Eclipse Jetty 4.0 插件附带了一个嵌入式 Jetty 9.3 服务器。但是，我们可以从启动配置中为我们的应用程序配置一个外部 jetty 服务器。

## 6.Eclipse Jetty 控制台

Eclipse Jetty 提供了一个控制台，其中包含一些有用的控制命令。这个控制台可以方便地管理服务器或从服务器收集一些指标。

**控制台需要在启动配置**中启用。启用后，我们可以从 Eclipse 控制台执行控制命令。

以下是我们可以使用的一些常用命令的列表:

*   memory–当前应用程序的内存信息
*   线程–正在运行的应用程序的线程转储
*   重启–重启正在运行的应用程序
*   停止–正常停止服务器和其上运行的所有应用程序

## 7.结论

Eclipse Jetty 插件是通过嵌入 Jetty 服务器来快速运行或调试应用程序的好方法。它还允许我们配置我们的应用程序和底层 Jetty 服务器。

在本教程中，我们安装了 Eclipse Jetty 插件并部署了我们的应用程序。我们还创建了一个启动配置，并提供了应用程序和服务器参数。