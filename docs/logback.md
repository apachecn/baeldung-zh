# 日志备份指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/logback>

## 1。概述

[Logback](https://web.archive.org/web/20220929130816/https://logback.qos.ch/) 是 Java 社区中使用最广泛的日志框架之一。这是它的前身 Log4j 的[的替代品。](https://web.archive.org/web/20220929130816/https://logback.qos.ch/reasonsToSwitch.html) Logback 提供了更快的实施速度，提供了更多的配置选项，并且在归档旧日志文件方面更加灵活。

## 延伸阅读:

## [SLF4J 简介](/web/20220929130816/https://www.baeldung.com/slf4j-with-log4j2-logback)

A quick and to the point guide of how to use Log4j2 and Logback with SLF4J, as well as how to bridge other logging APIs such as JCL to SLF4J[Read more](/web/20220929130816/https://www.baeldung.com/slf4j-with-log4j2-logback) →

## [发送带有回退功能的电子邮件](/web/20220929130816/https://www.baeldung.com/logback-send-email)

Learn how to configure Logback for sending out an email notification for any application errors[Read more](/web/20220929130816/https://www.baeldung.com/logback-send-email) →

## [Java 日志简介](/web/20220929130816/https://www.baeldung.com/java-logging-intro)

A quick intro to logging in Java - the libraries, the configuration details as well as pros and cons of each solution.[Read more](/web/20220929130816/https://www.baeldung.com/java-logging-intro) →

在本教程中，我们将介绍 Logback 的架构，并研究如何使用它来使我们的应用程序更好。

## 2。回溯架构

回溯架构由三个类组成:`Logger`、`Appender`和`Layout`。

`Logger`是日志消息的上下文。这是应用程序与之交互以创建日志消息的类。

将日志消息放在它们的最终目的地。一个`Logger`可以有多个`Appender`。我们通常认为`Appenders`是附加到文本文件的，但是 Logback 比它更有效。

`Layout`准备输出信息。Logback 支持为格式化消息创建定制类，以及为现有类创建健壮的配置选项。

## 3。设置

### 3.1。Maven 依赖关系

Logback 使用 Java 的简单日志门面(SLF4J)作为其本机接口。在开始记录消息之前，我们需要将 Logback 和 SLF4J 添加到我们的`pom.xml`:

```java
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-core</artifactId>
    <version>1.2.6</version>
</dependency>

<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.30</version>
    <scope>test</scope>
</dependency> 
```

Maven Central 有[最新版本的 Logback 核心](https://web.archive.org/web/20220929130816/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22ch.qos.logback%22%20AND%20a%3A%22logback-core%22)和[最新版本的`slf4j-api`](https://web.archive.org/web/20220929130816/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.slf4j%22%20AND%20a%3A%22slf4j-api%22) 。

### 3.2。类路径

Logback 还需要运行时类路径上的 [`logback-classic.jar `](https://web.archive.org/web/20220929130816/https://search.maven.org/classic/#search%7Cga%7C1%7Clogback-classic) 。

我们将把它作为测试依赖项添加到`pom.xml`:

```java
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.2.6</version>
</dependency> 
```

## 4.基本示例和配置

让我们从一个在应用程序中使用 Logback 的简单例子开始。

首先，我们需要一个配置文件。我们将创建一个名为`logback.xml` 的文本文件，并将其放在我们的类路径中:

```java
<configuration>
  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
      <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
    </encoder>
  </appender>

  <root level="debug">
    <appender-ref ref="STDOUT" />
  </root>
</configuration>
```

接下来，我们需要一个带有`main`方法的简单类:

```java
public class Example {

    private static final Logger logger 
      = LoggerFactory.getLogger(Example.class);

    public static void main(String[] args) {
        logger.info("Example log from {}", Example.class.getSimpleName());
    }
}
```

这个类创建一个`Logger`并调用`info()`来生成一个日志消息。

当我们运行`Example,`时，我们看到我们的消息被记录到控制台:

```java
20:34:22.136 [main] INFO Example - Example log from Example
```

很容易理解为什么 Logback 如此受欢迎；我们几分钟后就可以开始运行了。

这个配置和代码给了我们一些提示，告诉我们它是如何工作的:

1.  我们有一个名为`STDOUT`的`appender `，它引用了类名`ConsoleAppender.`
2.  有一种模式描述了我们的日志消息的格式。
3.  我们的代码创建了一个`Logger`，我们通过一个`info()` 方法`.`将我们的消息传递给它

现在我们已经了解了基础知识，让我们仔细看看。

## 5。`Logger`语境

### 5.1。创建上下文

为了将一条消息记录到 Logback，我们从 SLF4J 或 Logback 初始化一个`Logger`:

```java
private static final Logger logger 
  = LoggerFactory.getLogger(Example.class); 
```

然后我们使用它:

```java
logger.info("Example log from {}", Example.class.getSimpleName()); 
```

这是我们的日志上下文。当我们创建它时，我们通过了我们的类。这给了`Logger`一个名字(还有一个接受`String). `的重载)

日志记录上下文存在于与 Java 对象层次结构非常相似的层次结构中:

1.  **一个`logger`是一个祖先，当它的名字后面跟一个点，前缀是一个后代`logger`的名字**
2.  **当一个`logger`和一个子**之间没有祖先时，它就是一个父

比如下面的`Example`类就在`com.baeldung.logback`包里。在`com.baeldung.logback.appenders`包中还有一个名为`ExampleAppender`的职业。

`ExampleAppender's Logger`是`Example's Logger.`的孩子

**所有`loggers`都是预定义根记录器的后代。**

一个`Logger`有一个`Level,`,可以通过配置或者用`Logger.setLevel().`在代码覆盖配置文件中设置级别来设置。

按照优先顺序，可能的级别有:`TRACE, DEBUG, INFO, WARN`和`ERROR.` 每个级别都有一个相应的方法，我们用它来记录该级别的消息。

**如果一个`Logger`没有被明确地分配一个等级，它继承它最近的祖先的等级。**root logger 默认为`DEBUG.`，我们将在下面看到如何覆盖它。

### 5.2。使用上下文

让我们创建一个示例程序，演示如何在日志层次结构中使用上下文:

```java
ch.qos.logback.classic.Logger parentLogger = 
  (ch.qos.logback.classic.Logger) LoggerFactory.getLogger("com.baeldung.logback");

parentLogger.setLevel(Level.INFO);

Logger childlogger = 
  (ch.qos.logback.classic.Logger)LoggerFactory.getLogger("com.baeldung.logback.tests");

parentLogger.warn("This message is logged because WARN > INFO.");
parentLogger.debug("This message is not logged because DEBUG < INFO.");
childlogger.info("INFO == INFO");
childlogger.debug("DEBUG < INFO"); 
```

当我们运行它时，我们会看到这些消息:

```java
20:31:29.586 [main] WARN com.baeldung.logback - This message is logged because WARN > INFO.
20:31:29.594 [main] INFO com.baeldung.logback.tests - INFO == INFO
```

我们首先检索一个名为`com.baeldung.logback`的`Logger`，并将其转换为一个`ch.qos.logback.classic.Logger.`

需要一个回退上下文来设置下一条语句中的级别；注意，SLF4J 的抽象`logger`没有实现`setLevel().`

我们将上下文的级别设置为`INFO`。然后我们创建另一个名为`com.baeldung.logback.tests.`的 `logger`

最后，我们为每个上下文记录两条消息来演示层次结构。Logback 记录`WARN`和`INFO`消息，过滤`DEBUG` 消息。

现在让我们使用根日志记录器:

```java
ch.qos.logback.classic.Logger logger = 
  (ch.qos.logback.classic.Logger)LoggerFactory.getLogger("com.baeldung.logback");
logger.debug("Hi there!");

Logger rootLogger = 
  (ch.qos.logback.classic.Logger)LoggerFactory.getLogger(org.slf4j.Logger.ROOT_LOGGER_NAME);
logger.debug("This message is logged because DEBUG == DEBUG.");

rootLogger.setLevel(Level.ERROR);

logger.warn("This message is not logged because WARN < ERROR.");
logger.error("This is logged."); 
```

当我们执行以下代码片段时，我们会看到这些消息:

```java
20:44:44.241 [main] DEBUG com.baeldung.logback - Hi there!
20:44:44.243 [main] DEBUG com.baeldung.logback - This message is logged because DEBUG == DEBUG.
20:44:44.243 [main] ERROR com.baeldung.logback - This is logged. 
```

总而言之，我们从一个`Logger`上下文开始，并打印了一条`DEBUG`消息。

然后，我们使用静态定义的名称检索根日志记录器，并将其级别设置为`ERROR.`

最后，我们证明了 Logback 确实过滤了任何小于错误的语句。

### 5.3。参数化消息

与上面示例片段中的消息不同，大多数有用的日志消息需要附加`Strings.`，这需要分配内存、序列化对象、连接`Strings,`并可能在以后清理垃圾。

考虑以下消息:

```java
log.debug("Current count is " + count); 
```

无论`Logger`是否记录消息，我们都会产生构建消息的成本。

Logback 通过其参数化消息提供了一种替代方案:

```java
log.debug("Current count is {}", count); 
```

**大括号{}将接受任何`Object`并使用它的`toString()`方法，仅在验证日志消息是必需的之后才构建消息。**

让我们尝试一些不同的参数:

```java
String message = "This is a String";
Integer zero = 0;

try {
    logger.debug("Logging message: {}", message);
    logger.debug("Going to divide {} by {}", 42, zero);
    int result = 42 / zero;
} catch (Exception e) {
    logger.error("Error dividing {} by {} ", 42, zero, e);
} 
```

这个片段产生:

```java
21:32:10.311 [main] DEBUG com.baeldung.logback.LogbackTests - Logging message: This is a String
21:32:10.316 [main] DEBUG com.baeldung.logback.LogbackTests - Going to divide 42 by 0
21:32:10.316 [main] ERROR com.baeldung.logback.LogbackTests - Error dividing 42 by 0
java.lang.ArithmeticException: / by zero
  at com.baeldung.logback.LogbackTests.givenParameters_ValuesLogged(LogbackTests.java:64)
... 
```

我们看到如何将一个`String,`、一个`int,`和一个`Integer`作为参数传入。

此外，当一个`Exception`作为最后一个参数传递给一个日志方法时，Logback 将为我们打印堆栈跟踪。

## 6。详细配置

在前面的例子中，我们使用在第 4 节中创建的 11 行配置文件将日志消息打印到控制台。这是 Logback 的默认行为；如果找不到配置文件，它会创建一个`ConsoleAppender `并将它与根日志记录器关联起来。

### 6.1。定位配置信息

配置文件可以放在类路径中，命名为`logback.xml`或`logback-test.xml.`

以下是 Logback 尝试查找配置数据的方式:

1.  在类路径中依次搜索名为`logback-test.xml` `, logback.groovy,` 或`logback.xml`的文件
2.  如果这个库没有找到这些文件，它将尝试使用 Java 的`[ServiceLoader](https://web.archive.org/web/20220929130816/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/ServiceLoader.html)`来定位`com.qos.logback.classic.spi.Configurator.`的实现者
3.  将自身配置为将输出直接记录到控制台

**重要提示**:由于 logback 的官方文档，他们已经停止支持 logback.groovy 了，所以如果你想在你的应用中配置 Logback，还是用 XML 版本比较好。

### 6.2.基本配置

让我们仔细看看我们的[示例配置。](#example)

整个文件都在`<configuration>` 标签中。

**我们看到一个标签声明了一个类型为`ConsoleAppender`的`Appender`，并将其命名为`STDOUT`。嵌套在该标签中的是一个编码器。它有一个类似于`sprintf-style`转义码的模式:**

```java
<appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
        <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
    </encoder>
</appender>
```

最后，我们看到一个`root`标签。该标签将根日志记录器设置为`DEBUG`模式，并将其输出与名为`STDOUT`的`Appender`相关联:

```java
<root level="debug">
    <appender-ref ref="STDOUT" />
</root>
```

### 6.3。故障排除配置

回溯配置文件可能会变得复杂，因此有几种内置的故障排除机制。

为了在 Logback 处理配置时查看调试信息，我们可以打开调试日志记录:

```java
<configuration debug="true">
  ...
</configuration>
```

Logback 在处理配置时会将状态信息打印到控制台:

```java
23:54:23,040 |-INFO in ch.qos.logback.classic.LoggerContext[default] - Found resource [logback-test.xml] 
  at [file:/Users/egoebelbecker/ideaProjects/logback-guide/out/test/resources/logback-test.xml]
23:54:23,230 |-INFO in ch.qos.logback.core.joran.action.AppenderAction - About to instantiate appender 
  of type [ch.qos.logback.core.ConsoleAppender]
23:54:23,236 |-INFO in ch.qos.logback.core.joran.action.AppenderAction - Naming appender as [STDOUT]
23:54:23,247 |-INFO in ch.qos.logback.core.joran.action.NestedComplexPropertyIA - Assuming default type 
  [ch.qos.logback.classic.encoder.PatternLayoutEncoder] for [encoder] property
23:54:23,308 |-INFO in ch.qos.logback.classic.joran.action.RootLoggerAction - Setting level of ROOT logger to DEBUG
23:54:23,309 |-INFO in ch.qos.logback.core.joran.action.AppenderRefAction - Attaching appender named [STDOUT] to Logger[ROOT]
23:54:23,310 |-INFO in ch.qos.logback.classic.joran.action.ConfigurationAction - End of configuration.
23:54:23,313 |-INFO in [[email protected]](/web/20220929130816/https://www.baeldung.com/cdn-cgi/l/email-protection) - Registering current configuration 
  as safe fallback point
```

如果解析配置文件时遇到警告或错误，Logback 会将状态消息写入控制台。

还有第二种打印状态信息的机制:

```java
<configuration>
    <statusListener class="ch.qos.logback.core.status.OnConsoleStatusListener" />  
    ...
</configuration>
```

**`StatusListener`截取状态信息并在配置过程中打印出来，同时程序也在运行**。

所有配置文件的输出都被打印出来，这对于在类路径上定位“流氓”配置文件非常有用。

### 6.4。自动重新加载配置

在应用程序运行时重新加载日志配置是一个强大的故障排除工具。Logback 通过`scan`参数实现了这一点:

```java
<configuration scan="true">
  ...
</configuration>
```

默认行为是每 60 秒扫描一次配置文件的更改。我们可以通过添加`scanPeriod`来修改这个间隔:

```java
<configuration scan="true" scanPeriod="15 seconds">
  ...
</configuration>
```

我们可以用毫秒、秒、分钟或小时来指定值。

### 6.5.修改`Loggers`

在上面的示例文件中，我们设置了根日志记录器的级别，并将其与控制台`Appender`相关联。

我们可以为任何`logger`设置级别:

```java
<configuration>
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    <logger name="com.baeldung.logback" level="INFO" /> 
    <logger name="com.baeldung.logback.tests" level="WARN" /> 
    <root level="debug">
        <appender-ref ref="STDOUT" />
    </root>
</configuration>
```

让我们将它添加到我们的类路径中，并运行代码:

```java
Logger foobar = 
  (ch.qos.logback.classic.Logger) LoggerFactory.getLogger("com.baeldung.foobar");
Logger logger = 
  (ch.qos.logback.classic.Logger) LoggerFactory.getLogger("com.baeldung.logback");
Logger testslogger = 
  (ch.qos.logback.classic.Logger) LoggerFactory.getLogger("com.baeldung.logback.tests");

foobar.debug("This is logged from foobar");
logger.debug("This is not logged from logger");
logger.info("This is logged from logger");
testslogger.info("This is not logged from tests");
testslogger.warn("This is logged from tests"); 
```

我们看到这样的输出:

```java
00:29:51.787 [main] DEBUG com.baeldung.foobar - This is logged from foobar
00:29:51.789 [main] INFO com.baeldung.logback - This is logged from logger
00:29:51.789 [main] WARN com.baeldung.logback.tests - This is logged from tests 
```

通过不编程地设置我们的`Loggers`的级别，配置设置它们；`**com.baeldung.foobar**` **从根记录器继承** `** DEBUG**` **。**

`Loggers`也从根记录器继承`appender-ref`。正如我们将在下面看到的，我们可以覆盖它。

### 6.6。变量替换

回溯配置文件支持变量。我们在配置脚本内部或外部定义变量。可以在配置脚本中的任何位置指定变量来代替值。

例如，下面是一个`FileAppender`的配置:

```java
<property name="LOG_DIR" value="/var/log/application" />
<appender name="FILE" class="ch.qos.logback.core.FileAppender">
    <file>${LOG_DIR}/tests.log</file>
    <append>true</append>
    <encoder>
        <pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>
    </encoder>
</appender>
```

在配置的顶部，我们声明了一个名为`LOG_DIR.` 的`property` ，然后我们使用它作为`appender`定义内文件路径的一部分。

属性在配置脚本中的`<property>`标签中声明，但是它们也可以从外部来源获得，比如系统属性。我们可以省略这个例子中的`property`声明，在命令行中设置`LOG_DIR`的值:

```java
$ java -DLOG_DIR=/var/log/application com.baeldung.logback.LogbackTests
```

我们用`${propertyname}.` Logback 指定属性的值实现变量作为文本替换。变量替换可以发生在配置文件中可以指定值的任何位置。

## 7。`Appenders`

`Loggers`通过`LoggingEvents`到`Appenders.`T3 做实际的测井工作。我们通常认为日志是指向文件或控制台的东西，但是 Logback 可以做得更多。`Logback-core`提供了几个有用的`appenders`。

### 7.1。`ConsoleAppender`

我们已经看到了`ConsoleAppender` 的动作。尽管名字如此，`ConsoleAppender`将消息附加到`System.out` 或`System.err.`

它使用一个`OutputStreamWriter`来缓冲 I/O，因此将它指向`System.err`不会导致无缓冲写入。

### 7.2。`FileAppender`

`FileAppender` 将消息追加到文件中。它支持广泛的配置参数。让我们将文件`appender`添加到我们的基本配置中:

```java
<configuration debug="true">
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <!-- encoders are assigned the type
             ch.qos.logback.classic.encoder.PatternLayoutEncoder by default -->
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <appender name="FILE" class="ch.qos.logback.core.FileAppender">
        <file>tests.log</file>
        <append>true</append>
        <encoder>
            <pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>
        </encoder>
    </appender>

    <logger name="com.baeldung.logback" level="INFO" /> 
    <logger name="com.baeldung.logback.tests" level="WARN"> 
        <appender-ref ref="FILE" /> 
    </logger> 

    <root level="debug">
        <appender-ref ref="STDOUT" />
    </root>
</configuration>
```

`FileAppender`通过`<file>.`配置了一个文件名`<append>` 标签指示`Appender` 追加到一个已有的文件中，而不是截断它。如果我们多次运行测试，我们会看到日志输出被附加到同一个文件中。

如果我们从上面重新运行我们的测试，来自`com.baeldung.logback.tests`的消息会同时发送到控制台和一个名为 tests.log. **的文件。后代`logger`继承了根日志记录器与`ConsoleAppender`的关联，而它与`FileAppender.` `Appenders`的关联是累积的。**

我们可以覆盖这种行为:

```java
<logger name="com.baeldung.logback.tests" level="WARN" additivity="false" > 
    <appender-ref ref="FILE" /> 
</logger> 

<root level="debug">
    <appender-ref ref="STDOUT" />
</root> 
```

将`additivity`设置为`false` 会禁用默认行为。`Tests`不会登录到控制台，它的后代也不会。

### 7.3。`RollingFileAppender`

通常，将日志消息附加到同一个文件不是我们需要的行为。我们希望文件基于时间、日志文件大小或两者的组合“滚动”。

为此，我们有`RollingFileAppender:`

```java
<property name="LOG_FILE" value="LogFile" />
<appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>${LOG_FILE}.log</file>
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
        <!-- daily rollover -->
        <fileNamePattern>${LOG_FILE}.%d{yyyy-MM-dd}.gz</fileNamePattern>

        <!-- keep 30 days' worth of history capped at 3GB total size -->
        <maxHistory>30</maxHistory>
        <totalSizeCap>3GB</totalSizeCap>
    </rollingPolicy>
    <encoder>
        <pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>
    </encoder>
</appender> 
```

一个`RollingFileAppender` 有一个`RollingPolicy.` 在这个样本配置中，我们看到一个`TimeBasedRollingPolicy.`

类似于`FileAppender,`,我们用文件名配置了这个`appender`。我们声明了一个属性并使用了它，因为我们将重用下面的文件名。

我们在`RollingPolicy.` 中定义了一个`fileNamePattern`。这个模式不仅定义了文件名，还定义了滚动它们的频率。`TimeBasedRollingPolicy` 检查图案并在最精确定义的周期滚动。

例如:

```java
<property name="LOG_FILE" value="LogFile" />
<property name="LOG_DIR" value="/var/logs/application" />
<appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>${LOG_DIR}/${LOG_FILE}.log</file> 
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
        <fileNamePattern>${LOG_DIR}/%d{yyyy/MM}/${LOG_FILE}.gz</fileNamePattern>
        <totalSizeCap>3GB</totalSizeCap>
    </rollingPolicy>
```

活动日志文件是`/var/logs/application/LogFile.` 该文件在每月初转入`/Current Year/Current Month/LogFile.gz`**`RollingFileAppender`创建一个新的活动文件。**

 **当归档文件的总大小达到 3GB 时，`RollingFileAppender` 以先进先出的方式删除归档文件。

有星期、小时、分钟、秒甚至毫秒的代码。Logback 在这里有一个引用。

`RollingFileAppender` 也内置了对压缩文件的支持。它压缩我们的滚动文件，因为我们把它们命名为`LogFile.gz.`

`TimeBasedPolicy` 并不是我们滚动文件的唯一选择。Logback 还提供了`SizeAndTimeBasedRollingPolicy,` ，它将根据当前日志文件的大小和时间进行回滚。它还提供了一个`FixedWindowRollingPolicy,` ，在每次启动记录器时滚动日志文件名。

我们也可以自己写`[RollingPolicy](https://web.archive.org/web/20220929130816/https://logback.qos.ch/manual/appenders.html#onRollingPolicies).`

### 7.4。`Appenders`风俗

我们可以通过扩展 Logback 的一个基类`appender`来创建自定义的`appenders`。在这里我们有一个创建自定义`appenders`T3 的教程。

## 8。`Layouts`

`Layouts`格式化日志消息。像其余的 Logback 一样，`Layouts` 是可扩展的，我们可以[创建自己的。然而，默认的`PatternLayout`提供了大多数应用程序所需要的，甚至更多。](https://web.archive.org/web/20220929130816/https://logback.qos.ch/manual/layouts.html#writingYourOwnLayout)

到目前为止，我们在所有的例子中都使用了`PatternLayout`:

```java
<encoder>
    <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
</encoder> 
```

这个配置脚本包含对`PatternLayoutEncoder.` 的配置，我们将一个`Encoder`传递给我们的`Appender,` ，这个编码器使用`PatternLayout`来格式化消息。

`<pattern>`标签中的文本定义了日志消息的格式。 **`PatternLayout`实现了大量用于创建模式的转换词和格式修饰符。**

让我们把这个分解一下。`PatternLayout`识别带%的转换单词，因此我们模式中的转换生成:

*   `%d{HH:mm:ss.SSS}`–带有小时、分钟、秒和毫秒的时间戳
*   `[%thread]`–生成日志消息的线程名，用方括号括起来
*   `%-5level`–记录事件的级别，填充为 5 个字符
*   `%logger{36}`— `logger`的名称，被截断为 35 个字符
*   `%msg%n`–日志消息，后跟平台相关的行分隔符

所以我们看到类似这样的消息:

```java
21:32:10.311 [main] DEBUG com.baeldung.logback.LogbackTests - Logging message: This is a String
```

转换词和格式修饰语的详细列表可以在[这里](https://web.archive.org/web/20220929130816/https://logback.qos.ch/manual/layouts.html#conversionWord)找到。

## 9.结论

在这篇内容丰富的文章中，我们介绍了在应用程序中使用 Logback 的基础知识。

我们查看了 Logback 架构中的三个主要组件:`Logger`、 `Appender`和`Layout`。Logback 有强大的配置脚本，我们用它来操作组件来过滤和格式化消息。我们还讨论了两个最常用的文件附加器，用于创建、翻转、组织和压缩日志文件。

像往常一样，代码片段可以在 GitHub 上找到[。](https://web.archive.org/web/20220929130816/https://github.com/eugenp/tutorials/tree/master/logging-modules/logback)**