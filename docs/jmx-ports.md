# JMX 港口

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jmx-ports>

## 1.概观

在本教程中，我们将解释为什么 JMX 在启动时打开三个端口。此外，我们将展示如何在 Java 中启动 JMX。之后，我们将展示如何限制打开的端口数量。

## 2.JMX 定义

让我们首先定义什么是 JMX 框架。 **[Java 管理扩展](/web/20220625225825/https://www.baeldung.com/java-management-extensions) (JMX)** 框架为管理 Java 应用程序提供了一个可配置的、可伸缩的、可靠的基础设施。此外，它为应用程序的实时管理定义了 MBean 的概念。该框架允许本地或远程管理应用程序。

## 3.在 Java 中启用 JMX

现在让我们看看如何启用 JMX。**对于 Java 版及更早版本，有一个系统属性** `**com.sun.management.jmxremote**.` 用该属性启动的应用程序允许从本地和远程连接到 [JConsole](/web/20220625225825/https://www.baeldung.com/java-management-extensions#1-connecting-from-the-client-side) 。另一方面，在没有属性的情况下启动时，应用程序在 JConsole 中是不可见的。

**不过从 Java 6 及以上开始，该参数就没必要了**。应用程序在启动后可自动用于管理。此外，默认配置会自动分配端口，并且只在本地公开它。

## 4.JMX 港口

在我们的例子中，我们将使用 Java 6 或更高版本。首先，让我们创建一个无限循环的类。该类什么也不做，但是它允许我们检查哪些端口是打开的:

```java
public class JMXConfiguration {

    public static void main(String[] args) {
        while (true) {
            // to ensure application does not terminate
        }
    }
}
```

现在，我们将编译该类并启动它:

```java
java com.baeldung.jmx.JMXConfiguration
```

之后，我们可以**检查哪个 pid 被分配给进程，并检查进程**打开的端口:

```java
netstat -ao | grep <pid>
```

因此，我们将获得应用程序公开的端口列表:

```java
Active Connections
Proto  Local Address          Foreign Address        State           PID
TCP    127.0.0.1:55846        wujek:55845            ESTABLISHED     2604
```

另外，**在重启的情况下，端口会改变**。它是随机分配的。这个功能从 Java 6 开始就有了，它自动公开 Java Attach API 的应用程序。换句话说，它通过本地进程自动公开 JConsole 连接的应用程序。

现在让我们通过向 JVM 提供选项来启用远程连接:

```java
-Dcom.sun.management.jmxremote=true
-Dcom.sun.management.jmxremote.port=1234
-Dcom.sun.management.jmxremote.authenticate=false
-Dcom.sun.management.jmxremote.ssl=false
```

端口号是我们必须提供的强制参数，以便为远程连接公开 JMX。我们仅出于测试目的禁用了身份验证和 SSL。

现在， [`netstat`](/web/20220625225825/https://www.baeldung.com/linux/find-process-using-port#netstat) 命令返回:

```java
Proto  Local Address    Foreign Address State       PID
TCP    0.0.0.0:1234     wujek:0         LISTENING   11088
TCP    0.0.0.0:58738    wujek:0         LISTENING   11088
TCP    0.0.0.0:58739    wujek:0         LISTENING   11088 
```

正如我们所看到的，应用程序公开了三个端口。RMI/JMX 公开了两个端口。第三个是用于本地连接的随机端口。

## 5.限制打开的端口数量

首先，我们可以使用`-XX:+DisableAttachMechanism`选项禁止从 JConsole 公开本地连接的应用程序:

```java
java -XX:+DisableAttachMechanism com.baeldung.jmx.JMXConfiguration
```

之后，应用程序不会暴露任何 JMX/RMI 端口。

此外，从 JDK 16 开始，我们可以设置本地端口号:

```java
java 
  -Dcom.sun.management.jmxremote=true 
  -Dcom.sun.management.jmxremote.local.port=1235 
  com.baeldung.jmx.JMXConfiguration
```

现在，让我们更改配置，使用远程端口。

还有一个额外的选项`-Dcom.sun.management.jmxremote.rmi.port=1234`，允许我们将 RMI 端口设置为与 JMX 端口相同的值。现在，完整的命令是:

```java
java 
  -Dcom.sun.management.jmxremote=true 
  -Dcom.sun.management.jmxremote.port=1234 
  -Dcom.sun.management.jmxremote.rmi.port=1234 
  -Dcom.sun.management.jmxremote.local.port=1235 
  -Dcom.sun.management.jmxremote.authenticate=false 
  -Dcom.sun.management.jmxremote.ssl=false 
  com.baeldung.jmx.JMXConfiguration
```

之后，`netstat`命令返回:

```java
Proto  Local Address    Foreign Address State       PID
TCP    0.0.0.0:1234     wujek:0         LISTENING   19504
TCP    0.0.0.0:1235     wujek:0         LISTENING   19504
```

也就是说，应用程序只公开了两个端口，一个用于 JMX/RMI 远程连接，一个用于本地连接。得益于此，我们可以完全控制公开的端口，并避免与其他进程公开的端口发生冲突。

但是，当我们启用远程连接并禁用连接机制时:

```java
java 
  -XX:+DisableAttachMechanism 
  -Dcom.sun.management.jmxremote=true 
  -Dcom.sun.management.jmxremote.port=1234 
  -Dcom.sun.management.jmxremote.rmi.port=1234 
  -Dcom.sun.management.jmxremote.authenticate=false 
  -Dcom.sun.management.jmxremote.ssl=false 
  com.baeldung.jmx.JMXConfiguration
```

然后，应用程序仍然公开两个端口:

```java
Proto Local Address     Foreign Address     State       PID
TCP   0.0.0.0:1234      wujek:0             LISTENING   9856
TCP   0.0.0.0:60565     wujek:0             LISTENING   9856
```

## 6.结论

在这篇短文中，我们解释了如何在 Java 中启动 JMX。然后，我们展示了 JMX 在启动时打开哪些端口。最后，我们介绍了如何限制 JMX 开放的端口数量。

和往常一样，这个例子的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220625225825/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-perf)