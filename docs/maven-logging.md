# Maven 日志记录选项

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-logging>

## 1.概观

在这个快速教程中，我们将看到如何在 Maven 中配置日志选项。

## 2.命令行

默认情况下，Maven 只记录`info, warning,` 和 `error`日志。此外，对于错误，它不会显示该日志的完整堆栈跟踪。**为了看到完整的堆栈跟踪，我们可以使用`-e `或`–errors `选项**:

```java
$ mvn -e clean compile
// truncated
cannot find symbol
  symbol:   variable name
  location: class Compiled

    at org.apache.maven.lifecycle.internal.MojoExecutor.execute (MojoExecutor.java:213)
    at org.apache.maven.lifecycle.internal.MojoExecutor.execute (MojoExecutor.java:154)
    at org.apache.maven.lifecycle.internal.MojoExecutor.execute (MojoExecutor.java:146)
    ...
```

如上图，现在 Maven 显示了完整的错误报告。**也可以通过`-X `或`–debug `选项**查看调试级日志:

```java
$ mvn -X clean compile
// truncated
OS name: "mac os x", version: "10.15.5", arch: "x86_64", family: "mac"
[DEBUG] Created new class realm maven.api
[DEBUG] Importing foreign packages into class realm maven.api
...
```

当调试打开时，输出非常详细。为了解决这个问题，我们可以通过`-q `或`–quiet `选项要求 Maven 不要记录除错误之外的任何内容:

```java
$ mvn --quiet clean compile
```

此外，我们可以使用`-l `或`–log-file `选项将 Maven 日志重定向到一个文件:

```java
$ mvn --log-file ./mvn.log clean compile
```

所有日志都可以在当前目录下的`mvn.log `文件中找到，而不是标准输出。或者，也可以使用操作系统特性将 Maven 输出重定向到一个文件:

```java
$ mvn clean compile > ./mvn.log
```

## 3.SLF4J 设置

**目前，Maven 正在使用 [SLF4J](/web/20221206034139/https://www.baeldung.com/slf4j-with-log4j2-logback) API 结合 [SLF4J 简单](https://web.archive.org/web/20221206034139/https://www.slf4j.org/apidocs/org/slf4j/impl/SimpleLogger.html)实现**进行日志记录。因此，要配置 SLF4J 简单日志，我们可以编辑`${maven.home}/conf/logging/simplelogger.properties`文件中的属性。

例如，如果我们在该文件中添加以下行:

```java
org.slf4j.simpleLogger.showDateTime=true
org.slf4j.simpleLogger.dateTimeFormat=yyyy-MM-dd HH:mm:ss
```

然后 Maven 将以上面的格式显示日期时间信息。

让我们试试另一个版本:

```java
$ mvn clean compile
2020-07-08 12:08:07 [INFO] Scanning for projects...
```

我们还可以从命令行通过`-D `参数传递这些属性:

```java
$ mvn compile -Dorg.slf4j.simpleLogger.showThreadName=true
[main] [INFO] Scanning for projects...
```

这里我们显示了线程名和其他信息。

除了上面提到的属性，我们还可以用其他的[属性](/web/20221206034139/https://www.baeldung.com/slf4j-with-log4j2-logback)来配置简单的记录器:

*   `org.slf4j.simpleLogger.logFile` 使用日志文件进行记录，而不是标准输出
*   `org.slf4j.simpleLogger.defaultLogLevel` 代表默认日志级别。可以是`trace`、` debug`、` info`、` warn`、` error, `或 `off –` 中的一种，默认值为`info`
*   `org.slf4j.simpleLogger.showLogName` 显示 SLF4j 记录器的名称，如果它是`true`
*   `org.slf4j.simpleLogger.showShortLogName` 如果是`true`则截断长记录器名称

## 4.结论

在这个简短的教程中，我们看到了如何在 Maven 中配置不同的日志和详细选项。