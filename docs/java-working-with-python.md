# 如何从 Java 调用 Python

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-working-with-python>

## 1。概述

Python 是一种越来越受欢迎的编程语言，尤其是在科学界，因为它有丰富的数值和统计软件包。因此，从我们的 Java 应用程序中调用 Python 代码的需求并不少见。

在本教程中，我们将看看从 Java 调用 Python 代码的一些最常见的方法。

## 2。一个简单的 Python 脚本

在整个教程中，我们将使用一个非常简单的 Python 脚本，我们将在一个名为`hello.py`的专用文件中定义它:

```java
print("Hello Baeldung Readers!!")
```

假设我们有一个工作的 Python 安装，当我们运行我们的脚本时，我们应该看到打印的消息:

```java
$ python hello.py 
Hello Baeldung Readers!!
```

## 3。核心 Java

在这一节中，我们将看看两个不同的选项，我们可以用它们来调用使用核心 Java 的 Python 脚本。

### 3.1。使用`ProcessBuilder`

让我们首先来看看如何使用 [`ProcessBuilder` API](/web/20220812012137/https://www.baeldung.com/java-lang-processbuilder-api) 创建一个本地操作系统进程来启动`python`并执行我们的简单脚本:

```java
@Test
public void givenPythonScript_whenPythonProcessInvoked_thenSuccess() throws Exception {
    ProcessBuilder processBuilder = new ProcessBuilder("python", resolvePythonScriptPath("hello.py"));
    processBuilder.redirectErrorStream(true);

    Process process = processBuilder.start();
    List<String> results = readProcessOutput(process.getInputStream());

    assertThat("Results should not be empty", results, is(not(empty())));
    assertThat("Results should contain output of script: ", results, hasItem(
      containsString("Hello Baeldung Readers!!")));

    int exitCode = process.waitFor();
    assertEquals("No errors should be detected", 0, exitCode);
}
```

在第一个例子中，我们运行带有一个参数的`python`命令，该参数是我们的`hello.py`脚本的绝对路径。我们可以在我们的`test/resources`文件夹中找到它。

总而言之，我们创建了我们的`ProcessBuilder`对象，将命令和参数值传递给构造函数。**同样重要的是，如果出现任何错误，对`redirectErrorStream(true).`的调用，错误输出将与标准输出合并。**

这很有用，因为这意味着当我们在`Process`对象上调用`getInputStream()`方法时，我们可以从相应的输出中读取任何错误消息。如果我们不将这个属性设置为`true`，那么我们将需要使用`getInputStream()`和`getErrorStream()`方法从两个独立的流中读取输出。

现在，我们开始使用`start()`方法获取一个`Process`对象。然后，我们读取流程输出，并验证内容是否符合我们的预期。

**如前所述，我们假设`python`命令可以通过`PATH`变量获得。**

### 3.2。使用 JSR-223 脚本引擎

最初在 Java 6 中引入的 JSR-223 ，定义了一组提供基本脚本功能的脚本 API。这些方法提供了在 Java 和脚本语言之间执行脚本和共享值的机制。该标准的主要目标是试图为 Java 中不同脚本语言的互操作带来某种统一性。

当然，我们可以为任何动态语言使用可插拔脚本引擎架构，只要它有一个 [JVM](/web/20220812012137/https://www.baeldung.com/jvm-languages) 实现。 **[Jython](https://web.archive.org/web/20220812012137/https://www.jython.org/) 是运行在 JVM 上的 Python 的 Java 平台实现。**

假设我们在`CLASSPATH`上有 Jython，框架应该会自动发现我们有使用这个脚本引擎的可能性，并使我们能够直接请求 Python 脚本引擎。

由于 Jython 可以从 [Maven Central](https://web.archive.org/web/20220812012137/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22jython%22) 获得，我们可以将它包含在我们的`pom.xml`中:

```java
<dependency>
    <groupId>org.python</groupId>
    <artifactId>jython</artifactId>
    <version>2.7.2</version>
</dependency>
```

同样，也可以[下载](https://web.archive.org/web/20220812012137/https://www.jython.org/download)直接安装。

让我们列出所有可用的脚本引擎:

```java
ScriptEngineManagerUtils.listEngines();
```

如果我们有可能使用 Jython，我们应该会看到显示的适当的脚本引擎:

```java
...
Engine name: jython
Version: 2.7.2
Language: python
Short Names:
python
jython 
```

既然我们知道可以使用 Jython 脚本引擎，那么让我们来看看如何调用我们的`hello.py`脚本:

```java
@Test
public void givenPythonScriptEngineIsAvailable_whenScriptInvoked_thenOutputDisplayed() throws Exception {
    StringWriter writer = new StringWriter();
    ScriptContext context = new SimpleScriptContext();
    context.setWriter(writer);

    ScriptEngineManager manager = new ScriptEngineManager();
    ScriptEngine engine = manager.getEngineByName("python");
    engine.eval(new FileReader(resolvePythonScriptPath("hello.py")), context);
    assertEquals("Should contain script output: ", "Hello Baeldung Readers!!", writer.toString().trim());
}
```

正如我们所看到的，使用这个 API 非常简单。首先，我们开始设置一个包含一个`StringWriter`的`ScriptContext`。这将用于存储我们想要调用的脚本的输出。

**然后我们使用`ScriptEngineManager`类的`getEngineByName`方法来查找并为给定的简称**创建一个`ScriptEngine`。在我们的例子中，我们可以通过`python`或`jython`，这是与这个引擎相关的两个简称。

和前面一样，最后一步是从我们的脚本中获取输出，并检查它是否符合我们的预期。

## 4。Jython

继续 Jython，我们也有可能将 Python 代码直接嵌入到我们的 Java 代码中。我们可以使用`PythonInterpretor`类来做到这一点:

```java
@Test
public void givenPythonInterpreter_whenPrintExecuted_thenOutputDisplayed() {
    try (PythonInterpreter pyInterp = new PythonInterpreter()) {
        StringWriter output = new StringWriter();
        pyInterp.setOut(output);

        pyInterp.exec("print('Hello Baeldung Readers!!')");
        assertEquals("Should contain script output: ", "Hello Baeldung Readers!!", output.toString()
          .trim());
    }
}
```

**使用`PythonInterpreter`类允许我们通过`exec`方法**执行一串 Python 源代码。像以前一样，我们使用一个`StringWriter`来捕获这次执行的输出。

现在让我们看一个将两个数字相加的例子:

```java
@Test
public void givenPythonInterpreter_whenNumbersAdded_thenOutputDisplayed() {
    try (PythonInterpreter pyInterp = new PythonInterpreter()) {
        pyInterp.exec("x = 10+10");
        PyObject x = pyInterp.get("x");
        assertEquals("x: ", 20, x.asInt());
    }
}
```

在这个例子中，我们看到了如何使用`get`方法来访问变量的值。

在我们最后的 Jython 示例中，我们将看到当错误发生时会发生什么:

```java
try (PythonInterpreter pyInterp = new PythonInterpreter()) {
    pyInterp.exec("import syds");
}
```

当我们运行这段代码时，会抛出一个`PyException`,我们会看到相同的错误，就好像我们在使用原生 Python 一样:

```java
Traceback (most recent call last):
  File "<string>", line 1, in <module>
ImportError: No module named syds
```

有几点我们应该注意:

*   作为`PythonIntepreter`的实现`AutoCloseable`，在处理这个类时使用 [`try-with-resources`](/web/20220812012137/https://www.baeldung.com/java-try-with-resources) 是个好习惯
*   `PythonInterpreter`类名并不意味着我们的 Python 代码被解释了。Jython 中的 Python 程序是由 JVM 运行的，因此在执行之前要编译成 Java 字节码
*   **虽然 Jython 是 Java 的 Python 实现，但它可能不包含与原生 Python 相同的所有子包**

## 5。Apache Commons Exec

我们可以考虑使用的另一个第三方库是 [Apache Common Exec](https://web.archive.org/web/20220812012137/https://commons.apache.org/proper/commons-exec/index.html) ，它试图克服 [Java Process API](/web/20220812012137/https://www.baeldung.com/java-process-api) 的一些缺点。

`commons-exec`神器可以从 [Maven Central](https://web.archive.org/web/20220812012137/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22commons-exec%22) 获得:

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-exec</artifactId>
    <version>1.3</version>
</dependency>
```

现在让我们看看如何使用这个库:

```java
@Test
public void givenPythonScript_whenPythonProcessExecuted_thenSuccess() 
  throws ExecuteException, IOException {
    String line = "python " + resolvePythonScriptPath("hello.py");
    CommandLine cmdLine = CommandLine.parse(line);

    ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
    PumpStreamHandler streamHandler = new PumpStreamHandler(outputStream);

    DefaultExecutor executor = new DefaultExecutor();
    executor.setStreamHandler(streamHandler);

    int exitCode = executor.execute(cmdLine);
    assertEquals("No errors should be detected", 0, exitCode);
    assertEquals("Should contain script output: ", "Hello Baeldung Readers!!", outputStream.toString()
      .trim());
}
```

这个例子与我们使用`ProcessBuilder`的第一个例子没有太大的不同。我们为给定的命令创建一个`CommandLine`对象。接下来，我们设置一个流处理程序，用于在执行命令之前捕获流程的输出。

总而言之，本库背后的主要理念是提供一个流程执行包，旨在通过一致的 API 支持广泛的操作系统。

## 6。利用 HTTP 实现互操作性

让我们暂时后退一步，不要试图直接调用 Python，而是考虑使用 HTTP 等成熟的协议作为两种不同语言之间的抽象层。

**事实上，Python 附带了一个简单的内置 HTTP 服务器，我们可以使用它通过 HTTP** 共享内容或文件:

```java
python -m http.server 9000
```

如果我们现在转到 [`http://localhost:9000`](https://web.archive.org/web/20220812012137/http://localhost:9000/) ，我们将看到列出了我们启动前一个命令的目录的内容。

我们可以考虑使用其他一些流行的框架来创建更健壮的基于 Python 的 web 服务或应用程序，它们是 [Flask](https://web.archive.org/web/20220812012137/https://palletsprojects.com/p/flask/) 和 [Django](https://web.archive.org/web/20220812012137/https://www.djangoproject.com/) 。

一旦我们有了可以访问的端点，我们就可以使用几个 [Java HTTP](/web/20220812012137/https://www.baeldung.com/java-http-request) 库中的任何一个来调用我们的 Python web 服务/应用程序实现。

## 7 .**。结论**

在本教程中，我们学习了一些最流行的从 Java 调用 Python 代码的技术。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20220812012137/https://github.com/eugenp/tutorials/tree/master/language-interop)