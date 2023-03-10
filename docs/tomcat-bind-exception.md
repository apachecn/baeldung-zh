# java 中的 Tomcat java.net.BindException:地址已在使用中错误

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/tomcat-bind-exception>

## 1.概观

在这个快速教程中，我们将看看是什么导致了常见的`java.net.BindingException Error: Address already in Use `错误，以及我们如何处理它。

## 2.错误发生在什么时候？

我们知道，Apache Tomcat 服务器默认使用 8080 端口。

端口号范围从 0 到 65535，但是，**一个端口在任何时候都只能被一个应用程序占用**。

这个异常表明应用程序正在试图使用一个已经被其他进程占用的端口，或者我们没有正确地停止 Tomcat 服务器。

## 3.诊断

要解决这个错误，我们可以终止使用该端口的服务，或者将我们的 web 服务器改为在另一个端口上运行。

### 3.1。发现冲突

在这种情况下，我们需要找出哪个应用程序正在使用该端口。

`netstat`命令可用于发现当前的 TCP/IP 连接。

下面是在不同环境中可以用来查找和终止进程的命令。

**在 Windows 上，**输出的最后一列将给出当前在 8080 上运行的服务的进程 id:

```java
netstat -ano | find "8080"
```

输出:

```java
TCP 0.0.0.0:8080 0.0.0.0:0 LISTENING 21376 
```

这里，21376 是监听端口 8080 的进程的进程 id。

**在 Unix/Linux 环境下**:

```java
netstat -pant | grep "8080"
```

输出:

```java
TCP 0.0.0.0:8080 0.0.0.0:0 LISTENING 21376 
```

与 Windows 输出相同。这里，21376 是监听端口 8080 的进程的进程 id。

**在 Mac OS X 上:**

```java
lsof -t -i :8080
```

输出:

```java
21376
```

它将只显示 PID。

### 3.2。在另一个端口上运行服务器

如果我们知道什么进程正在运行，它为什么运行，以及它需要在那个端口上运行，我们就可以改变我们的服务器应用程序试图运行的端口。

要更改 Tomcat 端口，我们需要编辑`server.xml`文件。为此:

*   打开`tomcat/conf `文件夹
*   编辑`server.xml`
*   用`new port`替换`connector port `
*   重新启动 tomcat 服务器

`server.xml`文件如下所示:

```java
<Connector port="8080" protocol="HTTP/1.1" 
  connectionTimeout="20000" redirectPort="8443" />
```

现在 Tomcat 将在定制的端口上运行。

### 3.3。终止正在运行的服务

**要停止运行过程，我们可以使用`kill`** 命令。

使用我们在 3.1 中找到的进程 ID。，根据我们运行的操作系统，我们将需要不同的命令。

**在 Windows 环境下:**

```java
taskkill /F /PID 21376
```

**在 Unix/Linux 环境下:**

```java
kill - 21376
```

**Mac OS X 环境:**

```java
kill -9 21376
```

## 4.结论

正如本文开头提到的，`java.net.BindingException` 是一个普遍但容易解决的错误。

主要的困难是使用端口和`netstat`终端应用程序找到冲突的服务，然后决定适当的操作过程。

一旦被发现，修复是很容易的。