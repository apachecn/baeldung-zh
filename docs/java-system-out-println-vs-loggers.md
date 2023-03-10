# System.out.println vs Loggers

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-system-out-println-vs-loggers>

## 1.为什么是伐木工？

在编写程序或开发企业生产应用时，使用`System.out.println`似乎是最简单、最容易的选择。不需要向类路径添加额外的库，也不需要进行额外的配置。

但是使用`System.out.println`有几个缺点，影响了它在许多情况下的可用性。在本教程中，我们将讨论**为什么以及何时我们想要使用一个日志记录器而不是普通的`System.out`和`System.err`** 。我们还将展示一些使用 Log4J2 日志框架的快速示例。

## 2.设置

在我们开始之前，让我们看看 Maven 的依赖性和所需的配置。

### 2.1.Maven 依赖性

让我们从将 Log4J2 依赖项添加到我们的`pom.xml`开始:

```java
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-api</artifactId>
    <version>2.12.1</version>
</dependency>
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.12.1</version>
</dependency>
```

我们可以在 Maven Central 上找到最新版本的[`log4j-api`](https://web.archive.org/web/20221206080359/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.logging.log4j%22%20AND%20a%3A%22log4j-api%22)`[log4j-core](https://web.archive.org/web/20221206080359/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.logging.log4j%22%20AND%20a%3A%22log4j-core%22)`。

### 2.2.Log4J2 配置

使用`System.out`不需要任何额外的配置。然而，要使用 Log4J2，我们需要一个`log4j.xml`配置文件:

```java
<Configuration status="debug" name="baeldung" packages="">
    <Appenders>
        <Console name="stdout" target="SYSTEM_OUT">
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss} %p %m%n"/>
        </Console>
    </Appenders>
    <Root level="error">
        <AppenderRef ref="STDOUT"/>
    </Root>
</Configuration>
```

几乎所有的 logger 框架都需要某种级别的配置，或者通过编程方式或者通过外部配置文件，比如这里显示的 XML 文件。

## 3.分离日志输出

### 3.1。`System.out`和`System.err`

当我们将应用程序部署到像 Tomcat 这样的服务器上时，服务器使用自己的日志记录器。如果我们使用`System.out`，日志在`catalina.out`结束。如果日志放在一个单独的文件中，调试我们的应用程序会容易得多。对于 Log4j2，我们需要在配置中包含一个文件附加器，以便将应用程序日志保存在一个单独的文件中。

此外，使用`System.out.println`，无法控制或过滤要打印的日志。分离日志的唯一可能方式是使用`System.out.println `作为信息日志，使用`System.err.println `作为错误日志:

```java
System.out.println("This is an informational message");
System.err.println("This is an error message");
```

### 3.2。Log4J2 记录级别

在调试或开发环境中，我们希望看到应用程序正在打印的所有信息。但是在实时企业应用程序中，更多的日志意味着延迟的增加。**log4j 2 这样的日志框架提供了多种日志级别的控制:**

*   致命的
*   错误
*   警告
*   信息
*   调试
*   找到；查出
*   全部

使用这些级别，**我们可以轻松过滤何时何地打印什么信息**:

```java
logger.trace("Trace log message");
logger.debug("Debug log message");
logger.info("Info log message");
logger.error("Error log message");
logger.warn("Warn log message");
logger.fatal("Fatal log message");
```

我们也可以单独配置每个源代码包的级别。关于日志级别配置的更多细节，请参考我们的 [Java 日志文章。](/web/20221206080359/https://www.baeldung.com/java-logging-intro)

## 4.将日志写入文件

### 4.1。重新布线`System.out`和`System.err`和

可以使用`System.setOut()` 方法将`System.out.println`发送到一个文件:

```java
PrintStream outStream = new PrintStream(new File("outFile.txt"));
System.setOut(outStream);
System.out.println("This is a baeldung article");
```

在`System.err`的情况下:

```java
PrintStream errStream = new PrintStream(new File("errFile.txt"));
System.setErr(errStream);
System.err.println("This is a baeldung article error");
```

**当使用`System.out`或`System.err`将输出重定向到一个文件时，我们不能控制文件大小**，因此文件在应用程序运行期间保持增长。

随着文件变大，打开或分析这些更大的日志可能会变得困难。

### 4.2。用 Log4J2 记录文件

Log4J2 提供了一种机制，可以系统地将日志写入文件，还可以基于某些策略滚动文件。例如，我们可以**根据日期/时间模式**配置要转存的文件:

```java
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="INFO">
    <Appenders>
        <File name="fout" fileName="log4j/target/baeldung-log4j2.log"
          immediateFlush="false" append="false">
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss} %p %m%n"/>
        </File>
    <Loggers>
        <AsyncRoot level="DEBUG">
            <AppenderRef ref="stdout"/>
            <AppenderRef ref="fout"/>
        </AsyncRoot>
    </Loggers>
</Configuration>
```

或者，一旦文件达到给定的阈值，我们可以**根据文件大小滚动文件:**

```java
...
<RollingFile name="roll-by-size"
  fileName="target/log4j2/roll-by-size/app.log" filePattern="target/log4j2/roll-by-size/app.%i.log.gz"
  ignoreExceptions="false">
    <PatternLayout>
        <Pattern>%d{yyyy-MM-dd HH:mm:ss} %p %m%n</Pattern>
    </PatternLayout>
    <Policies>
        <OnStartupTriggeringPolicy/>
        <SizeBasedTriggeringPolicy size="5 KB"/>
    </Policies>
</RollingFile>
```

## 5.记录到外部系统

正如我们在上一节中看到的，日志框架允许将日志写到一个文件中。类似地，它们也提供了**附加器来将日志发送到其他系统和应用程序**。这使得使用 Log4J appenders 而不是使用`System.out.println.` 将日志发送到 Kafka 流或 Elasticsearch 数据库成为可能

请参考我们的 [Log4j appender 文章](/web/20221206080359/https://www.baeldung.com/log4j2-appenders-layouts-filters)了解更多关于如何使用这种 appender 的细节。

## 6.自定义日志输出

通过使用记录器，我们可以定制与实际消息一起打印的信息。我们可以打印的信息包括包名、日志级别、行号、时间戳、方法名等。

虽然这在`System.out.println,` 中是可能的，但它需要大量的手工工作，而日志框架提供了开箱即用的功能。对于记录器，**我们可以简单地在记录器配置中定义一个模式**:

```java
<Console name="ConsoleAppender" target="SYSTEM_OUT">
    <PatternLayout pattern="%style{%date{DEFAULT}}{yellow}
      %highlight{%-5level}{FATAL=bg_red, ERROR=red, WARN=yellow, INFO=green} %message"/>
</Console> 
```

如果我们考虑将 Log4J2 用于我们的 logger 框架，有几种模式可供我们选择或定制。参考[官方 Log4J2 文档](https://web.archive.org/web/20221206080359/https://logging.apache.org/log4j/2.x/)了解更多信息。

## 7.通过记录异常输出来避免`printStackTrace()`

当我们在代码中处理异常时，我们经常需要了解在运行时实际发生了什么异常。为此有两个常见的选项:`printStackTrace()`或使用日志程序调用。

使用`printStackTrace()`打印异常细节的异常处理非常常见:

```java
try {
    // some code
} catch (Exception e) {
    e.printStackTrace();
}
```

这里的问题是 **`printStackTrace()`将其信息打印给`System.err`** ，我们已经说过要避免这种情况。

相反，我们可以使用日志框架记录异常，然后，我们将能够轻松地检索日志:

```java
try {
    // some code
} catch (Exception e) {
    logger.log("Context message", e);
}
```

## 8.结论

这篇文章解释了为什么要使用日志框架以及为什么不仅仅依靠`System.out.println`来获取应用程序日志的各种原因。虽然对小型测试程序使用`System.out.println` 是合理的，但是我们不希望将其作为企业生产应用程序的主要日志来源。

和往常一样，文章中的代码示例可以在 GitHub 的[上找到。](https://web.archive.org/web/20221206080359/https://github.com/eugenp/tutorials/tree/master/logging-modules/log4j2)