# Java 9 流程 API 改进

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-9-process-api>

## 1。概述

在 Java 5 之前，Java 中的流程 API 相当原始，产生新流程的唯一方法是使用`Runtime.getRuntime().exec()` API。然后在 Java 5 中，`ProcessBuilder` API 被引入，它支持一种更干净的方式来产生新的进程。

Java 9 增加了一种获取当前进程和任何衍生进程信息的新方法。

在本文中，我们将研究这两种增强。

## 2。当前 Java 进程信息

我们现在可以通过 API `java.lang.ProcessHandle.Info` API 获得关于流程的大量信息:

*   用于启动进程的命令
*   该命令的参数
*   流程开始的时间
*   它和创建它的用户所花费的总时间

我们可以这样做:

```
private static void infoOfCurrentProcess() {
    ProcessHandle processHandle = ProcessHandle.current();
    ProcessHandle.Info processInfo = processHandle.info();

    log.info("PID: " + processHandle.pid());
    log.info("Arguments: " + processInfo.arguments());
    log.info("Command: " + processInfo.command());
    log.info("Instant: " + processInfo.startInstant());
    log.info("Total CPU duration: " + processInfo.totalCpuDuration());
    log.info("User: " + processInfo.user());
}
```

需要注意的是，`java.lang.ProcessHandle.Info`是在另一个接口`java.lang.ProcessHandle`中定义的公共接口。JDK 提供者(Oracle JDK、Open JDK、Zulu 或其他)应该为这些接口提供实现，以使这些实现返回流程的相关信息。

输出取决于操作系统和 Java 版本。以下是输出结果的一个示例:

```
16:31:24.784 [main] INFO  c.b.j.process.ProcessAPIEnhancements - PID: 22640
16:31:24.790 [main] INFO  c.b.j.process.ProcessAPIEnhancements - Arguments: Optional[[Ljava.lang.String;@2a17b7b6]
16:31:24.791 [main] INFO  c.b.j.process.ProcessAPIEnhancements - Command: Optional[/Library/Java/JavaVirtualMachines/jdk-13.0.1.jdk/Contents/Home/bin/java]
16:31:24.795 [main] INFO  c.b.j.process.ProcessAPIEnhancements - Instant: Optional[2021-08-31T14:31:23.870Z]
16:31:24.795 [main] INFO  c.b.j.process.ProcessAPIEnhancements - Total CPU duration: Optional[PT0.818115S]
16:31:24.796 [main] INFO  c.b.j.process.ProcessAPIEnhancements - User: Optional[username]
```

## 3。衍生的进程信息

还可以获得新产生的进程的进程信息。在这种情况下，在我们生成流程并获得一个`java.lang.Process`的实例后，我们调用它的`toHandle()`方法来获得一个`java.lang.ProcessHandle`的实例。

其余细节与上一节相同:

```
String javaCmd = ProcessUtils.getJavaCmd().getAbsolutePath();
ProcessBuilder processBuilder = new ProcessBuilder(javaCmd, "-version");
Process process = processBuilder.inheritIO().start();
ProcessHandle processHandle = process.toHandle();
```

## 4。枚举系统中的活动进程

我们可以列出系统中当前的所有进程，这些进程对当前进程是可见的。返回的列表是调用 API 时的快照，因此有可能一些进程在获取快照后终止，或者添加了一些新的进程。

为此，我们可以使用在`java.lang.ProcessHandle`接口中可用的静态方法`allProcesses()`，它返回给我们一个`ProcessHandle:`的`Stream`

```
private static void infoOfLiveProcesses() {
    Stream<ProcessHandle> liveProcesses = ProcessHandle.allProcesses();
    liveProcesses.filter(ProcessHandle::isAlive)
        .forEach(ph -> {
            log.info("PID: " + ph.pid());
            log.info("Instance: " + ph.info().startInstant());
            log.info("User: " + ph.info().user());
        });
}
```

## 5。枚举子进程

有两种方法可以做到这一点:

*   获取当前流程的直接子流程
*   获取当前进程的所有后代

前者通过使用方法`children()`实现，后者通过使用方法`descendants()`实现:

```
private static void infoOfChildProcess() throws IOException {
    int childProcessCount = 5;
    for (int i = 0; i < childProcessCount; i++) {
        String javaCmd = ProcessUtils.getJavaCmd()
          .getAbsolutePath();
        ProcessBuilder processBuilder
          = new ProcessBuilder(javaCmd, "-version");
        processBuilder.inheritIO().start();
    }

    Stream<ProcessHandle> children = ProcessHandle.current()
      .children();
    children.filter(ProcessHandle::isAlive)
      .forEach(ph -> log.info("PID: {}, Cmd: {}", ph.pid(), ph.info()
        .command()));
    Stream<ProcessHandle> descendants = ProcessHandle.current()
      .descendants();
    descendants.filter(ProcessHandle::isAlive)
      .forEach(ph -> log.info("PID: {}, Cmd: {}", ph.pid(), ph.info()
        .command()));
}
```

## 6。在流程终止时触发相关动作

我们可能希望在进程终止时运行一些东西。这可以通过使用`java.lang.ProcessHandle`接口中的`onExit()`方法来实现。该方法返回给我们一个 [`CompletableFuture`](/web/20221228152503/https://www.baeldung.com/java-completablefuture) ，它提供了在`CompletableFuture`完成时触发相关操作的能力。

这里，`CompletableFuture`表示进程已经完成，但是进程是否成功完成并不重要。我们调用`CompletableFuture`上的`get()`方法，等待其完成:

```
private static void infoOfExitCallback() throws IOException, InterruptedException, ExecutionException {
    String javaCmd = ProcessUtils.getJavaCmd()
      .getAbsolutePath();
    ProcessBuilder processBuilder
      = new ProcessBuilder(javaCmd, "-version");
    Process process = processBuilder.inheritIO()
      .start();
    ProcessHandle processHandle = process.toHandle();

    log.info("PID: {} has started", processHandle.pid());
    CompletableFuture onProcessExit = processHandle.onExit();
    onProcessExit.get();
    log.info("Alive: " + processHandle.isAlive());
    onProcessExit.thenAccept(ph -> {
        log.info("PID: {} has stopped", ph.pid());
    });
}
```

在`java.lang.Process`接口中也可以使用`onExit()`方法。

## 7。结论

在本教程中，我们介绍了 Java 9 中对`Process` API 的有趣补充，这些补充让我们对正在运行的和衍生的进程有了更多的控制。

本文中使用的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221228152503/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-os)