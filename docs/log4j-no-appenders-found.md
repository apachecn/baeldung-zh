# Log4j 警告:“找不到记录器的附加程序”

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/log4j-no-appenders-found>

## 1.概观

在本教程中，我们将展示如何修复警告，**“log4j:WARN No appenders can ' s found for logger”**。我们将解释什么是 appender 以及如何定义它。此外，我们将展示如何以不同的方式解决警告。

## 2.附录定义

我们先解释一下什么是 appender。Log4j 允许我们将日志放入多个目的地。**它打印输出的每个目的地被称为一个追加器**。我们有控制台、文件、JMS、GUI 组件等的附加器。

log4j 中没有定义默认的 appender。 **此外，[一个记录器可以有多个附加器](/web/20220906130055/https://www.baeldung.com/log4j2-appenders-layouts-filters)，**，在这种情况下，记录器将输出打印到所有附加器中。

## 3.警告消息解释

现在我们知道了什么是 appender，让我们来理解一下手头的问题。**警告信息表明没有为记录器找到附加程序。**

让我们创建一个`NoAppenderExample`类来重现警告:

```java
public class NoAppenderExample {
    private final static Logger logger = Logger.getLogger(NoAppenderExample.class);

    public static void main(String[] args) {
        logger.info("Info log message");
    }
}
```

我们在没有任何 log4j 配置的情况下运行我们的类。此后，我们可以在控制台输出中看到警告以及更多详细信息:

```java
log4j:WARN No appenders could be found for logger (com.baeldung.log4j.NoAppenderExample).
log4j:WARN Please initialize the log4j system properly.
log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
```

## 4.解决配置问题

默认情况下，Log4j 在应用程序的资源中查找配置文件，该文件可以是 XML 或 Java 属性格式。现在让我们定义 resources 目录下的`log4j.xml` 文件:

```java
<log4j:configuration debug="false">
    <!--Console appender -->
    <appender name="stdout" class="org.apache.log4j.ConsoleAppender">
        <layout class="org.apache.log4j.PatternLayout">
            <param name="ConversionPattern" value="%d{yyyy-MM-dd HH:mm:ss} %p %m%n"/>
        </layout>
    </appender>

    <root>
        <level value="DEBUG"/>
        <appender-ref ref="stdout"/>
    </root>
</log4j:configuration>
```

我们定义了`root `记录器，它位于记录器层次结构的顶层。所有应用程序记录器都是它的子组件，并覆盖它的配置。我们用一个 appender 定义了`root` logger，它将日志放入控制台。

让我们再次运行`NoAppenderExample`类并检查控制台输出。因此，日志包含我们的声明:

```java
2021-05-23 12:59:10 INFO Info log message
```

### 4.1.追加加法

不必为每个记录器定义一个 appender。给定记录器的记录请求将日志发送到为其定义的附加器以及为层次结构中更高的记录器指定的所有附加器。我们来举例说明一下。

如果记录器`A`定义了一个控制台附加器，并且记录器`B`是`A`的子节点，那么记录器`B`也会将其日志打印到控制台。**只有当中间祖先中的加法标志设置为`true`时，记录器才会从其祖先继承追加器。**如果相加性标志被设置为`false`，则层次结构中较高记录器的附加器不会被继承。

为了证明一个`logger`从祖先那里继承了 appenders，让我们在我们的`log4j.xml` 文件中为`NoAppenderExample`添加一个`logger`:

```java
<!DOCTYPE log4j:configuration SYSTEM "log4j.dtd" >
<log4j:configuration debug="false">
    ...
    <logger name="com.baeldung.log4j.NoAppenderExample" />
    ...
</log4j:configuration>
```

让我们再次运行`NoAppenderExample`类。这一次，日志语句出现在控制台中。**尽管`NoAppenderExample`记录器没有明确定义的附加器，但它从`root`记录器**继承了附加器。

## 5.配置文件不在类路径上

现在让我们考虑这样一种情况，我们希望在应用程序类路径之外定义配置文件。我们有两个选择:

*   使用`java`命令行选项:`-Dlog4j.configuration=<path to log4j configuration file>`指定文件的路径
*   在代码中定义路径:`PropertyConfigurator.configure(“<path to log4j properties file>”);`

在下一节中，我们将看到如何在 Java 代码中实现这一点。

## 6.用代码解决问题

假设我们不想要配置文件。让我们删除`log4.xml`文件并修改`main `方法:

```java
public class NoAppenderExample {
    private final static Logger logger = Logger.getLogger(NoAppenderExample.class);

    public static void main(String[] args) {
        BasicConfigurator.configure();
        logger.info("Info log message");
    }
}
```

我们从`BasicConfigurator`类中调用静态的`configure`方法。它将`ConsoleAppender `添加到`root `记录器中。让我们看看`configure` 方法的源代码:

```java
public static void configure() {
    Logger root = Logger.getRootLogger();
    root.addAppender(new ConsoleAppender(new PatternLayout("%r [%t] %p %c %x - %m%n")));
}
```

**因为 log4j 中的`root`记录器总是存在的，所以我们可以通过编程向它添加控制台 appender。**

## 7.结论

这个关于如何解决 log4j 关于缺少 appender 的警告的简短教程到此结束。我们解释了什么是 appender，以及如何用配置文件解决警告问题。然后，我们解释了 appender 加法是如何工作的。最后，我们展示了如何用代码解决这个警告。

和往常一样，这个例子的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220906130055/https://github.com/eugenp/tutorials/tree/master/logging-modules/log4j)