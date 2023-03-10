# Java 日志记录简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-logging-intro>

## 1。概述

日志是理解和调试程序运行时行为的有力工具。日志捕获和保存重要数据，并使其可用于任何时间点的分析。

本文讨论了最流行的 java 日志框架 logroorks、Log4j 2 和 Logback，以及它们的前身 Log4j，并简要介绍了为不同日志框架提供公共接口的日志门面 SLF4J。

## 2。启用日志记录

本文中讨论的所有日志框架都有记录器、附加器和布局的概念。在项目中启用日志记录遵循三个常见步骤:

1.  添加所需的库
2.  配置
3.  放置日志语句

接下来的部分将分别讨论每个框架的步骤。

## 3。Log4j 2

Log4j 2 是 Log4j 日志框架的新的改进版本。最引人注目的改进是异步日志记录的可能性。Log4j 2 需要以下库:

```java
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-api</artifactId>
    <version>2.6.1</version>
</dependency>
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.6.1</version>
</dependency>
```

最新版本`log4j-api`你可以在这里找到，在这里找到`log4j-core`—。

### 3.1。配置

配置 Log4j 2 基于主配置`log4j2.xml`文件。首先要配置的是 appender。

这些决定了日志消息将被路由到哪里。目的地可以是控制台、文件、套接字等。

Log4j 2 有许多用于不同目的的附加器，你可以在官方 [Log4j 2](https://web.archive.org/web/20220908223507/https://logging.apache.org/log4j/2.x/manual/appenders.html) 网站上找到更多信息。

让我们看一个简单的配置示例:

```java
<Configuration status="debug" name="baeldung" packages="">
    <Appenders>
        <Console name="stdout" target="SYSTEM_OUT">
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss} %p %m%n"/>
        </Console>
    </Appenders>
</Configuration>
```

您可以为每个 appender 设置一个名称，例如使用名称`console`而不是`stdout`。

注意`PatternLayout` 元素——它决定了消息的外观。在我们的示例中，模式是基于`pattern` 参数设置的，其中`%d`确定日期模式，`%p`–输出日志级别，`%m`–输出记录的消息，`%n`–添加新的行符号。更多关于模式的信息你可以在官方 [Log4j 2](https://web.archive.org/web/20220908223507/https://logging.apache.org/log4j/2.x/manual/layouts.html) 页面找到。

最后——**要启用一个追加器**(或多个)，您需要将其添加到`<Root>`部分:

```java
<Root level="error">
    <AppenderRef ref="STDOUT"/>
</Root>
```

### 3.2。记录到文件

有时您需要将日志记录到文件中，因此我们将把`fout` logger 添加到我们的配置中:

```java
<Appenders>
    <File name="fout" fileName="baeldung.log" append="true">
        <PatternLayout>
            <Pattern>%d{yyyy-MM-dd HH:mm:ss} %-5p %m%nw</Pattern>
        </PatternLayout>
    </File>
</Appenders>
```

`File` appender 有几个可以配置的参数:

*   `file`–确定日志文件的文件名
*   `append`–该参数的默认值为 true，这意味着默认情况下,`File` appender 将追加到一个现有文件中，而不会将其截断。
*   这在前面的例子中已经描述过了。

为了启用`File` 追加器，您需要将其添加到`<Root>`部分:

```java
<Root level="INFO">
    <AppenderRef ref="stdout" />
    <AppenderRef ref="fout"/>
</Root> 
```

### 3.3。异步日志记录

如果你想让你的 Log4j 2 异步，你需要添加 LMAX disruptor 库到你的`pom.xml`。LMAX disruptor 是一个无锁的线程间通信库。

向 pom.xml 添加中断器:

```java
<dependency>
    <groupId>com.lmax</groupId>
    <artifactId>disruptor</artifactId>
    <version>3.3.4</version>
</dependency>
```

最新版本的干扰器可以在[这里](https://web.archive.org/web/20220908223507/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.lmax%22%20AND%20a%3A%22disruptor%22)找到。

如果你想使用 LMAX 干扰器，你需要在你的配置中使用`<asyncRoot>`而不是`<Root>`。

```java
<AsyncRoot level="DEBUG">
    <AppenderRef ref="stdout" />
    <AppenderRef ref="fout"/>
</AsyncRoot>
```

或者您可以通过将系统属性`Log4jContextSelector`设置为`org.apache.logging.log4j.core.async.AsyncLoggerContextSelector`来启用异步日志记录。

你当然可以阅读更多关于 Log4j2 异步记录器的配置，并在 [Log4j2 官方页面](https://web.archive.org/web/20220908223507/https://logging.apache.org/log4j/2.x/manual/async.html)上看到一些性能图。

### 3.4。用途

下面是一个简单的示例，演示了如何使用 Log4j 进行日志记录:

```java
import org.apache.logging.log4j.Logger;
import org.apache.logging.log4j.LogManager;

public class Log4jExample {

    private static Logger logger = LogManager.getLogger(Log4jExample.class);

    public static void main(String[] args) {
        logger.debug("Debug log message");
        logger.info("Info log message");
        logger.error("Error log message");
    }
}
```

运行后，应用程序将在控制台和名为`baeldung.log:`的文件中记录以下消息

```java
2016-06-16 17:02:13 INFO  Info log message
2016-06-16 17:02:13 ERROR Error log message
```

如果将根日志级别提升到`ERROR`:

```java
<level value="ERROR" />
```

输出将如下所示:

```java
2016-06-16 17:02:13 ERROR Error log message
```

如您所见，将日志级别更改为 upper 参数会导致日志级别较低的消息不会被打印到 appenders。

方法`logger.error`也可用于记录发生的异常:

```java
try {
    // Here some exception can be thrown
} catch (Exception e) {
    logger.error("Error log message", throwable);
}
```

### 3.5。封装级配置

假设您需要显示带有日志级别跟踪的消息——例如来自特定包(如`com.baeldung.log4j2`)的消息:

```java
logger.trace("Trace log message");
```

对于所有其他包，您希望继续只记录信息消息。

请记住，跟踪低于我们在配置中指定的根日志级别信息。

要只为其中一个包启用日志记录，您需要在`<Root>`之前向您的`log4j2.xml`添加以下部分:

```java
<Logger name="com.baeldung.log4j2" level="debug">
    <AppenderRef ref="stdout"/>
</Logger>
```

它将启用对`com.baeldung.log4j`包的日志记录，您的输出将如下所示:

```java
2016-06-16 17:02:13 TRACE Trace log message
2016-06-16 17:02:13 DEBUG Debug log message
2016-06-16 17:02:13 INFO  Info log message
2016-06-16 17:02:13 ERROR Error log message
```

## 4。回退

Logback 是 Log4j 的改进版本，由开发 Log4j 的同一开发人员开发。

与 Log4j 相比，Logback 还具有更多的特性，其中许多特性也被引入到 Log4j 2 中。下面快速浏览一下[官方网站](https://web.archive.org/web/20220908223507/http://logback.qos.ch/reasonsToSwitch.html)上 Logback 的所有优势。

让我们从向`pom.xml`添加以下依赖项开始:

```java
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.2.6</version>
</dependency>
```

这个依赖项将过渡地引入另外两个依赖项，即 `logback-core`和`slf4j-api`。注意，最新版本的 Logback 可以在[这里](https://web.archive.org/web/20220908223507/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22ch.qos.logback%22%20AND%20a%3A%22logback-classic%22)找到。

### 4.1。配置

现在让我们来看一个回退配置示例:

```java
<configuration>
  # Console appender
  <appender name="stdout" class="ch.qos.logback.core.ConsoleAppender">
    <layout class="ch.qos.logback.classic.PatternLayout">
      # Pattern of log message for console appender
      <Pattern>%d{yyyy-MM-dd HH:mm:ss} %-5p %m%n</Pattern>
    </layout>
  </appender>

  # File appender
  <appender name="fout" class="ch.qos.logback.core.FileAppender">
    <file>baeldung.log</file>
    <append>false</append>
    <encoder>
      # Pattern of log message for file appender
      <pattern>%d{yyyy-MM-dd HH:mm:ss} %-5p %m%n</pattern>
    </encoder>
  </appender>

  # Override log level for specified package
  <logger name="com.baeldung.log4j" level="TRACE"/>

  <root level="INFO">
    <appender-ref ref="stdout" />
    <appender-ref ref="fout" />
  </root>
</configuration>
```

Logback 使用 SLF4J 作为接口，所以需要导入 SLF4J 的`Logger`和`LoggerFactory.`

### 4.2。SLF4J

SLF4J 为大多数 Java 日志框架提供了一个公共接口和抽象。它作为一个门面，为访问日志框架的底层特性提供标准化的 API。

Logback 使用 SLF4J 作为其功能的本机 API。以下是使用回退日志记录的示例:

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class Log4jExample {

    private static Logger logger = LoggerFactory.getLogger(Log4jExample.class);

    public static void main(String[] args) {
        logger.debug("Debug log message");
        logger.info("Info log message");
        logger.error("Error log message");
    }
}
```

输出将保持与前面的示例相同。

## 5。Log4J

最后，让我们来看看备受尊敬的 Log4j 日志框架。

在这一点上，它当然过时了，但值得讨论，因为它为更现代的继任者奠定了基础。

许多配置细节与 Log4j 2 一节中讨论的相匹配。

### 5.1。配置

首先你需要将 Log4j 库添加到你的项目中`pom.xml:`

```java
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
```

在这里你应该可以找到 Log4j 的最新版本。

让我们来看一个简单 Log4j 配置的完整示例，它只有一个控制台附加器:

```java
<!DOCTYPE log4j:configuration SYSTEM "log4j.dtd" >
<log4j:configuration debug="false">

    <!--Console appender-->
    <appender name="stdout" class="org.apache.log4j.ConsoleAppender">
        <layout class="org.apache.log4j.PatternLayout">
            <param name="ConversionPattern" 
              value="%d{yyyy-MM-dd HH:mm:ss} %p %m%n" />
        </layout>
    </appender>

    <root>
        <level value="INFO" />
        <appender-ref ref="stdout" />
    </root>

</log4j:configuration>
```

`<log4j:configuration debug=”false”>`是整个配置的开放标签，有一个属性——`debug`。它决定了是否要将 Log4j 调试信息添加到日志中。

### 5.2。用途

添加 Log4j 库和配置后，您可以在代码中使用 logger。让我们看一个简单的例子:

```java
import org.apache.log4j.Logger;

public class Log4jExample {
    private static Logger logger = Logger.getLogger(Log4jExample.class);

    public static void main(String[] args) {
        logger.debug("Debug log message");
        logger.info("Info log message");
        logger.error("Error log message");
    }
}
```

## 6。结论

本文展示了一些非常简单的例子，说明如何使用不同的日志框架，比如 Log4j、Log4j2 和 Logback。它涵盖了所有提到的框架的简单配置示例。

本文附带的例子可以在 GitHub 上找到。