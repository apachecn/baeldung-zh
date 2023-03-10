# 用 IntelliJ IDEA 进行远程调试

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/intellij-remote-debugging>

## 1.介绍

远程调试使开发人员能够诊断服务器或另一个进程上的独特错误。它提供了跟踪那些恼人的运行时错误并识别性能瓶颈和资源接收器的方法。

在本教程中，我们将看看使用 JetBrains IntelliJ IDEA 的远程调试。让我们首先通过修改 JVM 来准备我们的示例应用程序。

## 2.配置 JVM

我们将使用 Spring scheduler 示例应用程序来轻松地连接定期调度的任务并向其添加断点。

此外， **IntelliJ IDEA 提供了我们的 JVM 参数作为配置的一部分**:

```java
-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:5005
```

### 2.1.JVM 参数

除了 Java 调试有线协议(JDWP)配置—`jdwp=transport=dt_socket` —我们还可以看到`server`、`suspend`和`address`参数。

`server`参数将 JVM 配置为调试器的目标。`suspend`参数告诉 JVM 在启动前等待调试器客户端连接。最后，`address`参数使用通配符主机和声明的端口。

因此，让我们构建调度程序应用程序:

```java
mvn clean package
```

**现在让我们启动应用程序，包括`-agentlib:jdwp`参数:**

```java
java -jar -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:5005 \
  target/gs-scheduling-tasks-0.1.0.jar
```

打开任何终端并运行命令。随着我们的应用程序启动，现在让我们切换到 IntelliJ。

## 3.在 IntelliJ IDEA 中运行配置

接下来，在 IntelliJ 中，我们为远程调试创建一个新的运行配置:

[![](img/d1c7988b7470fae2ac7d815cada1de76.png)](/web/20220703152239/https://www.baeldung.com/wp-content/uploads/2019/11/run_configuration.png)

现在我们的应用程序正在运行，让我们通过点击`Debug`按钮来启动远程调试会话。

## 4.远程调试

接下来，我们打开`ScheduleTask`文件，在第 36 行放置一个断点，如下所示:

```java
public void reportCurrentTime() {
  log.info("The time is now {}", dateFormat.format(new Date()));
}
```

因为任务每五秒执行一次，所以它在添加后会很快停止。因此，我们现在可以单步执行整个应用程序。

对于应用程序启动问题，我们将`suspend`标志改为`n `，并在`Application.`的`main` 方法中放置一个断点

### 4.1.限制

远程调试时，有时日志记录和输出会让我们感到困惑。日志不会发送到 IDE 控制台，因此可以使用外部日志文件并将其映射到 IDE 中，以获得更强大的调试功能。

还要记住，虽然远程调试是一个非常强大的工具，**生产环境并不是调试**的合适目标。

## 5.结论

正如我们在本文中所提到的，使用 IntelliJ 进行远程调试只需几个简单的步骤就可以轻松设置和使用。

我们研究了如何配置应用程序 JVM 进行调试，以及这个重要工具在开发人员工具箱中的一些限制。

示例应用程序可以在 GitHub 上找到[。](https://web.archive.org/web/20220703152239/https://github.com/eugenp/tutorials/tree/master/spring-scheduling)