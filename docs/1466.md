# log4j 2–记录文件和控制台

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-log4j2-file-and-console>

## 1.概观

在本教程中，我们将探索如何使用 [Apache Log4j2 库](/web/20221006181921/https://www.baeldung.com/java-logging-intro#Log4j)将消息记录到文件和控制台。

这在非 prod 环境中非常有用，在这种环境中，我们可能希望在控制台中看到调试消息，并且我们可能希望将更高级别的日志保存到一个文件中，以供以后分析。

## 2.项目设置

让我们从创建一个 Java 项目开始。我们将添加 log4j2 依赖项，并了解如何配置和使用日志记录器。

### 2.1.Log4j2 依赖性

让我们将 log4j2 依赖项添加到我们的项目中。我们将需要 [Apache Log4J 核心](https://web.archive.org/web/20221006181921/https://mvnrepository.com/artifact/org.apache.logging.log4j/log4j-core)和 [Apache Log4J API](https://web.archive.org/web/20221006181921/https://mvnrepository.com/artifact/org.apache.logging.log4j/log4j-api) 依赖项:

```
<dependencies>
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-core</artifactId>
        <version>2.18.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-api</artifactId>
        <version>2.18.0</version>
    </dependency>
</dependencies>
```

### 2.2.应用程序类别

现在让我们使用 log4j2 库向我们的应用程序添加一些日志记录:

```
public class Log4j2ConsoleAndFile {

    private static final Logger logger = LogManager.getLogger(Log4j2ConsoleAndFile.class);

    public static void main(String[] args) {
        logger.info("Hello World!");
        logger.debug("Hello World!");
    }
}
```

## 3.Log4j2 配置

为了自动配置记录器，**我们需要在类路径上有一个配置文件。它可以是 JSON、XML、YAML 或属性格式。该文件应该命名为 log4j2。**对于我们的例子，让我们使用一个名为`log4j2.properties`的配置文件。

### 3.1.记录到控制台

为了记录到任何目的地，我们首先需要定义一个记录到控制台的 [appender](/web/20221006181921/https://www.baeldung.com/log4j2-appenders-layouts-filters) 。让我们来看看实现这一点的配置:

```
appender.console.type = Console
appender.console.name = STDOUT
appender.console.layout.type = PatternLayout
appender.console.layout.pattern = [%-5level] %d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %c{1} - %msg%n
```

让我们了解一下该配置的每个组件:

*   `appender.console.type`–在这里，我们指定**将用于记录的附加器的类型。**类型`Console `指定 appender 将只写控制台。我们需要注意的是，键名中的`console`只是一个约定，并不是强制性的。
*   `appender.console.name – `我们可以给**起一个惟一的名字，以后我们可以用它来指代这个 appender。**
*   `appender.console.layout.type`–这决定了**用于格式化日志消息的布局类的名称。**
*   `appender.console.layout.pattern`–这是将用于格式化日志消息的**模式。**

为了启用控制台记录器，我们需要将控制台附加器添加到根记录器中。我们可以使用上面指定的名字:

```
rootLogger=debug, STDOUT
```

使用这个配置，我们将把所有的`debug`和以上的消息记录到控制台。对于在本地环境中运行的控制台，`debug`级日志记录是常见的。

### 3.2.记录到文件

类似地，我们可以配置日志记录器来记录文件。这对于持久化日志通常很有用。让我们定义一个文件附加器:

```
appender.file.type = File
appender.file.name = LOGFILE
appender.file.fileName=logs/log4j.log
appender.file.layout.type=PatternLayout
appender.file.layout.pattern=[%-5level] %d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %c{1} - %msg%n
appender.file.filter.threshold.type = ThresholdFilter
appender.file.filter.threshold.level = info
```

对于文件附加器，还必须指定文件名。

除此之外，我们还要设置门槛级别。因为我们记录到一个文件，我们不想记录所有的消息，因为它会占用大量的持久存储。我们只想记录级别为`info`或以上的消息。**我们可以使用过滤器`ThresholdFilter `并设置其等级**和`**info.** `来实现

**要启用文件记录器，我们需要将文件附加器添加到根记录器中。**我们需要更改`rootLogger`配置，以包含文件附加器:

```
rootLogger=debug, STDOUT, LOGFILE
```

即使我们已经在根级别使用了级别`debug`，文件记录器也只会记录`info`及以上的消息。

## 4.测试

现在让我们运行应用程序并检查控制台中的输出:

```
12:43:47,891 INFO  Application:8 - Hello World!
12:43:47,892 DEBUG Application:9 - Hello World!
```

正如所料，我们可以在控制台中看到这两个日志消息。如果我们检查路径`logs/log4j.log`下的日志文件，我们只能看到`info`级别的日志消息:

```
12:43:47,891 INFO  Application:8 - Hello World!
```

## 5.结论

在本文中，我们学习了如何将消息记录到控制台和文件中。我们创建了一个 Java 项目，使用一个属性文件配置了 Log4j2，并测试了消息被打印到控制台和文件中。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221006181921/https://github.com/eugenp/tutorials/tree/master/logging-modules/log4j2)