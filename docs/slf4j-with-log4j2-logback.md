# SLF4J 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/slf4j-with-log4j2-logback>

## 1。概述

Java 的简单日志门面(简写为 SLF4J)充当不同日志框架的[门面](https://web.archive.org/web/20220927045338/https://en.wikipedia.org/wiki/Facade_pattern)(例如 [java.util.logging、logback、Log4j](/web/20220927045338/https://www.baeldung.com/java-logging-intro) )。它提供了一个通用的 API，使得日志独立于实际的实现。

这允许不同的日志框架共存。它有助于从一个框架迁移到另一个框架。最后，除了标准化的 API，它还提供了一些“语法糖”

本教程将讨论将 SLF4J 与 Log4j、Logback、Log4j 2 和 Jakarta Commons 日志集成所需的依赖性和配置。

有关这些实现的更多信息，请查看我们的文章[Java 日志介绍](/web/20220927045338/https://www.baeldung.com/java-logging-intro)。

## 延伸阅读:

## [回退指南](/web/20220927045338/https://www.baeldung.com/logback)

Explore the fundamentals of using Logback in your application.[Read more](/web/20220927045338/https://www.baeldung.com/logback) →

## [Java 日志简介](/web/20220927045338/https://www.baeldung.com/java-logging-intro)

A quick intro to logging in Java - the libraries, the configuration details as well as pros and cons of each solution.[Read more](/web/20220927045338/https://www.baeldung.com/java-logging-intro) →

## [log4j 2 简介——附加器、布局和过滤器](/web/20220927045338/https://www.baeldung.com/log4j2-appenders-layouts-filters)

This article, using an example rich approach, introduces Log4J 2 Appender, Layout and Filter concepts[Read more](/web/20220927045338/https://www.baeldung.com/log4j2-appenders-layouts-filters) →

## 2。**Log4j 2 设置**

为了在 Log4j 2 中使用 SLF4J，我们将以下库添加到`pom.xml`:

```
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-api</artifactId>
    <version>2.7</version>
</dependency>
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.7</version>
</dependency>
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-slf4j-impl</artifactId>
    <version>2.7</version>
</dependency>
```

最新版本可以在这里找到: [log4j-api](https://web.archive.org/web/20220927045338/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.logging.log4j%22%20AND%20a%3A%22log4j-api%22) ， [log4j-core](https://web.archive.org/web/20220927045338/https://mvnrepository.com/artifact/org.apache.logging.log4j/log4j-core) ， [log4j-slf4j-impl](https://web.archive.org/web/20220927045338/https://mvnrepository.com/artifact/org.apache.logging.log4j/log4j-slf4j-impl) 。

实际的日志配置遵循本地 Log4j 2 配置。

让我们看看如何创建`Logger`实例:

```
public class SLF4JExample {

    private static Logger logger = LoggerFactory.getLogger(SLF4JExample.class);

    public static void main(String[] args) {
        logger.debug("Debug log message");
        logger.info("Info log message");
        logger.error("Error log message");
    }
}
```

注意`Logger` 和 `LoggerFactory` 属于`org.slf4j`包。

使用这种配置运行的项目示例可在[这里](https://web.archive.org/web/20220927045338/https://github.com/eugenp/tutorials/tree/master/logging-modules/log4j)找到。

## 3。 **回退设置**

我们不需要将 SLF4J 添加到我们的类路径中来与 Logback 一起使用，因为 Logback 已经在使用 SLF4J。这是参考实现。

因此，我们只需要包含回退库:

```
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.2.6</version>
</dependency>
```

最新版本可以在这里找到: [logback-classic](https://web.archive.org/web/20220927045338/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22ch.qos.logback%22%20AND%20a%3A%22logback-classic%22) 。

该配置是特定于回退的，但是可以与 SLF4J 无缝协作。有了适当的依赖项和配置，我们可以使用前面章节中的相同代码来处理日志记录。

## 4 **。****Log4j 设置**

在前面的章节中，我们介绍了一个用例，其中 SLF4J“位于”特定日志实现之上。像这样使用，它完全抽象了底层框架。

有些情况下，我们无法替换现有的日志记录解决方案，例如，由于第三方的要求。但是这并没有将项目局限于已经使用的框架。

我们可以将 SLF4J 配置为一个网桥，并将呼叫重定向到现有的 it 框架。

让我们添加必要的依赖项来为 Log4j 创建一个桥:

```
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>log4j-over-slf4j</artifactId>
    <version>1.7.30</version>
</dependency>
```

有了依赖关系(在 [log4j-over-slf4j](https://web.archive.org/web/20220927045338/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.slf4j%22%20AND%20a%3A%22log4j-over-slf4j%22) 检查最新的)，所有对 log4j 的调用都将被重定向到 slf4j。

看一下官方文档，了解更多关于桥接现有框架的信息。

正如其他框架一样，Log4j 可以作为底层实现。

让我们添加必要的依赖项:

```
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
    <version>1.7.30</version>
</dependency>
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
```

下面是 slf4j-log4j12 和 [log4j](https://web.archive.org/web/20220927045338/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22log4j%22%20AND%20a%3A%22log4j%22) 的最新版本。以这种方式配置的示例项目在[这里](https://web.archive.org/web/20220927045338/https://github.com/eugenp/tutorials/tree/master/testing-modules/rest-assured)可用。

## 5。 **JCL 桥设置**

在前面的小节中，我们展示了如何使用相同的代码库来支持使用不同实现的日志记录。虽然这是 SLF4J 的主要承诺和优势，但它也是 JCL (Jakarta Commons Logging 或 Apache Commons Logging)背后的目标。

JCL 旨在成为一个类似于 SLF4J 的框架。主要区别在于，JCL 通过一个类加载系统在运行时解析底层实现。在有自定义类装入器的情况下，这种方法似乎有问题。

SLF4J 在编译时解析它的绑定。它被认为是简单而强大的。

幸运的是，两个框架可以在桥接模式下协同工作:

```
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>jcl-over-slf4j</artifactId>
    <version>1.7.30</version>
</dependency>
```

最新的依赖版本可以在这里找到: [jcl-over-slf4j](https://web.archive.org/web/20220927045338/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.slf4j%22%20AND%20a%3A%22jcl-over-slf4j%22) 。

与其他情况一样，相同的代码库也能很好地运行。运行此设置的完整项目示例可在[此处](https://web.archive.org/web/20220927045338/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java)获得。

## 6。进一步 **SLF4J 特色**

SLF4J 提供了额外的特性，可以使日志记录更有效，代码更可读。

例如，SLF4J 为处理参数提供了一个非常有用的界面:

```
String variable = "Hello John";
logger.debug("Printing variable value: {}", variable);
```

下面是做同样事情的 Log4j 代码:

```
String variable = "Hello John";
logger.debug("Printing variable value: " + variable);
```

正如我们所见，无论`debug`电平是否使能，Log4j 都会连接`Strings`。在高负载应用程序中，这可能会导致性能问题。另一方面，只有当`debug`电平被使能时，SLF4J 才会连接`Strings`。

为了对 Log4J 做同样的事情，我们需要添加一个额外的`if`块，它将检查*调试*级别是否启用:

```
String variable = "Hello John";
if (logger.isDebugEnabled()) {
    logger.debug("Printing variable value: " + variable);
}
```

SLF4J 标准化了日志记录级别，这些级别对于特定的实现是不同的。它取消了`FATAL`日志级别(在 Log4j 中引入),前提是在日志框架中我们不应该决定何时终止应用程序。

使用的测井级别有`ERROR`、`WARN`、`INFO`、`DEBUG` 和`TRACE`。在我们的[Java 日志介绍](/web/20220927045338/https://www.baeldung.com/java-logging-intro)中阅读更多关于使用它们的内容。

## 7 .**。结论**

SLF4J 有助于日志框架之间的静默切换。它很简单，但是很灵活，可以提高可读性和性能。

像往常一样，代码可以在 GitHub 上找到。此外，我们引用了另外两个项目，它们致力于不同的文章，但是包含了所讨论的日志配置，可以在[这里](https://web.archive.org/web/20220927045338/https://github.com/eugenp/tutorials/tree/master/feign)和[这里](https://web.archive.org/web/20220927045338/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java)找到。