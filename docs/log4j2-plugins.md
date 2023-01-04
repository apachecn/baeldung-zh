# Log4j 2 插件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/log4j2-plugins>

## 1.概观

Log4j 2 使用类似 Appenders 和 Layouts 的[插件](/web/20220728144649/https://www.baeldung.com/log4j2-appenders-layouts-filters)来格式化和输出日志。这些被称为核心插件，Log4j 2 提供了很多[选项](https://web.archive.org/web/20220728144649/https://logging.apache.org/log4j/2.x/manual/plugins.html)供我们选择。

然而，在某些情况下，我们可能还需要扩展现有的插件，甚至编写自定义插件。

在本教程中，我们将使用 Log4j 2 扩展机制来实现定制插件。

## 2.扩展 Log4j 2 插件

Log4j 2 中的插件大致分为五类:

1.  核心插件
2.  转换器
3.  主要供应商
4.  查找
5.  类型转换器

Log4j 2 允许我们使用一种通用的机制来实现上述所有类别的定制插件。此外，它还允许我们用同样的方法扩展现有的插件。

在 Log4j 1.x 中，扩展现有插件的唯一方法是覆盖它的实现类。**另一方面，Log4j 2 通过用`@Plugin.`** 注释一个类，使得扩展现有插件变得更加容易

在接下来的几节中，我们将在这些类别中实现一个自定义插件。

## 3.核心插件

### 3.1.实现自定义核心插件

在 Log4j 2 `.`中，像附加器、布局和过滤器这样的关键元素被称为核心插件。虽然这类插件有很多种，但在某些情况下，我们可能需要实现一个定制的核心插件。例如，考虑一个只将日志记录写入内存中的`List`的`[ListAppender](/web/20220728144649/https://www.baeldung.com/log4j2-custom-appender)`:

```java
@Plugin(name = "ListAppender", 
  category = Core.CATEGORY_NAME, 
  elementType = Appender.ELEMENT_TYPE)
public class ListAppender extends AbstractAppender {

    private List<LogEvent> logList;

    protected ListAppender(String name, Filter filter) {
        super(name, filter, null);
        logList = Collections.synchronizedList(new ArrayList<>());
    }

    @PluginFactory
    public static ListAppender createAppender(
      @PluginAttribute("name") String name, @PluginElement("Filter") final Filter filter) {
        return new ListAppender(name, filter);
    }

    @Override
    public void append(LogEvent event) {
        if (event.getLevel().isLessSpecificThan(Level.WARN)) {
            error("Unable to log less than WARN level.");
            return;
        }
        logList.add(event);
    }
}
```

**我们用`@Plugin `注释了这个类，允许我们将插件命名为**。同样，参数用`@PluginAttribute. `标注。像过滤器或布局这样的嵌套元素作为`@PluginElement.`传递。现在我们可以在配置中使用相同的名称引用这个插件:

```java
<?xml version="1.0" encoding="UTF-8"?>
<Configuration xmlns:xi="http://www.w3.org/2001/XInclude"
    packages="com.baeldung" status="WARN">
    <Appenders>
        <ListAppender name="ListAppender">
            <BurstFilter level="INFO" rate="16" maxBurst="100"/>
        </MapAppender>
    </Appenders>
    <Loggers
        <Root level="DEBUG">
            <AppenderRef ref="ConsoleAppender" />
            <AppenderRef ref="ListAppender" />
        </Root>
    </Loggers>
</Configuration>
```

### 3.2.插件构建器

上一节中的例子相当简单，只接受一个参数`name. `。一般来说，像 appenders 这样的核心插件要复杂得多，通常接受几个可配置的参数。

例如，考虑一个将日志写入`Kafka:`的 appender

```java
<Kafka2 name="KafkaLogger" ip ="127.0.0.1" port="9010" topic="log" partition="p-1">
    <PatternLayout pattern="%pid%style{%message}{red}%n" />
</Kafka2>
```

**为了实现这样的附加器，Log4j 2 提供了一个基于 [`Builder`模式](/web/20220728144649/https://www.baeldung.com/creational-design-patterns)** 的插件生成器实现:

```java
@Plugin(name = "Kafka2", category = Core.CATEGORY_NAME)
public class KafkaAppender extends AbstractAppender {

    public static class Builder implements org.apache.logging.log4j.core.util.Builder<KafkaAppender> {

        @PluginBuilderAttribute("name")
        @Required
        private String name;

        @PluginBuilderAttribute("ip")
        private String ipAddress;

        // ... additional properties

        // ... getters and setters

        @Override
        public KafkaAppender build() {
            return new KafkaAppender(
              getName(), getFilter(), getLayout(), true, new KafkaBroker(ipAddress, port, topic, partition));
        }
    }

    private KafkaBroker broker;

    private KafkaAppender(String name, Filter filter, Layout<? extends Serializable> layout, 
      boolean ignoreExceptions, KafkaBroker broker) {
        super(name, filter, layout, ignoreExceptions);
        this.broker = broker;
    }

    @Override
    public void append(LogEvent event) {
        connectAndSendToKafka(broker, event);
    }
}
```

简而言之，我们引入了一个`Builder `类并用`@PluginBuilderAttribute. `标注参数，因此，`KafkaAppender`从上面显示的配置中接受 Kafka 连接参数。

### 3.3.扩展现有插件

我们也可以在 Log4j 2 `.` 中扩展一个现有的核心插件，我们可以通过给我们的插件起一个与现有插件相同的名字来实现。例如，如果我们扩展了`RollingFileAppender:`

```java
@Plugin(name = "RollingFile", category = Core.CATEGORY_NAME, elementType = Appender.ELEMENT_TYPE)
public class RollingFileAppender extends AbstractAppender {

    public RollingFileAppender(String name, Filter filter, Layout<? extends Serializable> layout) {
        super(name, filter, layout);
    }
    @Override
    public void append(LogEvent event) {
    }
}
```

值得注意的是，我们现在有两个同名的追加器。在这样的场景中，Log4j 2 将使用最先发现的 appender。我们将在后面的章节中看到更多关于插件发现的内容。

**请注意，Log4j 2 不鼓励多个插件同名**。最好实现一个自定义插件，并在日志配置中使用它。

## 4.转换器插件

布局是 Log4j 2 `. `中一个强大的插件，它允许我们定义日志的输出结构。例如，我们可以使用 [`JsonLayout`](/web/20220728144649/https://www.baeldung.com/java-log-json-output) 来编写 JSON 格式的日志。

另一个这样的插件是`[PatternLayout](/web/20220728144649/https://www.baeldung.com/java-logging-intro). `。在某些情况下，应用程序希望发布像线程 id、线程名称或每个日志语句的时间戳这样的信息。`PatternLayout `插件允许我们通过一个转换模式字符串在配置中嵌入[这样的细节](https://web.archive.org/web/20220728144649/https://logging.apache.org/log4j/2.x/manual/layouts.html#PatternLayout):

```java
<Configuration status="debug" name="baeldung" packages="">
    <Appenders>
        <Console name="stdout" target="SYSTEM_OUT">
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss} %p %m%n"/>
        </Console>
    </Appenders>
</Configuration>
```

这里，`%d `是转换模式。 **Log4j 2 通过理解转换模式的`DatePatternConverter` 转换这个%d 模式，并用格式化的日期或时间戳**替换它。

现在假设一个运行在 Docker 容器中的应用程序想要打印每个日志语句的容器名。为此，我们将实现一个`DockerPatterConverter `并更改上面的配置以包含转换字符串:

```java
@Plugin(name = "DockerPatternConverter", category = PatternConverter.CATEGORY)
@ConverterKeys({"docker", "container"})
public class DockerPatternConverter extends LogEventPatternConverter {

    private DockerPatternConverter(String[] options) {
        super("Docker", "docker");
    }

    public static DockerPatternConverter newInstance(String[] options) {
        return new DockerPatternConverter(options);
    }

    @Override
    public void format(LogEvent event, StringBuilder toAppendTo) {
        toAppendTo.append(dockerContainer());
    }

    private String dockerContainer() {
        return "container-1";
    }
}
```

所以我们实现了一个类似于日期模式的自定义`DockerPatternConverter `。它将用 Docker 容器的名称替换转换模式。

这个插件类似于我们之前实现的核心插件。值得注意的是，只有一个注释与上一个插件不同。 **`@ConverterKeys `注释接受该插件**的转换模式。

因此，这个插件将把`%docker `或`%container`模式字符串转换成应用程序正在其中运行的容器名:

```java
<?xml version="1.0" encoding="UTF-8"?>
<Configuration xmlns:xi="http://www.w3.org/2001/XInclude" packages="com.baeldung" status="WARN">
    <Appenders>
        <xi:include href="log4j2-includes/console-appender_pattern-layout_colored.xml" />
        <Console name="DockerConsoleLogger" target="SYSTEM_OUT">
            <PatternLayout pattern="%pid %docker %container" />
        </Console>
    </Appenders>
    <Loggers>
        <Logger name="com.baeldung.logging.log4j2.plugins" level="INFO">
            <AppenderRef ref="DockerConsoleLogger" />
        </Logger>
    </Loggers>
</Configuration>
```

## 5.查找插件

查找插件用于在 Log4j 2 配置文件中添加动态值。它们允许应用程序将运行时值嵌入到配置文件的某些属性中。该值是通过在各种源(如文件系统、数据库等)中进行基于关键字的查找来添加的。

**一个这样的插件是`DateLookupPlugin`，它允许用应用程序**的当前系统日期替换日期模式:

```java
<RollingFile name="Rolling-File" fileName="${filename}" 
  filePattern="target/rolling1/test1-${date:MM-dd-yyyy}.%i.log.gz">
    <PatternLayout>
        <pattern>%d %p %c{1.} [%t] %m%n</pattern>
    </PatternLayout>
    <SizeBasedTriggeringPolicy size="500" />
</RollingFile>
```

在这个示例配置文件中，`RollingFileAppender `使用了一个`date `查找，其输出格式为 MM-dd-yyyy。因此，Log4j 2 将日志写入带有日期后缀的输出文件。

与其他插件类似，Log4j 2 为[查找](https://web.archive.org/web/20220728144649/https://logging.apache.org/log4j/2.x/manual/lookups.html)提供了很多来源。此外，如果需要一个新的数据源，它可以很容易地实现自定义查找:

```java
@Plugin(name = "kafka", category = StrLookup.CATEGORY)
public class KafkaLookup implements StrLookup {

    @Override
    public String lookup(String key) {
        return getFromKafka(key);
    }

    @Override
    public String lookup(LogEvent event, String key) {
        return getFromKafka(key);
    }

    private String getFromKafka(String topicName) {
        return "topic1-p1";
    }
}
```

所以`KafkaLookup`会通过查询一个 Kafka 主题来解析值。我们现在将从配置中传递主题名称:

```java
<RollingFile name="Rolling-File" fileName="${filename}" 
  filePattern="target/rolling1/test1-${kafka:topic-1}.%i.log.gz">
    <PatternLayout>
        <pattern>%d %p %c{1.} [%t] %m%n</pattern>
    </PatternLayout>
    <SizeBasedTriggeringPolicy size="500" />
</RollingFile>
```

我们用将查询`topic-1`的 Kafka lookup 替换了之前示例中的`date ` lookup。

由于 Log4j 2 只调用查找插件的默认构造函数，我们没有像在早期插件中那样实现`@PluginFactory `。

## 6.插件发现

最后，让我们了解一下 Log4j 2 是如何发现应用程序中的插件的。正如我们在上面的例子中看到的，我们给每个插件一个唯一的名字。这个名称充当一个键，Log4j 2 将这个键解析为一个插件类。

Log4j 2 执行查找以解析插件类的顺序是特定的:

1.  `log4j2-core` 库中的序列化插件列表文件。具体来说，这个 jar 中打包了一个`Log4j2Plugins.dat`来列出默认的 Log4j 2 插件
2.  OSGi 捆绑包中的类似`Log4j2Plugins.dat `文件
3.  `log4j.plugin.packages` 系统属性中以逗号分隔的包列表
4.  在[编程 Log4j 2 配置](/web/20220728144649/https://www.baeldung.com/log4j2-programmatic-config)中，我们可以调用`PluginManager.addPackages()` 方法来添加一个`list` 包名
5.  可以在 Log4j 2 配置文件中添加逗号分隔的包列表

作为先决条件，必须启用注释处理，以允许 Log4j 2 通过`@Plugin `注释中给出的`name `来解析插件。

由于 Log4j 2 使用名称来查找插件，所以上面的顺序变得很重要。例如，如果我们有两个同名的插件，Log4j 2 将首先发现被解析的插件。**因此，如果我们需要扩展 Log4j 2 `, `中现有的插件，我们必须将插件打包在一个单独的 jar 中，并将其放在** `**log4j2-core.jar**.`之前

## 7.结论

在本文中，我们查看了 Log4j 2 `.` 中插件的大类，我们讨论了即使有现有插件的详尽列表，我们可能需要为一些用例实现定制插件。

后来我们看了一些有用插件的自定义实现。此外，我们看到 Log4j 2 如何允许我们命名这些插件，并随后在配置文件中使用这个插件名称。最后，我们讨论了 Log4j 2 如何基于这个名称解析插件。

和往常一样，所有的例子都可以在 GitHub 上找到[。](https://web.archive.org/web/20220728144649/https://github.com/eugenp/tutorials/tree/master/logging-modules/log4j2)