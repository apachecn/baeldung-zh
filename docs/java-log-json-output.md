# 获取 JSON 中的日志输出

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-log-json-output>

## 1。简介

如今，大多数 Java 日志库都为格式化日志提供了不同的布局选项，以精确满足每个项目的需求。

在这篇简短的文章中，我们希望将日志条目格式化并输出为 JSON。我们将看到如何为两个最广泛使用的日志库:`Log4j2`和`Logback`这样做。

两者都在内部使用`Jackson`来表示 JSON 格式的日志。

关于这些库的介绍，请看一下我们的[Java 日志文章](/web/20221126232502/https://www.baeldung.com/java-logging-intro)。

## 2。Log4j2

Log4j2 是最流行的 Java 日志库 Log4j 的直接继承者。

由于它是 Java 项目的新标准，我们将展示如何将其配置为输出 JSON。

### 2.1。依赖性

首先，我们必须在我们的`pom.` xml 文件中包含以下依赖项:

```java
<dependencies>
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-core</artifactId>
        <version>2.10.0</version>
    </dependency>

    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.13.0</version>
    </dependency>

</dependencies>
```

之前依赖的最新版本可以在 Maven Central 上找到: [log4j-api](https://web.archive.org/web/20221126232502/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.logging.log4j%22%20AND%20a%3A%22log4j-api%22) 、 [log4j-core](https://web.archive.org/web/20221126232502/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.logging.log4j%22%20AND%20a%3A%22log4j-core%22) 、 [jackson-databind。](https://web.archive.org/web/20221126232502/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.fasterxml.jackson.core%22%20AND%20a%3A%22jackson-databind%22)

### 2.2。配置

然后，在我们的`log4j2.` xml 文件中，我们可以创建一个使用`JsonLayout` 的新`Appender`和一个使用这个 `Appender`的新`Logger`:

```java
<Appenders>
    <Console name="ConsoleJSONAppender" target="SYSTEM_OUT">
        <JsonLayout complete="false" compact="false">
            <KeyValuePair key="myCustomField" value="myCustomValue" />
        </JsonLayout>
    </Console>
</Appenders>

<Logger name="CONSOLE_JSON_APPENDER" level="TRACE" additivity="false">
    <AppenderRef ref="ConsoleJSONAppender" />
</Logger>
```

正如我们在示例配置中看到的，可以使用`KeyValuePair`将我们自己的值添加到日志中，这甚至支持对日志上下文的观察。

将`compact`参数设置为`false`会增加输出的大小，但也会使其更易于阅读。

### 2.3。使用 Log4j2

在我们的代码中，我们现在可以实例化新的 JSON 记录器，并创建新的调试级别跟踪:

```java
Logger logger = LogManager.getLogger("CONSOLE_JSON_APPENDER");
logger.debug("Debug message"); 
```

前面代码的调试输出消息应该是:

```java
{
  "timeMillis" : 1513290111664,
  "thread" : "main",
  "level" : "DEBUG",
  "loggerName" : "CONSOLE_JSON_APPENDER",
  "message" : "My debug message",
  "endOfBatch" : false,
  "loggerFqcn" : "org.apache.logging.log4j.spi.AbstractLogger",
  "threadId" : 1,
  "threadPriority" : 5,
  "myCustomField" : "myCustomValue"
}
```

## 3。回退

[Logback](/web/20221126232502/https://www.baeldung.com/custom-logback-appender) 可以认为是 Log4J 的另一个继承者。它是由相同的开发者编写的，并声称比它的前身更有效和更快。

因此，让我们看看如何配置它以获得 JSON 格式的日志输出。

### 3.1。依赖性

让我们在`pom.xml`中包含以下依赖项:

```java
<dependencies>
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.2.6</version>
    </dependency>

    <dependency>
        <groupId>ch.qos.logback.contrib</groupId>
        <artifactId>logback-json-classic</artifactId>
        <version>0.1.5</version>
    </dependency>

    <dependency>
        <groupId>ch.qos.logback.contrib</groupId>
        <artifactId>logback-jackson</artifactId>
        <version>0.1.5</version>
    </dependency>

    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.9.3</version>
    </dependency>
</dependencies>
```

我们可以在这里查看这些依赖项的最新版本: [logback-classic](https://web.archive.org/web/20221126232502/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22ch.qos.logback%22%20AND%20a%3A%22logback-classic%22) ， [logback-json-classic](https://web.archive.org/web/20221126232502/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22ch.qos.logback.contrib%22%20AND%20a%3A%22logback-json-classic%22) ， [logback-jackson](https://web.archive.org/web/20221126232502/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22ch.qos.logback.contrib%22%20AND%20a%3A%22logback-jackson%22) ， [jackson-databind](https://web.archive.org/web/20221126232502/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.fasterxml.jackson.core%22%20AND%20a%3A%22jackson-databind%22)

### 3.2。配置

首先，我们在使用了`JsonLayout`和`JacksonJsonFormatter.`的`logback.xml`中创建新的`appender`

之后，我们可以创建一个新的使用这个`appender`的`logger`:

```java
<appender name="json" class="ch.qos.logback.core.ConsoleAppender">
    <layout class="ch.qos.logback.contrib.json.classic.JsonLayout">
        <jsonFormatter
            class="ch.qos.logback.contrib.jackson.JacksonJsonFormatter">
            <prettyPrint>true</prettyPrint>
        </jsonFormatter>
        <timestampFormat>yyyy-MM-dd' 'HH:mm:ss.SSS</timestampFormat>
    </layout>
</appender>

<logger name="jsonLogger" level="TRACE">
    <appender-ref ref="json" />
</logger>
```

正如我们看到的，参数`prettyPrint`被启用来获得人类可读的 JSON。

### 3.3。使用回退

让我们在代码中实例化记录器，并记录一条调试消息:

```java
Logger logger = LoggerFactory.getLogger("jsonLogger");
logger.debug("Debug message"); 
```

这样，我们将获得以下输出:

```java
{
  "timestamp" : "2017-12-14 23:36:22.305",
  "level" : "DEBUG",
  "thread" : "main",
  "logger" : "jsonLogger",
  "message" : "Debug log message",
  "context" : "default"
}
```

## 4。结论

我们在这里已经看到了如何轻松地配置 Log4j2 和 Logback，使之具有 JSON 输出格式。我们已经将所有复杂的解析委托给日志库，所以我们不需要修改任何现有的日志调用。

一如既往，这篇文章的代码可以在 GitHub [这里](https://web.archive.org/web/20221126232502/https://github.com/eugenp/tutorials/tree/master/logging-modules/log4j2)和[这里](https://web.archive.org/web/20221126232502/https://github.com/eugenp/tutorials/tree/master/logging-modules/logback)找到。