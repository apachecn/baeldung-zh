# 用 JUnit 断言日志消息

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/junit-asserting-logs>

## 1.介绍

在本教程中，我们将看看如何在 JUnit 测试中覆盖生成的日志。

我们将使用`slf4j-api`和 `**logback**` **实现**并创建**一个定制的 appender，我们可以用它来进行日志断言**。

## 2.Maven 依赖性

在我们开始之前，让我们添加 [`logback`](https://web.archive.org/web/20221130232653/https://search.maven.org/search?q=g:%22ch.qos.logback%22%20AND%20a:%22logback-classic%22) 依赖项。由于它本身实现了`slf4j-api`，所以它会被 Maven transitivity 自动下载并注入到项目中:

```java
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>. 
    <version>1.2.6</version>
</dependency>
```

[`AssertJ`](https://web.archive.org/web/20221130232653/https://search.maven.org/search?q=g:%22org.assertj%22%20AND%20a:%22assertj-core%22) 在测试时提供了非常有用的函数，所以让我们也将它的依赖项添加到项目中:

```java
<dependency>
    <groupId>org.assertj</groupId>
    <artifactId>assertj-core</artifactId>
    <version>3.15.0</version>
    <scope>test</scope>
</dependency>
```

## 3.一个基本的商业功能

现在，让我们创建一个对象，它将生成我们测试所基于的日志。

我们的`BusinessWorker`对象将只公开一个方法。此方法将为每个日志级别生成具有相同内容的日志。尽管这种方法在现实世界中并不那么有用，但它将很好地服务于我们的测试目的:

```java
public class BusinessWorker {
    private static Logger LOGGER = LoggerFactory.getLogger(BusinessWorker.class);

    public void generateLogs(String msg) {
        LOGGER.trace(msg);
        LOGGER.debug(msg);
        LOGGER.info(msg);
        LOGGER.warn(msg);
        LOGGER.error(msg);
    }
}
```

## 4.测试日志

我们想要生成日志，所以让我们在`src/test/resources`文件夹中创建一个`logback.xml`文件。让我们尽可能简单地将所有日志重定向到一个`CONSOLE`附加器:

```java
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <layout class="ch.qos.logback.classic.PatternLayout">
            <Pattern>
                %d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n
            </Pattern>
        </layout>
    </appender>

    <root level="error">
        <appender-ref ref="CONSOLE"/>
    </root>
</configuration>
```

### 4.1.`MemoryAppender`

现在，让我们创建一个在内存中保存日志的**定制 appender】。我们将**扩展`logback`提供的**的`ListAppender<ILoggingEvent>`，并且我们将用一些有用的方法来丰富它:**

```java
public class MemoryAppender extends ListAppender<ILoggingEvent> {
    public void reset() {
        this.list.clear();
    }

    public boolean contains(String string, Level level) {
        return this.list.stream()
          .anyMatch(event -> event.toString().contains(string)
            && event.getLevel().equals(level));
    }

    public int countEventsForLogger(String loggerName) {
        return (int) this.list.stream()
          .filter(event -> event.getLoggerName().contains(loggerName))
          .count();
    }

    public List<ILoggingEvent> search(String string) {
        return this.list.stream()
          .filter(event -> event.toString().contains(string))
          .collect(Collectors.toList());
    }

    public List<ILoggingEvent> search(String string, Level level) {
        return this.list.stream()
          .filter(event -> event.toString().contains(string)
            && event.getLevel().equals(level))
          .collect(Collectors.toList());
    }

    public int getSize() {
        return this.list.size();
    }

    public List<ILoggingEvent> getLoggedEvents() {
        return Collections.unmodifiableList(this.list);
    }
}
```

`MemoryAppender`类处理由日志记录系统自动填充的`List`。

它公开了各种方法，以涵盖广泛的测试目的:

*   `reset()`–清除列表
*   `contains(msg, level)`–仅当列表包含符合指定内容和严重性级别的`ILoggingEvent`时，返回`true`
*   `countEventForLoggers(loggerName)`–返回命名记录器生成的`ILoggingEvent`的编号
*   `search(msg)`–返回符合特定内容的`ILoggingEvent`的`List`
*   `search(msg, level)`–返回符合指定内容和严重性级别的`ILoggingEvent`的`List`
*   `getSize()`–返回`ILoggingEvent`的数量
*   `getLoggedEvents()`–返回`ILoggingEvent`元素的不可修改视图

### 4.2.单元测试

接下来，让我们为业务人员创建一个 JUnit 测试。

我们将把我们的`MemoryAppender`声明为一个字段，并以编程方式将其注入日志系统。然后，我们将启动附加器。

对于我们的测试，我们将级别设置为`DEBUG`:

```java
@Before
public void setup() {
    Logger logger = (Logger) LoggerFactory.getLogger(LOGGER_NAME);
    memoryAppender = new MemoryAppender();
    memoryAppender.setContext((LoggerContext) LoggerFactory.getILoggerFactory());
    logger.setLevel(Level.DEBUG);
    logger.addAppender(memoryAppender);
    memoryAppender.start();
}
```

现在我们可以创建一个简单的测试，我们实例化我们的`BusinessWorker`类并调用`generateLogs`方法。然后，我们可以对它生成的日志做出断言:

```java
@Test
public void test() {
    BusinessWorker worker = new BusinessWorker();
    worker.generateLogs(MSG);

    assertThat(memoryAppender.countEventsForLogger(LOGGER_NAME)).isEqualTo(4);
    assertThat(memoryAppender.search(MSG, Level.INFO).size()).isEqualTo(1);
    assertThat(memoryAppender.contains(MSG, Level.TRACE)).isFalse();
}
```

该测试使用了`MemoryAppender`的三个特性:

*   已经生成了四个日志——每个严重性应该有一个条目，并过滤了跟踪级别
*   只有一个内容为`message`且严重性级别为`INFO`的日志条目
*   不存在内容为`message`且严重性为`TRACE`的日志条目

如果我们计划在生成大量日志时，在同一个测试类中使用这个类的同一个实例，内存的使用将会增加。我们可以在每次测试前调用 **`MemoryAppender.clear()`方法来释放内存，避免`OutOfMemoryException`** 。

在本例中，我们将保留日志的范围缩小到了`LOGGER_NAME`包，我们将其定义为“`com.baeldung.junit.log`”。我们可能会保留所有使用 **`LoggerFactory.getLogger(Logger.ROOT_LOGGER_NAME),`的日志，但我们应该尽可能避免这样做，因为这会消耗大量内存**。

## 5.结论

通过这篇教程，我们已经展示了**如何在我们的单元测试**中覆盖日志生成。

和往常一样，代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221130232653/https://github.com/eugenp/tutorials/tree/master/testing-modules/testing-assertions)