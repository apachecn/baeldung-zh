# 在 Java 中寻找自由端口

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-free-port>

## 1.概观

当在我们的 Java 应用程序中启动一个套接字服务器时，`java.net` API 要求我们指定一个空闲的端口号来监听。端口号是必需的，以便 TCP 层可以识别传入数据的目标应用程序。

显式指定端口号并不总是一个好的选择，因为应用程序可能已经占用了它。这将导致我们的 Java 应用程序出现输入/输出异常。

在这个快速教程中，我们将了解如何检查特定的端口状态，以及如何使用自动分配的端口。我们将研究如何用普通 Java 和 Spring framework 实现这一点。我们还将看看其他一些服务器实现，比如嵌入式 Tomcat 和 Jetty。

## 2.检查端口状态

让我们看看如何使用`java.net` API 检查特定端口是空闲还是被占用。

### 2.1.特定端口

我们将利用`java.net` API 中的 [`ServerSocket`](/web/20221205210444/https://www.baeldung.com/a-guide-to-java-sockets) 类创建一个服务器套接字，绑定到指定的端口。**在其构造函数** **中，`ServerSocket` 接受一个显式端口号。该类还实现了`Closeable` 接口，因此可以在 [`try-with-resources`](/web/20221205210444/https://www.baeldung.com/java-try-with-resources) 中使用它来自动关闭套接字并释放端口:**

```java
try (ServerSocket serverSocket = new ServerSocket(FREE_PORT_NUMBER)) {
    assertThat(serverSocket).isNotNull();
    assertThat(serverSocket.getLocalPort()).isEqualTo(FREE_PORT_NUMBER);
} catch (IOException e) {
    fail("Port is not available");
}
```

如果我们使用一个特定的端口两次，或者它已经被另一个应用程序占用，`ServerSocket`构造函数将抛出一个`IOException`:

```java
try (ServerSocket serverSocket = new ServerSocket(FREE_PORT_NUMBER)) {
    new ServerSocket(FREE_PORT_NUMBER);
    fail("Same port cannot be used twice");
} catch (IOException e) {
    assertThat(e).hasMessageContaining("Address already in use");
}
```

### 2.2.P **ort** 范围

现在让我们看看如何利用抛出的`IOException,` 来创建一个服务器套接字，使用给定端口号范围内的第一个空闲端口:

```java
for (int port : FREE_PORT_RANGE) {
    try (ServerSocket serverSocket = new ServerSocket(port)) {
        assertThat(serverSocket).isNotNull();
        assertThat(serverSocket.getLocalPort()).isEqualTo(port);
        return;
    } catch (IOException e) {
        assertThat(e).hasMessageContaining("Address already in use");
    }
}
fail("No free port in the range found");
```

## 3.找到一个空闲端口

使用显式端口号并不总是一个好的选择，所以让我们研究一下自动分配空闲端口的可能性。

### 3.1.普通爪哇咖啡

我们可以在`ServerSocket`类构造函数中使用一个特殊的端口号零。因此，`java.net` API 会自动为我们分配一个空闲端口:

```java
try (ServerSocket serverSocket = new ServerSocket(0)) {
    assertThat(serverSocket).isNotNull();
    assertThat(serverSocket.getLocalPort()).isGreaterThan(0);
} catch (IOException e) {
    fail("Port is not available");
}
```

### 3.2.弹簧框架

Spring framework 包含一个`SocketUtils`类，我们可以用它来找到一个可用的空闲端口。它的内部实现使用了`ServerSocket`类，如我们前面的例子所示:

```java
int port = SocketUtils.findAvailableTcpPort();
try (ServerSocket serverSocket = new ServerSocket(port)) {
    assertThat(serverSocket).isNotNull();
    assertThat(serverSocket.getLocalPort()).isEqualTo(port);
} catch (IOException e) {
    fail("Port is not available");
}
```

## 4.其他服务器实现

现在让我们看看其他一些流行的服务器实现。

### 4.1.码头

[Jetty 是一款非常流行的 Java 应用嵌入式服务器](/web/20221205210444/https://www.baeldung.com/jetty-embedded)。**它会自动为我们分配一个空闲端口，除非我们通过`ServerConnector`类的`setPort`方法明确地设置它:**

```java
Server jettyServer = new Server();
ServerConnector serverConnector = new ServerConnector(jettyServer);
jettyServer.addConnector(serverConnector);
try {
    jettyServer.start();
    assertThat(serverConnector.getLocalPort()).isGreaterThan(0);
} catch (Exception e) {
    fail("Failed to start Jetty server");
} finally {
    jettyServer.stop();
    jettyServer.destroy();
}
```

### 4.2.雄猫

[Tomcat](/web/20221205210444/https://www.baeldung.com/tomcat) ，另一个流行的 Java 嵌入式服务器，工作方式有点不同。我们可以通过`Tomcat`类的`setPort`方法指定一个显式端口号。**如果我们提供的端口号为零，Tomcat 将自动分配一个空闲端口。**但是，如果我们不设置任何端口号，Tomcat 将使用其默认端口 8080。请注意，默认的 Tomcat 端口可能被其他应用程序占用:

```java
Tomcat tomcatServer = new Tomcat();
tomcatServer.setPort(0);
try {
    tomcatServer.start();
    assertThat(tomcatServer.getConnector().getLocalPort()).isGreaterThan(0);
} catch (LifecycleException e) {
    fail("Failed to start Tomcat server");
} finally {
    tomcatServer.stop();
    tomcatServer.destroy();
}
```

## 5.结论

在本文中，我们探讨了如何检查特定的端口状态。我们还讲述了如何从一系列端口号中找到空闲端口，并解释了如何使用自动分配的空闲端口。

在示例中，我们介绍了来自`java.net` API 的基本`ServerSocket`类和其他流行的服务器实现，包括 Jetty 和 Tomcat。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221205210444/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-networking-3)