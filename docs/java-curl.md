# 在 Java 中使用 Curl

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-curl>

## 1。概述

在本教程中，我们将看看如何在 Java 程序中使用`[curl](https://web.archive.org/web/20220625221328/https://curl.haxx.se/)`工具。

**`Curl`是一个网络工具，用于使用 HTTP、FTP、TELNET 和 SCP 等协议在服务器和`curl`客户端**之间传输数据。

## 2。Curl 的基本用法

**我们可以通过使用`ProcessBuilder`从 Java 中执行`curl`命令，T1 是一个帮助类，用于构建 [`Process`](/web/20220625221328/https://www.baeldung.com/java-process-api) 类的实例。**

让我们看一个直接向操作系统发送命令的例子:

```java
String command =
  "curl -X GET https://postman-echo.com/get?foo1=bar1&foo2;=bar2";
ProcessBuilder processBuilder = new ProcessBuilder(command.split(" ")); 
```

首先，在将变量传递给`ProcessBuilder`构造函数之前，我们创建了`command`变量。

这里值得注意的是，如果`curl`可执行文件不在我们的系统路径上，我们必须在命令字符串中提供它的完整路径。

然后我们可以为`ProcessBuilder `设置工作目录，并开始这个过程:

```java
processBuilder.directory(new File("/home/"));
Process process = processBuilder.start(); 
```

从这里开始，我们可以通过从`Process`实例访问`InputStream`来获取它:

```java
InputStream inputStream = process.getInputStream(); 
```

当处理完成时，我们可以用以下代码获得退出代码:

```java
int exitCode = process.exitValue(); 
```

**如果我们需要运行额外的命令，我们可以通过在一个`String`数组中传递新的命令和参数来重用`ProcessBuilder` 实例:**

```java
processBuilder.command(
  new String[]{"curl", "-X", "GET", "https://postman-echo.com?foo=bar"}); 
```

最后，为了终止每个`Process`实例，我们应该使用:

```java
process.destroy(); 
```

## 3。`ProcessBuilder` 的简单替代

作为使用`ProcessBuilder`类的替代方法，我们可以使用`Runtime.getRuntime()`来获得`Process`类的一个实例。

让我们看看另一个示例`curl`命令——这次使用的是 **POST** 请求:

```java
curl -X POST https://postman-echo.com/post --data foo1=bar1&foo2;=bar2
```

现在，让我们通过使用`Runtime.getRuntime() `方法来执行命令:

```java
String command = "curl -X POST https://postman-echo.com/post --data foo1=bar1&foo2;=bar2";
Process process = Runtime.getRuntime().exec(command); 
```

首先，我们再次创建一个`Process`类的实例，但是这次使用的是`Runtime.getRuntime()`。我们可以通过调用`getInputStream()`方法得到一个`InputStream`,就像前面的例子一样:

```java
process.getInputStream();
```

当不再需要实例时，我们应该通过调用`destroy()`方法来释放系统资源。

## 4。结论

在本文中，我们展示了在 Java 中使用`curl`的两种方式。

GitHub 上有这个和更多的代码示例。