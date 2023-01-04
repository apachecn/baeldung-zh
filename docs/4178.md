# 用 Java 创建和配置 Jetty 9 服务器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jetty-java-programmatic>

## 1。概述

在本文中，我们将讨论如何以编程方式创建和配置 Jetty 实例。

Jetty 是一个 HTTP 服务器和 servlet 容器，被设计成轻量级和易于嵌入的。我们将看看如何设置和配置服务器的一个或多个实例。

## 2。Maven 依赖关系

首先，我们希望[将带有以下 Maven 依赖项](https://web.archive.org/web/20221206062905/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.eclipse.jetty%22%20AND%20a%3A%22jetty-server%22)的 Jetty 9 添加到我们的`pom.xml`中:

```
<dependency>
    <groupId>org.eclipse.jetty</groupId>
    <artifactId>jetty-server</artifactId>
    <version>9.4.8.v20171121</version>
</dependency>
<dependency>
    <groupId>org.eclipse.jetty</groupId>
    <artifactId>jetty-webapp</artifactId>
    <version>9.4.8.v20171121</version>
</dependency>
```

## 3。创建基本服务器

使用 Jetty 构建嵌入式服务器就像编写代码一样简单:

```
Server server = new Server();
server.start();
```

关闭它同样简单:

```
server.stop();
```

## 4。处理者

既然我们的服务器已经启动并运行，我们需要指示它如何处理传入的请求。这可以使用`Handler` 界面来完成。

我们可以自己创建一个，但是 Jetty 已经为最常见的用例提供了一组实现。让我们来看看其中的两个。

### 4.1。`WebAppContext`

`WebAppContext`类允许您将请求处理委托给现有的 web 应用程序。该应用程序可以作为 WAR 文件路径或 webapp 文件夹路径提供。

如果我们想在“myApp”上下文中公开一个应用程序，我们应该写:

```
Handler webAppHandler = new WebAppContext(webAppPath, "/myApp");
server.setHandler(webAppHandler);
```

### 4.2。`HandlerCollection`

对于复杂的应用程序，我们甚至可以使用`HandlerCollection`类指定多个处理程序。

假设我们已经实现了两个自定义处理程序。第一个只执行日志记录操作，而第二个创建并向用户发回实际的响应。我们希望按照这个顺序处理每个传入的请求。

以下是如何做到这一点:

```
Handler handlers = new HandlerCollection();
handlers.addHandler(loggingRequestHandler);
handlers.addHandler(customRequestHandler);
server.setHandler(handlers);
```

## 5。连接器

接下来我们要做的是配置服务器将监听哪些地址和端口，并添加一个空闲超时。

`Server`类声明了两个方便的构造函数，可以用来绑定到特定的端口或地址。

虽然在处理小型应用程序时这可能还可以，但如果我们想在不同的套接字上打开多个连接，这就不够了。

在这种情况下，Jetty 提供了`Connector` 接口，更具体地说是`ServerConnector`类，它允许定义各种连接配置参数:

```
ServerConnector connector = new ServerConnector(server);
connector.setPort(80);
connector.setHost("169.20.45.12");
connector.setIdleTimeout(30000);
server.addConnector(connector);
```

使用这种配置，服务器将监听 169.20.45.12:80。在此地址上建立的每个连接都将有 30 秒的超时。

如果我们需要配置其他插座，我们可以添加其他连接器。

## 6。结论

在这个快速教程中，我们重点介绍了如何使用 Jetty 设置嵌入式服务器。我们还看到了如何使用`Handlers`和`Connectors`执行进一步的配置。

和往常一样，这里使用的所有代码都可以在 GitHub 上找到[。](https://web.archive.org/web/20221206062905/https://github.com/eugenp/tutorials/tree/master/libraries-server)