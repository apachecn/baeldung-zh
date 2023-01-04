# 如何在 Java 中运行 Shell 命令

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/run-shell-command-in-java>

## 1。概述

在本教程中，我们将展示从`Java`代码中执行 shell 命令的两种方式。

第一种是使用`Runtime`类并调用它的`exec`方法。

第二种也是更可定制的方式是创建和使用一个`ProcessBuilder`实例。

## 2。操作系统依赖性

在我们将要创建一个新的执行 shell 命令的`Process`之前，我们需要首先确定运行我们的`JVM`的操作系统。

这是因为，在`Windows`上，我们需要运行我们的命令作为`cmd.exe` shell 的参数，而在所有其他操作系统上，我们可以发布一个标准的 shell，叫做`sh:`

```
boolean isWindows = System.getProperty("os.name")
  .toLowerCase().startsWith("windows");
```

## 3.输入和输出

此外，我们需要一种方法来连接到流程的输入和输出流。

至少**输出必须被消耗**——否则我们的进程不会成功返回，而是会挂起。

让我们实现一个常用的名为`StreamGobbler`的类，它使用一个`InputStream`:

```
private static class StreamGobbler implements Runnable {
    private InputStream inputStream;
    private Consumer<String> consumer;

    public StreamGobbler(InputStream inputStream, Consumer<String> consumer) {
        this.inputStream = inputStream;
        this.consumer = consumer;
    }

    @Override
    public void run() {
        new BufferedReader(new InputStreamReader(inputStream)).lines()
          .forEach(consumer);
    }
}
```

**注意:**这个类实现了`Runnable`接口，这意味着它可以被任何`[Executor](/web/20221002220632/https://www.baeldung.com/java-executor-service-tutorial)`执行。

## 4。`Runtime.exec()`

对`Runtime.exec()`的方法调用是一种简单的、还不可定制的方式来产生一个新的子流程。

在下面的示例中，我们将请求一个用户主目录的目录列表，并将其打印到控制台:

```
String homeDirectory = System.getProperty("user.home");
Process process;
if (isWindows) {
    process = Runtime.getRuntime()
      .exec(String.format("cmd.exe /c dir %s", homeDirectory));
} else {
    process = Runtime.getRuntime()
      .exec(String.format("sh -c ls %s", homeDirectory));
}
StreamGobbler streamGobbler = 
  new StreamGobbler(process.getInputStream(), System.out::println);
Future<?> future = Executors.newSingleThreadExecutor().submit(streamGobbler);

int exitCode = process.waitFor();
assert exitCode == 0;

future.get(); // waits for streamGobbler to finish
```

当您想要使用过程的结果时，确保调用 `future.get()`方法来等待计算完成。

## 5。`ProcessBuilder`

对于我们计算问题的第二个实现，我们将使用一个`ProcessBuilder`。这优于`Runtime`方法，因为我们能够定制一些细节。

例如，我们能够:

*   使用`builder.directory()`改变运行 shell 命令的工作目录
*   使用`builder.environment()`设置自定义键值映射作为环境
*   将输入和输出流重定向到自定义替换
*   使用`builder.inheritIO()`将它们继承到当前`JVM`进程的流中

```
ProcessBuilder builder = new ProcessBuilder();
if (isWindows) {
    builder.command("cmd.exe", "/c", "dir");
} else {
    builder.command("sh", "-c", "ls");
}
builder.directory(new File(System.getProperty("user.home")));
Process process = builder.start();
StreamGobbler streamGobbler = 
  new StreamGobbler(process.getInputStream(), System.out::println);
Future<?> future = Executors.newSingleThreadExecutor().submit(streamGobbler);
int exitCode = process.waitFor();
assert exitCode == 0;
future.get(10, TimeUnit.SECONDS)
```

## 6。结论

正如我们在这个快速教程中看到的，我们可以用两种不同的方式在`Java`中执行 shell 命令。

通常，如果您计划定制衍生进程的执行，例如，更改其工作目录，您应该考虑使用`ProcessBuilder`。

和往常一样，你会找到来源。