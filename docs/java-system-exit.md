# System.exit()指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-system-exit>

## 1。概述

在本教程中，我们将看看`System.exit`在 Java 中是什么意思。

我们会看到它的用途，用在哪里，怎么用。我们还将看到用不同的状态代码调用它有什么不同。

## 2。什么是`System.exit`？

`System.exit`是一种`void`方法。它接受一个退出代码，并将其传递给调用脚本或程序。

代码为**0 的退出表示正常退出:**

```
System.exit(0);
```

我们可以将任何整数作为参数传递给该方法。**非零状态代码被视为异常退出。**

调用`System.exit` 方法终止当前运行的 JVM 并退出程序。此方法通常不会返回。

这意味着`System.exit`之后的**代码实际上是不可到达的** **，然而编译器并不知道。**

```
System.exit(0);
System.out.println("This line is unreachable");
```

**用`System.exit(0)`** 关闭程序不是个好主意。它给出了与退出`main`方法相同的结果，并且也停止了后续行的执行，**调用`System.exit`的线程也会阻塞，直到 JVM 终止。如果一个关闭钩子向这个线程提交一个任务，就会导致死锁。**

## 3。我们为什么需要它？

`System.exit`的典型用例是当出现异常情况时，我们需要立即退出程序。

同样，如果我们必须从 main 方法之外的地方终止程序，`System.exit`是实现它的一种方法。

## 4。我们什么时候需要它？

脚本依赖于它调用的命令的退出代码是很常见的。如果这样的命令是一个 Java 应用程序，那么`System.exit`可以方便地发送这个退出代码。

例如，我们可以返回一个异常退出代码，然后调用脚本可以解释这个代码，而不是抛出一个异常。

或者，我们可以使用`System.exit`来调用我们已经注册的任何关闭挂钩。这些钩子可以被设置为清理持有的资源，并从其他非[守护进程](/web/20221023124834/https://www.baeldung.com/java-daemon-thread)线程中安全退出。

## 5。一个简单的例子

在这个例子中，我们试图读取一个文件，如果它存在，我们打印其中的一行。如果文件不存在，我们用 catch 块中的`System.exit`退出程序。

```
try {
    BufferedReader br = new BufferedReader(new FileReader("file.txt"));
    System.out.println(br.readLine());
    br.close();
} catch (IOException e) {
    System.exit(2);
} finally {
    System.out.println("Exiting the program");
}
```

这里，我们必须注意，如果没有找到文件，finally 块不会被执行。因为 catch 块上的`System.exit`退出 JVM，不允许`finally` 块执行。

## 6。选择状态代码

我们可以将任何整数作为状态码传递，但是通常的做法是状态码为 0 的`System.exit`是正常的，其他的是异常退出。

注意，这只是一个“好的实践”,并不是编译器关心的严格规则。

另外，值得注意的是，当我们从命令行调用一个 Java 程序时，状态代码也被考虑在内。

在下面的例子中，当我们试图执行`SystemExitExample.class,`时，如果它通过调用带有非零状态码的`System.exit` 退出 JVM，那么下面的 echo 就不会被打印出来。

```
java SystemExitExample && echo "I will not be printed"
```

为了使我们的程序能够与其他标准工具通信，我们可以考虑遵循相关系统用于通信的标准代码。

[具有特殊含义的退出代码](https://web.archive.org/web/20221023124834/https://tldp.org/LDP/abs/html/exitcodes.html)由 Linux 文档项目准备的文档提供了保留代码的列表。它还建议在特定情况下使用什么代码。

## 7。结论

在本教程中，我们讨论了`System.exit`如何工作，何时使用，以及如何使用。

在使用应用服务器和其他常规应用程序时，使用[异常处理](/web/20221023124834/https://www.baeldung.com/java-exceptions)或普通返回语句来退出程序是一个很好的实践。使用`System.exit`方法更适合基于脚本的应用程序或任何解释状态代码的地方。

你可以在 GitHub 上查看本文[中提供的例子。](https://web.archive.org/web/20221023124834/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-jvm)