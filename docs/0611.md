# 为 JVM 应用程序添加关闭挂钩

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jvm-shutdown-hooks>

## 1.概观

启动一项服务通常很容易。然而，有时我们需要有一个计划来优雅地关闭一个。

在本教程中，我们将看看 JVM 应用程序终止的不同方式。然后，我们将使用 Java APIs 来管理 JVM 关闭挂钩。请参考本文来了解更多关于在 Java 应用程序中关闭 JVM 的信息。

## 2.JVM 关闭

可以通过两种不同的方式关闭 JVM:

1.  受控过程
2.  唐突的态度

**受控进程在以下任一情况下关闭 JVM:**

*   最后一个非[守护进程](/web/20221223070804/https://www.baeldung.com/java-daemon-thread)线程终止。例如，当主线程退出时，JVM 启动它的关闭过程
*   从操作系统发送中断信号。例如，按 Ctrl + C 或注销操作系统
*   从 Java 代码调用`System.exit() `

虽然我们都在努力实现优雅的关闭，但有时 JVM 可能会以突然和意外的方式关闭。**JVM 在以下情况下突然关闭**:

*   从操作系统发送终止信号。例如，通过发出一个`kill -9 <jvm_pid>`
*   从 Java 代码调用`Runtime.getRuntime().halt() `
*   主机操作系统意外死机，例如，在电源故障或操作系统死机时

## 3.关机挂钩

JVM 允许注册函数在完成关闭之前运行。这些功能通常是释放资源或其他类似内务任务的好地方。在 JVM 术语中，**这些函数被称为 s `hutdown hooks`** 。

**关机钩子基本初始化但未启动的线程**。当 JVM 开始它的关闭过程时，它将以一种不确定的顺序启动所有注册的钩子。在运行完所有钩子之后，JVM 将会暂停。

### 3.1.添加挂钩

为了添加一个关机挂钩，我们可以使用`[Runtime.getRuntime().addShutdownHook()](https://web.archive.org/web/20221223070804/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Runtime.html#addShutdownHook(java.lang.Thread)) `方法:

```
Thread printingHook = new Thread(() -> System.out.println("In the middle of a shutdown"));
Runtime.getRuntime().addShutdownHook(printingHook);
```

这里，我们只是在 JVM 自行关闭之前将一些内容打印到标准输出中。如果我们像下面这样关闭 JVM:

```
> System.exit(129);
In the middle of a shutdown
```

然后我们会看到钩子实际上将消息打印到标准输出。

JVM 负责启动钩子线程。因此，如果给定的钩子已经启动，Java 将抛出一个异常:

```
Thread longRunningHook = new Thread(() -> {
    try {
        Thread.sleep(300);
    } catch (InterruptedException ignored) {}
});
longRunningHook.start();

assertThatThrownBy(() -> Runtime.getRuntime().addShutdownHook(longRunningHook))
  .isInstanceOf(IllegalArgumentException.class)
  .hasMessage("Hook already running"); 
```

显然，我们也不能多次注册一个钩子:

```
Thread unfortunateHook = new Thread(() -> {});
Runtime.getRuntime().addShutdownHook(unfortunateHook);

assertThatThrownBy(() -> Runtime.getRuntime().addShutdownHook(unfortunateHook))
  .isInstanceOf(IllegalArgumentException.class)
  .hasMessage("Hook previously registered");
```

### 3.2.拆卸挂钩

Java 提供了一个双`remove `方法来删除注册后的特定关闭挂钩:

```
Thread willNotRun = new Thread(() -> System.out.println("Won't run!"));
Runtime.getRuntime().addShutdownHook(willNotRun);

assertThat(Runtime.getRuntime().removeShutdownHook(willNotRun)).isTrue(); 
```

当关闭挂钩成功移除时， [`removeShutdownHook()`](https://web.archive.org/web/20221223070804/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Runtime.html#removeShutdownHook(java.lang.Thread)) 方法返回`true `。

### 3.3.警告

JVM 只在正常终止的情况下运行关闭挂钩。所以，当外力突然杀死 JVM 进程时，JVM 就没有机会执行 shutdown 钩子了。此外，从 Java 代码中暂停 JVM 也会产生相同的效果:

```
Thread haltedHook = new Thread(() -> System.out.println("Halted abruptly"));
Runtime.getRuntime().addShutdownHook(haltedHook);

Runtime.getRuntime().halt(129); 
```

[`halt`](https://web.archive.org/web/20221223070804/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Runtime.html#halt(int)) 方法强制终止当前运行的 JVM。因此，注册的关闭挂钩将没有机会执行。

## 4.结论

在本教程中，我们看了 JVM 应用程序可能终止的不同方式。然后，我们使用一些运行时 API 来注册和注销关闭挂钩。

像往常一样，GitHub 上的示例代码[可用。](https://web.archive.org/web/20221223070804/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-jvm-2)