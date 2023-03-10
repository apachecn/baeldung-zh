# java.lang.ProcessBuilder API 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-lang-processbuilder-api>

## 1。概述

[进程 API](/web/20221228152449/https://www.baeldung.com/java-process-api) 提供了一种在 Java 中执行操作系统命令的强大方法。但是，它有几个选项，可能会使其使用起来很麻烦。

在本教程中，**我们将看看 Java 如何通过`ProcessBuilder` API 来缓解这个问题。**

## 2。`ProcessBuilder` API

`ProcessBuilder`类提供了创建和配置操作系统进程的方法。**每个`ProcessBuilder`实例允许我们管理一组流程属性**。然后我们可以用这些给定的属性开始一个新的`Process`。

下面是一些我们可以使用该 API 的常见场景:

*   查找当前的 Java 版本
*   为我们的环境设置自定义键值映射
*   更改运行 shell 命令的工作目录
*   将输入和输出流重定向到自定义替换
*   继承当前 JVM 进程的两个流
*   [从 Java 代码执行 shell 命令](/web/20221228152449/https://www.baeldung.com/run-shell-command-in-java)

在后面的章节中，我们将看一看这些方法的实际例子。

但是在我们深入工作代码之前，让我们看一下这个 API 提供了什么样的功能。

### 2.1。方法总结

在本节中，**我们将后退一步，简要地看一下`ProcessBuilder`类**中最重要的方法。这将有助于我们以后深入研究一些真实的例子:

*   ```java
    ProcessBuilder(String... command)
    ```

    要使用指定的操作系统程序和参数创建新的 process builder，我们可以使用这个方便的构造函数。

*   ```java
    directory(File directory)
    ```

    我们可以通过调用`directory`方法并传递一个`File`对象来覆盖当前进程的默认工作目录。**默认情况下，当前工作目录设置为`user.dir` 系统属性**返回的值。

*   ```java
    environment()
    ```

    如果我们想获得当前的环境变量，我们可以简单地调用 `environment` 方法。它返回给我们一个 当前进程环境的副本，使用 `System.getenv()` 但是作为 `Map` 。

*   ```java
    inheritIO()
    ```

    如果我们想要指定我们的子进程标准 I/O 的源和目的地应该与当前 Java 进程的相同，我们可以使用`inheritIO`方法。

*   ```java
    redirectInput(File file), redirectOutput(File file), redirectError(File file)
    ```

    当我们想要将 process builder 的标准输入、输出和错误目标重定向到一个文件时，我们可以使用这三种类似的重定向方法。

*   ```java
    start()
    ```

    最后但同样重要的是，要用我们已经配置好的东西启动一个新的进程，我们只需调用`start()`。

**我们应该注意，这个类不是同步的**。例如，如果我们有多个线程同时访问一个`ProcessBuilder`实例，那么同步必须在外部管理。

## 3。示例

现在我们已经对`ProcessBuilder` API 有了基本的了解，让我们来看一些例子。

### 3.1。使用`ProcessBuilder`打印 Java 版本

**在第一个例子中，我们将运行带有一个参数的`java`命令来获取版本**。

```java
Process process = new ProcessBuilder("java", "-version").start();
```

首先，我们创建我们的`ProcessBuilder`对象，将命令和参数值传递给构造函数。接下来，我们开始使用`start()`方法获取一个`Process`对象。

现在让我们看看如何处理输出:

```java
List<String> results = readOutput(process.getInputStream());

assertThat("Results should not be empty", results, is(not(empty())));
assertThat("Results should contain java version: ", results, hasItem(containsString("java version")));

int exitCode = process.waitFor();
assertEquals("No errors should be detected", 0, exitCode);
```

在这里，我们读取流程输出，并验证内容是否符合我们的预期。在最后一步，我们使用`process.waitFor()`等待流程结束。

**流程完成后，返回值告诉我们流程是否成功**。

需要记住几个要点:

*   参数必须按正确的顺序排列
*   此外，在这个例子中，使用了默认的工作目录和环境
*   我们故意不调用`process.waitFor()`,直到我们已经读取了输出，因为输出缓冲区可能会拖延进程
*   我们假设通过`PATH`变量可以使用`java`命令

### 3.2。使用修改后的环境启动进程

在下一个例子中，我们将看到如何修改工作环境。

**但在此之前，让我们先来看看在默认环境中可以找到的信息种类**:

```java
ProcessBuilder processBuilder = new ProcessBuilder();        
Map<String, String> environment = processBuilder.environment();
environment.forEach((key, value) -> System.out.println(key + value));
```

这只是打印出默认情况下提供的每个变量条目:

```java
PATH/usr/bin:/bin:/usr/sbin:/sbin
SHELL/bin/bash
...
```

**现在我们要给我们的`ProcessBuilder`对象添加一个新的环境变量，并运行一个命令来输出它的值:**

```java
environment.put("GREETING", "Hola Mundo");

processBuilder.command("/bin/bash", "-c", "echo $GREETING");
Process process = processBuilder.start();
```

让我们分解这些步骤来理解我们所做的事情:

*   向我们的环境添加一个名为“GREETING”的变量，其值为“Hola Mundo ”,这是一个标准的`Map<String, String>`
*   这一次，我们没有使用构造函数，而是通过`command(String… command)`方法直接设置命令和参数。
*   然后，我们按照前面的例子开始我们的过程。

为了完成该示例，我们验证输出是否包含我们的问候语:

```java
List<String> results = readOutput(process.getInputStream());
assertThat("Results should not be empty", results, is(not(empty())));
assertThat("Results should contain java version: ", results, hasItem(containsString("Hola Mundo")));
```

### 3.3。使用修改后的工作目录启动进程

有时改变工作目录会很有用。在下一个示例中，我们将了解如何做到这一点:

```java
@Test
public void givenProcessBuilder_whenModifyWorkingDir_thenSuccess() 
  throws IOException, InterruptedException {
    ProcessBuilder processBuilder = new ProcessBuilder("/bin/sh", "-c", "ls");

    processBuilder.directory(new File("src"));
    Process process = processBuilder.start();

    List<String> results = readOutput(process.getInputStream());
    assertThat("Results should not be empty", results, is(not(empty())));
    assertThat("Results should contain directory listing: ", results, contains("main", "test"));

    int exitCode = process.waitFor();
    assertEquals("No errors should be detected", 0, exitCode);
}
```

**在上面的例子中，我们使用便利的方法`directory(File directory)`** 将工作目录设置为项目的`src` dir。然后，我们运行一个简单的目录列表命令，检查输出是否包含子目录`main`和`test`。

### 3.4。重定向标准输入和输出

**在现实世界中，我们可能希望将运行过程的结果捕获到一个日志文件中，以供进一步分析**。幸运的是，`ProcessBuilder` API 内置了对这一点的支持，我们将在这个例子中看到。

默认情况下，我们的流程从管道中读取输入。我们可以通过`Process.getOutputStream()` 返回的输出流来访问这个管道。

然而，我们很快就会看到，标准输出可能会被重定向到另一个源，比如使用方法`redirectOutput`的文件。**在这种情况下，`getOutputStream()`** 会返回一个`**ProcessBuilder.NullOutputStream**.`

让我们回到最初的例子，打印出 Java 的版本。但是这次让我们将输出重定向到一个日志文件，而不是标准的输出管道:

```java
ProcessBuilder processBuilder = new ProcessBuilder("java", "-version");

processBuilder.redirectErrorStream(true);
File log = folder.newFile("java-version.log");
processBuilder.redirectOutput(log);

Process process = processBuilder.start();
```

在上面的例子中，**我们创建了一个名为 log 的新临时文件，并告诉我们的`ProcessBuilder`将输出重定向到这个文件目的地**。

在最后一个代码片段中，我们简单地检查了`getInputStream()`确实是`null`,并且我们文件的内容与预期的一样:

```java
assertEquals("If redirected, should be -1 ", -1, process.getInputStream().read());
List<String> lines = Files.lines(log.toPath()).collect(Collectors.toList());
assertThat("Results should contain java version: ", lines, hasItem(containsString("java version")));
```

现在让我们来看看这个例子的一个细微变化。例如，当我们希望追加到一个日志文件中，而不是每次都创建一个新的日志文件时:

```java
File log = tempFolder.newFile("java-version-append.log");
processBuilder.redirectErrorStream(true);
processBuilder.redirectOutput(Redirect.appendTo(log));
```

**同样重要的是，如果出现任何错误，对`redirectErrorStream(true).`的调用，错误输出将被合并到正常流程输出文件中。**

当然，我们可以为标准输出和标准错误输出指定单独的文件:

```java
File outputLog = tempFolder.newFile("standard-output.log");
File errorLog = tempFolder.newFile("error.log");

processBuilder.redirectOutput(Redirect.appendTo(outputLog));
processBuilder.redirectError(Redirect.appendTo(errorLog));
```

### 3.5。继承当前进程的 I/O

在倒数第二个例子中，我们将看到`inheritIO()`方法的运行。**当我们想要将子进程 I/O 重定向到当前进程的标准 I/O 时，我们可以使用这个方法:**

```java
@Test
public void givenProcessBuilder_whenInheritIO_thenSuccess() throws IOException, InterruptedException {
    ProcessBuilder processBuilder = new ProcessBuilder("/bin/sh", "-c", "echo hello");

    processBuilder.inheritIO();
    Process process = processBuilder.start();

    int exitCode = process.waitFor();
    assertEquals("No errors should be detected", 0, exitCode);
}
```

在上面的例子中，通过使用`inheritIO()`方法，我们可以在 IDE 的控制台中看到一个简单命令的输出。

在下一节中，我们将看看 Java 9 中的`ProcessBuilder` API 增加了什么。

## 4。Java 9 新增功能

**Java 9 向`ProcessBuilder` API:** 引入了管道的概念

```java
public static List<Process> startPipeline​(List<ProcessBuilder> builders) 
```

使用`startPipeline`方法，我们可以传递一组`ProcessBuilder`对象。这个静态方法将为每个`ProcessBuilder`启动一个`Process`。因此，创建了一个由标准输出和标准输入流链接的流程管道。

例如，如果我们想运行这样的程序:

```java
find . -name *.java -type f | wc -l
```

我们要做的是为每个独立的命令创建一个流程构建器，并将它们组合成一个管道:

```java
@Test
public void givenProcessBuilder_whenStartingPipeline_thenSuccess()
  throws IOException, InterruptedException {
    List builders = Arrays.asList(
      new ProcessBuilder("find", "src", "-name", "*.java", "-type", "f"), 
      new ProcessBuilder("wc", "-l"));

    List processes = ProcessBuilder.startPipeline(builders);
    Process last = processes.get(processes.size() - 1);

    List output = readOutput(last.getInputStream());
    assertThat("Results should not be empty", output, is(not(empty())));
}
```

**在这个例子中，我们正在搜索`src`目录中的所有 java 文件，并将结果传送到另一个进程中进行计数。**

要了解 Java 9 中对流程 API 的其他改进，请查看我们关于 [Java 9 流程 API 改进](/web/20221228152449/https://www.baeldung.com/java-9-process-api)的精彩文章。

## 5。结论

总而言之，在本教程中，我们已经详细探索了`java.lang.ProcessBuilder` API。

首先，我们从解释 API 可以做什么开始，并总结了最重要的方法。

接下来，我们看了一些实际的例子。最后，我们看了 Java 9 中引入了哪些新的 API。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20221228152449/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-os)