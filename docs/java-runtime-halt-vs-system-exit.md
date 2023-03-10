# Runtime.getRuntime()。Java 中的 halt()与 System.exit()

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-runtime-halt-vs-system-exit>

## 1.概观

在本教程中，我们将研究[`System.exit()`](/web/20220628051500/https://www.baeldung.com/java-system-exit)`Runtime.getRuntime().halt()`，以及这两种方法之间的比较。

## 2.`System.exit()`

`System.exit()`方法**停止正在运行的 Java 虚拟机**。但是，在停止 JVM 之前，它**调用关闭序列**，也称为有序关闭。请[参考本文](/web/20220628051500/https://www.baeldung.com/adding-shutdown-hooks-for-jvm-applications/)了解更多关于添加关机挂钩的信息。

JVM 的关机序列首先调用所有注册的关机挂钩，并等待它们完成。然后，如果`finalization-on-exit`被启用，它运行所有未调用的终结器。最后，它会暂停 JVM。

这个方法实际上在内部调用了`Runtime.getRuntime().exit()`方法。它以一个整数状态代码作为参数，并有一个`void`返回类型:

```java
public static void exit(int status)
```

如果状态码不为零，则表明程序异常停止。

## 3.`Runtime.getRuntime().halt()`

`Runtime`类允许应用程序与运行该应用程序的环境进行交互。

它有一个`halt`方法，可以用来**强制终止正在运行的 JVM** 。

与`exit`方法不同，该方法不会触发 JVM 关闭序列。因此，当我们调用`halt`方法时，**既不执行** **关闭挂钩，也不执行**终结器。

该方法是非静态的，并且具有与`System.exit()`相似的签名:

```java
public void halt(int status)
```

与`exit`类似，该方法中的非零状态码也表示程序异常终止。

## 4.例子

现在，让我们在关闭挂钩的帮助下，看看`exit`和`halt`方法的例子。

为了简单起见，我们将创建一个 Java 类，并在`static`块中注册一个关闭挂钩。此外，我们将创建两个方法；第一个调用`exit`方法，第二个调用`halt`方法:

```java
public class JvmExitAndHaltDemo {

    private static Logger LOGGER = LoggerFactory.getLogger(JvmExitAndHaltDemo.class);

    static {
        Runtime.getRuntime()
          .addShutdownHook(new Thread(() -> {
            LOGGER.info("Shutdown hook initiated.");
          }));
    }

    public void processAndExit() {
        process();
        LOGGER.info("Calling System.exit().");
        System.exit(0);
    }

    public void processAndHalt() {
        process();
        LOGGER.info("Calling Runtime.getRuntime().halt().");
        Runtime.getRuntime().halt(0);
    }

    private void process() {
        LOGGER.info("Process started.");
    }

}
```

因此，为了首先测试退出方法，让我们创建一个测试用例:

```java
@Test
public void givenProcessComplete_whenExitCalled_thenTriggerShutdownHook() {
    jvmExitAndHaltDemo.processAndExit();
}
```

现在让我们运行测试用例，看看关闭挂钩被调用了:

```java
12:48:43.156 [main] INFO com.baeldung.exitvshalt.JvmExitAndHaltDemo - Process started.
12:48:43.159 [main] INFO com.baeldung.exitvshalt.JvmExitAndHaltDemo - Calling System.exit().
12:48:43.160 [Thread-0] INFO com.baeldung.exitvshalt.JvmExitAndHaltDemo - Shutdown hook initiated.
```

类似地，我们将为`halt`方法创建一个测试用例:

```java
@Test
public void givenProcessComplete_whenHaltCalled_thenDoNotTriggerShutdownHook() {
    jvmExitAndHaltDemo.processAndHalt();
}
```

现在，我们也可以运行这个测试用例，并看到没有调用关闭挂钩:

```java
12:49:16.839 [main] INFO com.baeldung.exitvshalt.JvmExitAndHaltDemo - Process started.
12:49:16.842 [main] INFO com.baeldung.exitvshalt.JvmExitAndHaltDemo - Calling Runtime.getRuntime().halt().
```

## 5.何时使用`exit`和`halt`

正如我们前面看到的，`System.exit()`方法触发 JVM 的关闭序列，而`Runtime.getRuntime().halt()`突然终止 JVM。

我们也可以通过使用操作系统命令来做到这一点。比如我们可以用 SIGINT 或者 Ctrl+C 触发类似`System.exit()`的有序关闭，用 SIGKILL 突然杀死 JVM 进程。

因此，我们很少需要使用这些方法。话虽如此，当我们需要 JVM 运行注册的关闭挂钩或向调用者返回特定的状态代码时，我们可能需要使用`exit`方法，就像使用 shell 脚本一样。

但是，需要注意的是，如果设计不当，关闭挂钩可能会导致死锁。因此， **`exit`方法可能会被阻塞**，因为它会一直等待，直到注册的关闭挂钩完成。因此，一种可能的解决方法是使用`halt` 方法强制 JVM 暂停，以防`exit`阻塞。

最后，应用程序还可以限制这些方法的意外使用。这两个方法都调用 [`SecurityManager`](/web/20220628051500/https://www.baeldung.com/java-security-manager) 类的`checkExit`方法。因此，为了**禁止`exit`和`halt`操作**，应用程序可以使用`SecurityManager`类创建一个安全策略，并从`checkExit`方法中抛出`SecurityException`。

## 6.结论

在本教程中，我们借助一个例子研究了`System.exit()`和`Runtime.getRuntime().halt()`方法。此外，我们还讨论了这些方法的用法和最佳实践。

和往常一样，这篇文章的完整源代码可以在 Github 的[上找到。](https://web.archive.org/web/20220628051500/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-jvm)