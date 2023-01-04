# Java-R 集成

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-r-integration>

## 1.概观

R 是一种流行的用于统计的编程语言。因为它有各种各样的可用函数和包，所以将 R 代码嵌入到其他语言中并不是不常见的需求。

在本文中，我们将看看将 R 代码集成到 Java 中的一些最常见的方法。

## 2.r 脚本

对于我们的项目，我们将从实现一个非常简单的 R 函数开始，它将一个向量作为输入，并返回其值的平均值。我们将在一个专用文件中对此进行定义:

```
customMean <- function(vector) {
    mean(vector)
}
```

在本教程中，我们将使用一个 Java helper 方法来读取该文件，并将其内容作为`String`返回:

```
String getMeanScriptContent() throws IOException, URISyntaxException {
    URI rScriptUri = RUtils.class.getClassLoader().getResource("script.R").toURI();
    Path inputScript = Paths.get(rScriptUri);
    return Files.lines(inputScript).collect(Collectors.joining());
}
```

现在，让我们看看从 Java 调用这个函数的不同选项。

## 3.RCaller

我们要考虑的第一个库是 [RCaller](https://web.archive.org/web/20220625161849/https://github.com/jbytecode/rcaller) ，它可以通过在本地机器上生成一个专用的 R 进程来执行代码。

由于 RCaller 可从 [Maven Central](https://web.archive.org/web/20220625161849/https://search.maven.org/classic/#artifactdetails%7Ccom.github.jbytecode%7CRCaller%7C3.0%7Cjar) 获得，我们可以将它包含在我们的`pom.xml`:

```
<dependency>
    <groupId>com.github.jbytecode</groupId>
    <artifactId>RCaller</artifactId>
    <version>3.0</version>
</dependency>
```

接下来，让我们编写一个自定义方法，使用我们原来的 R 脚本返回我们的平均值:

```
public double mean(int[] values) throws IOException, URISyntaxException {
    String fileContent = RUtils.getMeanScriptContent();
    RCode code = RCode.create();
    code.addRCode(fileContent);
    code.addIntArray("input", values);
    code.addRCode("result <- customMean(input)");
    RCaller caller = RCaller.create(code, RCallerOptions.create());
    caller.runAndReturnResult("result");
    return caller.getParser().getAsDoubleArray("result")[0];
}
```

在这个方法中，我们主要使用两个对象:

*   `RCode`，它代表我们的代码上下文，包括我们的函数、它的输入和一个调用语句
*   `RCaller`，它让我们运行代码并返回结果

重要的是要注意到 **RCaller 不适合小而频繁的计算**,因为它需要时间来启动 R 进程。这是一个明显的缺点。

另外， **RCaller 只能在 R 安装在本地机器**上的情况下工作。

## 4.仁进

Renjin 是 R 集成领域另一个流行的解决方案。**它被更广泛地采用，而且它还提供企业支持**。

将 Renjin 添加到我们的项目中稍微简单一些，因为我们必须添加 [`Mulesoft`](https://web.archive.org/web/20220625161849/https://repository.mulesoft.org/nexus/content/repositories/public/) 库以及 Maven 依赖项:

```
<repositories>
    <repository>
        <id>mulesoft</id>
        <name>Mulesoft Repository</name>
        <url>https://repository.mulesoft.org/nexus/content/repositories/public/</url>
    </repository>
</repositories>

<dependencies>
    <dependency>
        <groupId>org.renjin</groupId>
        <artifactId>renjin-script-engine</artifactId>
        <version>RELEASE</version>
    </dependency>
</dependencies>
```

让我们再次为 R 函数构建一个 Java 包装器:

```
public double mean(int[] values) throws IOException, URISyntaxException, ScriptException {
    RenjinScriptEngine engine = new RenjinScriptEngine();
    String meanScriptContent = RUtils.getMeanScriptContent();
    engine.put("input", values);
    engine.eval(meanScriptContent);
    DoubleArrayVector result = (DoubleArrayVector) engine.eval("customMean(input)");
    return result.asReal();
}
```

正如我们所看到的，**的概念与 RCaller 非常相似，尽管没有**那么冗长，因为我们可以使用`eval`方法通过名字直接调用函数。

Renjin 的主要优势是它不需要安装 R，因为它使用基于 JVM 的解释器。不过，仁金目前还不能 100%兼容 GNU r

## 5.Rserve

到目前为止，我们已经讨论过的库是在本地运行代码的好选择。但是如果我们想让多个客户端调用我们的 R 脚本呢？这就是 [Rserve](https://web.archive.org/web/20220625161849/https://www.rforge.net/Rserve/index.html) 发挥作用的地方，**让我们通过 TCP 服务器**在远程机器上运行 R 代码。

设置 Rserve 包括安装相关的包，并通过 R 控制台启动服务器加载我们的脚本:

```
> install.packages("Rserve")
...
> library("Rserve")
> Rserve(args = "--RS-source ~/script.R")
Starting Rserve...
```

接下来，我们现在可以像往常一样，通过添加 [Maven 依赖项](https://web.archive.org/web/20220625161849/https://search.maven.org/classic/#artifactdetails%7Corg.rosuda.REngine%7CRserve%7C1.8.1%7Cjar)将 Rserve 包含在我们的项目中:

```
<dependency>
    <groupId>org.rosuda.REngine</groupId>
    <artifactId>Rserve</artifactId>
    <version>1.8.1</version>
</dependency>
```

最后，让我们将 R 脚本包装成一个 Java 方法。这里我们将使用一个带有服务器地址的`RConnection`对象，如果没有提供，默认为 127.0.0.1:6311:

```
public double mean(int[] values) throws REngineException, REXPMismatchException {
    RConnection c = new RConnection();
    c.assign("input", values);
    return c.eval("customMean(input)").asDouble();
}
```

## 6\. FastR

我们要讨论的最后一个库是 [FastR](https://web.archive.org/web/20220625161849/https://github.com/oracle/fastr) 。基于 [GraalVM](https://web.archive.org/web/20220625161849/https://www.graalvm.org/) 的高性能 R 实现。在撰写本文时， **FastR 只在 Linux 和 Darwin x64 系统上可用**。

为了使用它，我们首先需要从官网安装 GraalVM。之后，我们需要使用 Graal 组件更新程序安装 FastR 本身，然后运行它附带的配置脚本:

```
$ bin/gu install R
...
$ languages/R/bin/configure_fastr
```

这一次我们的代码将依赖于 [Polyglot](https://web.archive.org/web/20220625161849/https://www.graalvm.org/reference-manual/polyglot-programming/) ，GraalVM 内部 API 用于在 Java 中嵌入不同的客户语言。由于 Polyglot 是一个通用 API，我们指定了想要运行的代码的语言。同样，我们将使用`c` R 函数将我们的输入转换成一个向量:

```
public double mean(int[] values) {
    Context polyglot = Context.newBuilder().allowAllAccess(true).build();
    String meanScriptContent = RUtils.getMeanScriptContent(); 
    polyglot.eval("R", meanScriptContent);
    Value rBindings = polyglot.getBindings("R");
    Value rInput = rBindings.getMember("c").execute(values);
    return rBindings.getMember("customMean").execute(rInput).asDouble();
}
```

当遵循这种方法时，请记住它使我们的代码与 JVM 紧密耦合。要了解更多关于 GraalVM 的信息，请查看我们关于 [Graal Java JIT 编译器](/web/20220625161849/https://www.baeldung.com/graal-java-jit-compiler)的文章。

## 7.结论

在本文中，我们介绍了一些在 Java 中集成 R 的最流行的技术。总而言之:

*   RCaller 更容易集成，因为它在 Maven Central 上可用
*   Renjin 提供企业支持，并且不要求 R 安装在本地机器上，但是它不是 100%兼容 GNU R
*   Rserve 可用于在远程服务器上执行 R 代码
*   FastR 允许与 Java 无缝集成，但是使我们的代码依赖于 VM，并且不是对每个操作系统都可用

和往常一样，本教程中使用的所有代码都可以在 GitHub 上获得[。](https://web.archive.org/web/20220625161849/https://github.com/eugenp/tutorials/tree/master/libraries-6)