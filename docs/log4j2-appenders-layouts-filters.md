# Log4j2 简介–附加器、布局和过滤器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/log4j2-appenders-layouts-filters>

## 1。概述

记录事件是软件开发的一个重要方面。虽然 Java 生态系统中有很多可用的框架，但是 Log4J 几十年来一直是最受欢迎的，因为它提供了灵活性和简单性。

[Log4j 2](/web/20220924071158/https://www.baeldung.com/java-logging-intro) 是经典 Log4j 框架的新改进版本。

在本文中，我们将通过实例介绍最常见的附加器、布局和过滤器。

在 Log4J2 中，appender 只是日志事件的目的地；它可以像控制台一样简单，也可以像任何 RDBMS 一样复杂。布局决定了日志的显示方式，过滤器根据不同的标准过滤数据。

## 2。设置

为了理解几个日志组件及其配置，让我们设置不同的测试用例，每个用例由一个`log4J2.xml`配置文件和一个`JUnit 4`测试类组成。

两个 maven 依赖项是所有示例共有的:

```java
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.7</version>
</dependency>
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.7</version>
    <type>test-jar</type>
    <scope>test</scope>
</dependency>
```

除了主 [`log4j-core`](https://web.archive.org/web/20220924071158/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.logging.log4j%22%20AND%20a%3A%22log4j-core%22) 包之外，我们还需要包含属于这个包的‘test jar ’,以访问测试不常见的配置文件所需的上下文规则。

## 3。默认配置

`ConsoleAppender`是`Log4J 2`核心包的默认配置。它以一种简单的模式向系统控制台记录消息:

```java
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN">
    <Appenders>
        <Console name="ConsoleAppender" target="SYSTEM_OUT">
            <PatternLayout 
              pattern="%d [%t] %-5level %logger{36} - %msg%n%throwable"/>
        </Console>
    </Appenders>
    <Loggers>
        <Root level="ERROR">
            <AppenderRef ref="ConsoleAppender"/>
        </Root>
    </Loggers>
</Configuration>
```

让我们分析这个简单的 XML 配置中的标记:

*   `Configuration`**:**`Log4J 2`配置文件的根元素和属性`status`是我们想要记录的内部 Log4J 事件的级别
*   `Appenders` **:** 这个元素包含一个或多个附加器。这里我们将配置一个 appender，它在标准输出端输出到系统控制台
*   `Loggers` **:** 该元素可以由多个配置好的`Logger` 元素组成。使用特殊的`Root` 标记，您可以配置一个无名的标准日志记录器，它将接收来自应用程序的所有日志消息。每个记录器都可以设置为最低日志级别
*   `AppenderRef` **:** 该元素定义了对来自`Appenders` 部分的元素的引用。因此，属性'`ref`'与 appenders ' `name`'属性相链接

相应的单元测试也同样简单。我们将获得一个`Logger` 引用并打印两条消息:

```java
@Test
public void givenLoggerWithDefaultConfig_whenLogToConsole_thanOK()
  throws Exception {
    Logger logger = LogManager.getLogger(getClass());
    Exception e = new RuntimeException("This is only a test!");

    logger.info("This is a simple message at INFO level. " +
      "It will be hidden.");
    logger.error("This is a simple message at ERROR level. " +
    "This is the minimum visible level.", e);
} 
```

## 4。`ConsoleAppender`同`PatternLayout`

让我们在一个单独的 XML 文件中定义一个带有定制颜色模式的新控制台 appender，并将其包含在我们的主配置中:

```java
<?xml version="1.0" encoding="UTF-8"?>
<Console name="ConsoleAppender" target="SYSTEM_OUT">
    <PatternLayout pattern="%style{%date{DEFAULT}}{yellow}
      %highlight{%-5level}{FATAL=bg_red, ERROR=red, WARN=yellow, INFO=green} 
      %message"/>
</Console>
```

该文件使用了一些在运行时被`Log4J 2`替换的模式变量:

*   `%style{…}{colorname}` **:** 这将以给定的颜色(`colorname`)打印第一对括号(`**…**`)中的文本。
*   `%highlight{…}{FATAL=colorname, …}` **:** 这类似于‘风格’变量。但是可以为每个日志级别指定不同的颜色。
*   `%date{format}` **:** 这将被指定的`format`中的当前日期替换。这里我们使用“默认”的日期时间格式，`‘` yyyy `-MM-dd HH:mm:ss,SSS'`。
*   `%-5level` **:** 以右对齐的方式打印日志消息的级别。
*   `%message` **:** 代表原始日志消息

但是在`PatternLayout` `.` 中存在更多的变量和格式，你可以参考 [Log4J 2](https://web.archive.org/web/20220924071158/https://logging.apache.org/log4j/2.x/) 的官方文档。

现在我们将把定义的控制台 appender 包含到我们的主配置中:

```java
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN" xmlns:xi="http://www.w3.org/2001/XInclude">
    <Appenders>
        <xi:include href="log4j2-includes/
          console-appender_pattern-layout_colored.xml"/>
    </Appenders>
    <Loggers>
        <Root level="DEBUG">
            <AppenderRef ref="ConsoleAppender"/>
        </Root>
    </Loggers>
</Configuration>
```

单元测试:

```java
@Test
public void givenLoggerWithConsoleConfig_whenLogToConsoleInColors_thanOK() 
  throws Exception {
    Logger logger = LogManager.getLogger("CONSOLE_PATTERN_APPENDER_MARKER");
    logger.trace("This is a colored message at TRACE level.");
    ...
} 
```

## 5。带有`JSONLayout`和`BurstFilter`和的异步文件附加器

有时，以异步方式编写日志消息很有用。例如，如果应用程序性能优先于日志的可用性。

在这样的用例中，我们可以使用一个`AsyncAppender.`

对于我们的例子，我们正在配置一个异步的`JSON`日志文件。此外，我们将包括一个突发过滤器，它将日志输出限制在指定的速率:

```java
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN">
    <Appenders>
        ...
        <File name="JSONLogfileAppender" fileName="target/logfile.json">
            <JSONLayout compact="true" eventEol="true"/>
            <BurstFilter level="INFO" rate="2" maxBurst="10"/>
        </File>
        <Async name="AsyncAppender" bufferSize="80">
            <AppenderRef ref="JSONLogfileAppender"/>
        </Async>
    </Appenders>
    <Loggers>
        ...
        <Logger name="ASYNC_JSON_FILE_APPENDER" level="INFO"
          additivity="false">
            <AppenderRef ref="AsyncAppender" />
        </Logger>
        <Root level="INFO">
            <AppenderRef ref="ConsoleAppender"/>
        </Root>
    </Loggers>
</Configuration>
```

请注意:

*   `JSONLayout`的配置方式是每行写一个日志事件
*   如果有两个以上的事件，那么`BurstFilter`将删除所有“信息”级别及以上的事件，但最多删除 10 个事件
*   `AsyncAppender`被设置为 80 个日志消息的缓冲区；之后，缓冲区被刷新到日志文件中

我们来看看相应的单元测试。我们在一个循环中填充附加的缓冲区，让它写入磁盘并检查日志文件的行数:

```java
@Test
public void givenLoggerWithAsyncConfig_whenLogToJsonFile_thanOK() 
  throws Exception {
    Logger logger = LogManager.getLogger("ASYNC_JSON_FILE_APPENDER");

    final int count = 88;
    for (int i = 0; i < count; i++) {
        logger.info("This is async JSON message #{} at INFO level.", count);
    }

    long logEventsCount 
      = Files.lines(Paths.get("target/logfile.json")).count();
    assertTrue(logEventsCount > 0 && logEventsCount <= count);
}
```

## 6。`RollingFile`追加器和`XMLLayout`

接下来，我们将创建一个滚动日志文件。达到配置的文件大小后，日志文件将被压缩和循环。

这次我们使用的是`XML` 布局:

```java
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN">
    <Appenders>
        <RollingFile name="XMLRollingfileAppender"
          fileName="target/logfile.xml"
          filePattern="target/logfile-%d{yyyy-MM-dd}-%i.log.gz">
            <XMLLayout/>
            <Policies>
                <SizeBasedTriggeringPolicy size="17 kB"/>
            </Policies>
        </RollingFile>
    </Appenders>
    <Loggers>
        <Logger name="XML_ROLLING_FILE_APPENDER" 
       level="INFO" additivity="false">
            <AppenderRef ref="XMLRollingfileAppender" />
        </Logger>
        <Root level="TRACE">
            <AppenderRef ref="ConsoleAppender"/>
        </Root>
    </Loggers>
</Configuration>
```

请注意:

*   `RollingFile` appender 有一个‘file pattern’属性，用于命名循环日志文件，可以用占位符变量进行配置。在我们的例子中，它应该在文件后缀前包含一个日期和一个计数器。
*   `XMLLayout`的默认配置将写入不带根元素的单个日志事件对象。
*   我们使用基于大小的策略来循环日志文件。

我们的单元测试类将类似于上一节中的那个:

```java
@Test
public void givenLoggerWithRollingFileConfig_whenLogToXMLFile_thanOK()
  throws Exception {
    Logger logger = LogManager.getLogger("XML_ROLLING_FILE_APPENDER");
    final int count = 88;
    for (int i = 0; i < count; i++) {
        logger.info(
          "This is rolling file XML message #{} at INFO level.", i);
    }
}
```

## 7。`Syslog`附录

假设我们需要通过网络将记录的事件发送到远程机器。使用 Log4J2 最简单的方法是使用它的`Syslog Appender:`

```java
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN">
    <Appenders>
        ...
        <Syslog name="Syslog" 
          format="RFC5424" host="localhost" port="514" 
          protocol="TCP" facility="local3" connectTimeoutMillis="10000" 
          reconnectionDelayMillis="5000">
        </Syslog>
    </Appenders>
    <Loggers>
        ...
        <Logger name="FAIL_OVER_SYSLOG_APPENDER" 
          level="INFO" 
          additivity="false">
            <AppenderRef ref="FailoverAppender" />
        </Logger>
        <Root level="TRACE">
            <AppenderRef ref="Syslog" />
        </Root>
    </Loggers>
</Configuration>
```

`Syslog` 标签中的属性:

*   `name` **:** 定义了追加人的名称，必须唯一。因为对于同一个应用程序和配置，我们可以有多个 Syslog appenders
*   `format` **:** 可以设置为 BSD 或 RFC5424，系统日志记录会相应格式化
*   `host & port` **:** 远程 Syslog 服务器机器的主机名和端口
*   `protocol` **:** 是用 TCP 还是 UPD
*   `facility` **:** 事件将被写入哪个系统日志设备
*   `connectTimeoutMillis` **:** 等待建立连接的时间段，默认为零
*   `reconnectionDelayMillis` **:** 重新尝试连接前等待的时间

## 8。`FailoverAppender`

现在可能有这样的情况，一个 appender 无法处理日志事件，我们不想丢失数据。在这种情况下，`FailoverAppender`就派上了用场。

例如，如果`Syslog` appender 未能将事件发送到远程机器，我们可能会暂时退回到`FileAppender`,而不是丢失数据。

`FailoverAppender`接受一个主 appender 和一些次 appender。如果主节点失败，它会尝试用辅助节点按顺序处理日志事件，直到一个节点成功或者没有任何辅助节点可供尝试:

```java
<Failover name="FailoverAppender" primary="Syslog">
    <Failovers>
        <AppenderRef ref="ConsoleAppender" />
    </Failovers>
</Failover>
```

让我们来测试一下:

```java
@Test
public void givenLoggerWithFailoverConfig_whenLog_thanOK()
  throws Exception {
    Logger logger = LogManager.getLogger("FAIL_OVER_SYSLOG_APPENDER");
    Exception e = new RuntimeException("This is only a test!"); 

    logger.trace("This is a syslog message at TRACE level.");
    logger.debug("This is a syslog message at DEBUG level.");
    logger.info("This is a syslog message at INFO level. 
      This is the minimum visible level.");
    logger.warn("This is a syslog message at WARN level.");
    logger.error("This is a syslog message at ERROR level.", e);
    logger.fatal("This is a syslog message at FATAL level.");
}
```

## 9。JDBC 附录

JDBC 附加器使用标准 JDBC 将日志事件发送到 RDBMS。可以使用任何 JNDI 数据源或任何连接工厂来获取连接。

基本配置由`DataSource` 或`ConnectionFactory`、列配置和`tableName:`组成

```java
<JDBC name="JDBCAppender" tableName="logs">
    <ConnectionFactory 
      class="com.baeldung.logging.log4j2.tests.jdbc.ConnectionFactory" 
      method="getConnection" />
    <Column name="when" isEventTimestamp="true" />
    <Column name="logger" pattern="%logger" />
    <Column name="level" pattern="%level" />
    <Column name="message" pattern="%message" />
    <Column name="throwable" pattern="%ex{full}" />
</JDBC>
```

现在让我们试试:

```java
@Test
public void givenLoggerWithJdbcConfig_whenLogToDataSource_thanOK()
  throws Exception {
    Logger logger = LogManager.getLogger("JDBC_APPENDER");
    final int count = 88;
    for (int i = 0; i < count; i++) {
        logger.info("This is JDBC message #{} at INFO level.", count);
    }

    Connection connection = ConnectionFactory.getConnection();
    ResultSet resultSet = connection.createStatement()
      .executeQuery("SELECT COUNT(*) AS ROW_COUNT FROM logs");
    int logCount = 0;
    if (resultSet.next()) {
        logCount = resultSet.getInt("ROW_COUNT");
    }
    assertTrue(logCount == count);
}
```

## 10。结论

这篇文章展示了一些非常简单的例子，说明如何在 Log4J2 中使用不同的日志附加器、过滤器和布局，以及配置它们的方法。

GitHub 上的[提供了本文附带的示例。](https://web.archive.org/web/20220924071158/https://github.com/eugenp/tutorials/tree/master/logging-modules/log4j2)