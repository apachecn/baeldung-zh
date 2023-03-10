# java.lang.Process API 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-process-api>

## 1。简介

在本教程中，我们将深入了解一下`Process` API 。

关于如何使用`Process`来执行 shell 命令，我们可以参考之前的教程[这里是](/web/20221113233803/https://www.baeldung.com/run-shell-command-in-java)。

它引用的进程是一个正在执行的应用程序。`Process`类提供了与这些过程交互的方法，包括提取输出、执行输入、监控生命周期、检查退出状态以及销毁(终止)它。

## 2。使用`Process`类编译运行 Java 程序

让我们看一个借助于`Process` API 编译和运行另一个 Java 程序的例子:

```java
@Test
public void whenExecutedFromAnotherProgram_thenSourceProgramOutput3() throws IOException {

    Process process = Runtime.getRuntime()
      .exec("javac -cp src src\\main\\java\\com\\baeldung\\java9\\process\\OutputStreamExample.java");
    process = Runtime.getRuntime() 
      .exec("java -cp src/main/java com.baeldung.java9.process.OutputStreamExample");
    BufferedReader output = new BufferedReader(new InputStreamReader(process.getInputStream()));
    int value = Integer.parseInt(output.readLine());

    assertEquals(3, value);
}
```

因此，在现有 Java 代码中执行 Java 代码的应用实际上是无限的。

## 3。创建过程

我们的 Java 应用程序可以调用在我们的计算机系统中运行的任何应用程序，而不受操作系统的限制。

因此我们可以执行应用程序。让我们看看利用流程 API 可以运行哪些不同的用例。

`ProcessBuilder`类允许我们在应用程序中创建子流程。

让我们来看一个打开基于 Windows 的记事本应用程序的演示:

```java
ProcessBuilder builder = new ProcessBuilder("notepad.exe");
Process process = builder.start();
```

## 4。销毁过程

`Process`也为我们提供了破坏子过程或过程的方法。**虽然，应用如何被杀死是平台相关的**。

让我们看看可能的不同用例。

### 4.1。通过引用销毁进程

假设我们使用的是 Windows 操作系统，想要生成记事本应用程序并销毁它。

和以前一样，我们可以通过使用`ProcessBuilder`类和`start()`方法创建一个记事本应用程序的实例。

然后我们可以在我们的`Process`对象上调用`destroy()`方法。

### 4.2。通过 ID 销毁进程

我们还可以终止运行在操作系统中的进程，这些进程可能不是由我们的应用程序创建的。

执行此操作时应小心谨慎，因为我们可能会在不知不觉中破坏一个可能导致操作系统不稳定的关键进程。

我们首先需要通过检查任务管理器找出当前运行进程的进程 ID，并找出 pid。

让我们看一个例子:

```java
long pid = /* PID to kill */;
Optional<ProcessHandle> optionalProcessHandle = ProcessHandle.of(pid);
optionalProcessHandle.ifPresent(processHandle -> processHandle.destroy()); 
```

### 4.3。 **强行破坏进程**

在执行`destroy()`方法时，正如我们在本文前面看到的，子流程将被终止。

**在`destroy()`不起作用的情况下，我们有`destroyForcibly()`的选择。**

我们应该总是先从`destroy()`方法开始。之后，我们可以通过执行`isAlive()`来快速检查子流程。

如果返回 true，则执行`destroyForcibly()`:

```java
ProcessBuilder builder = new ProcessBuilder("notepad.exe");
Process process = builder.start();
process.destroy();
if (process.isAlive()) {
    process.destroyForcibly();
}
```

## 5。等待流程完成

我们还有两个重载方法，通过它们我们可以确保我们可以等待一个进程的完成。

### 5.1。`waitfor()`

当这个方法被执行时，它会将当前执行的进程线程**置于阻塞等待状态，除非子进程被终止**。

让我们来看看这个例子:

```java
ProcessBuilder builder = new ProcessBuilder("notepad.exe");
Process process = builder.start();
assertThat(process.waitFor() >= 0); 
```

从上面的例子中我们可以看到，当前线程要继续执行，它会一直等待子进程线程结束。一旦子进程结束，当前线程将继续执行。

### 5.2。`waitfor(long timeOut, TimeUnit time)`

当这个方法被执行时，它将把当前执行的进程线程**置于阻塞等待状态，除非子进程被终止或者超时**。

让我们来看看这个例子:

```java
ProcessBuilder builder = new ProcessBuilder("notepad.exe");
Process process = builder.start();
assertFalse(process.waitFor(1, TimeUnit.SECONDS));
```

从上面的例子中我们可以看到，当前线程要继续执行，它会一直等待子进程线程结束，或者等待指定的时间间隔结束。

当执行此方法时，如果子流程已经退出，它将返回布尔值 true 如果在子流程退出之前等待时间已过，它将返回布尔值 false。

## 6。`exitValue()`

当这个方法运行时，当前线程不会等待子进程终止或销毁，但是，如果子进程没有终止，它会抛出一个`IllegalThreadStateException`。

**换句话说，如果子流程已经成功终止，那么它将导致流程**的退出值。

它可以是任何可能的正整数。

让我们看一个例子，当子流程成功终止时，`exitValue()`方法返回一个正整数:

```java
@Test
public void 
  givenSubProcess_whenCurrentThreadWillNotWaitIndefinitelyforSubProcessToEnd_thenProcessExitValueReturnsGrt0() 
  throws IOException {
    ProcessBuilder builder = new ProcessBuilder("notepad.exe");
    Process process = builder.start();
    assertThat(process.exitValue() >= 0);
}
```

## 7。`isAlive()`

当我们希望执行业务处理时，这是一个主观的过程，不管这个过程是否是活动的。

我们可以执行一个快速检查来发现进程是否是活动的，这将返回一个布尔值。

让我们看一个简单的例子:

```java
ProcessBuilder builder = new ProcessBuilder("notepad.exe");
Process process = builder.start();
Thread.sleep(10000);
process.destroy();
assertTrue(process.isAlive());
```

## 8。处理工艺流

默认情况下，创建的子流程没有终端或控制台。它的所有标准 I/O(即 stdin、stdout、stderr)操作都将被发送到父进程。因此，父流程可以使用这些流向子流程提供输入并从中获取输出。

因此，这给了我们巨大的灵活性，因为它给了我们对子流程的输入/输出的控制。

### 8.1。`getErrorStream()`

有趣的是，我们可以获取子流程生成的错误，并在此基础上执行业务处理。

之后，我们可以根据我们的需求执行特定的业务处理检查。

让我们看一个例子:

```java
@Test
public void givenSubProcess_whenEncounterError_thenErrorStreamNotNull() throws IOException {
    Process process = Runtime.getRuntime().exec(
      "javac -cp src src\\main\\java\\com\\baeldung\\java9\\process\\ProcessCompilationError.java");
    BufferedReader error = new BufferedReader(new InputStreamReader(process.getErrorStream()));
    String errorString = error.readLine();
    assertNotNull(errorString);
}
```

### 8.2。`getInputStream()`

我们还可以获取子流程生成的输出，并在父流程中使用，从而允许流程之间共享信息:

```java
@Test
public void givenSourceProgram_whenReadingInputStream_thenFirstLineEquals3() throws IOException {
    Process process = Runtime.getRuntime().exec(
      "javac -cp src src\\main\\java\\com\\baeldung\\java9\\process\\OutputStreamExample.java");
    process = Runtime.getRuntime()
      .exec("java -cp  src/main/java com.baeldung.java9.process.OutputStreamExample");
    BufferedReader output = new BufferedReader(new InputStreamReader(process.getInputStream()));
    int value = Integer.parseInt(output.readLine());

    assertEquals(3, value);
}
```

### 8.3。`ge`T3****

我们可以从父流程向子流程发送输入:

```java
Writer w = new OutputStreamWriter(process.getOutputStream(), "UTF-8");
w.write("send to child\n");
```

### 8.4。过滤过程流

这是与选择性运行流程进行交互的完美有效的用例。

`Process`为我们提供了基于某个谓词有选择地过滤正在运行的流程的工具。

之后，我们可以在这个选择性的过程集上执行业务操作:

```java
@Test
public void givenRunningProcesses_whenFilterOnProcessIdRange_thenGetSelectedProcessPid() {
    assertThat(((int) ProcessHandle.allProcesses()
      .filter(ph -> (ph.pid() > 10000 && ph.pid() < 50000))
      .count()) > 0);
}
```

## 9。结论

`Process`是一个功能强大的操作系统级交互类。触发终端命令以及启动、监控和终止应用程序。

关于 Java 9 Process API 的更多阅读，请看我们的文章[这里](/web/20221113233803/https://www.baeldung.com/java-9-process-api)。

和往常一样，你可以在 Github 上找到来源[。](https://web.archive.org/web/20221113233803/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-os)