# 创建定制的 Log4j2 Appender

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/log4j2-custom-appender>

## 1.介绍

在本教程中，我们将学习如何创建一个定制的 Log4j2 appender。如果你在找 Log4j2 的介绍，请看看[这篇文章](/web/20221208143816/https://www.baeldung.com/log4j2-appenders-layouts-filters)。

**Log4j2 附带了许多内置的附加器**，这些附加器可用于各种目的，如记录到文件、数据库、套接字或 NoSQL 数据库。

但是，根据应用程序的需求，可能需要一个定制的 appender。

Log4j2 是 Log4j 的升级版本，在 Log4j 的基础上有显著的改进。因此，我们将使用 Log4j2 框架来演示定制 appender 的创建。

## 2.Maven 设置

首先，我们需要`pom.xml`中的`log4j-core`依赖关系:

```java
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.11.0</version>
</dependency>
```

最新版本`log4j-core` 可以在[这里](https://web.archive.org/web/20221208143816/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.logging.log4j%22%20AND%20a%3A%22log4j-core%22)找到。

## 3.定制附加器

有两种方法可以实现我们的自定义 appender。**首先是通过实现`Appender`接口，其次是通过扩展`AbstractAppender `类。**第二种方法提供了一种简单的方法来实现我们自己的定制 appender，这就是我们将要使用的方法。

对于这个例子，我们将创建一个`MapAppender`。我们将捕获日志事件，并将它们存储在一个带有键时间戳的`Concurrent` `HashMap `中。

下面是我们如何创建`MapAppender:`

```java
@Plugin(
  name = "MapAppender", 
  category = Core.CATEGORY_NAME, 
  elementType = Appender.ELEMENT_TYPE)
public class MapAppender extends AbstractAppender {

    private ConcurrentMap<String, LogEvent> eventMap = new ConcurrentHashMap<>();

    protected MapAppender(String name, Filter filter) {
        super(name, filter, null);
    }

    @PluginFactory
    public static MapAppender createAppender(
      @PluginAttribute("name") String name, 
      @PluginElement("Filter") Filter filter) {
        return new MapAppender(name, filter);
    }

    @Override
    public void append(LogEvent event) {
        eventMap.put(Instant.now().toString(), event);
    }
}
```

我们已经用 `@Plugin`注释对该类进行了注释，这表明我们的 appender 是一个插件。

插件的`name`表示我们将在配置中提供的名称，以使用这个 appender。`category`指定了我们放置插件的类别。`elementType`是附录。

我们还需要一个工厂方法来创建 appender。我们的`createAppender`方法服务于这个目的，并且用`@PluginFactory`注释进行了注释。

在这里，我们通过调用受保护的构造函数来初始化我们的 appender，并且我们将`layout`作为 null 传递，因为我们不打算在配置文件中提供任何布局，并且我们期望框架解析默认布局。

接下来，**我们覆盖了`append`方法，该方法具有处理`LogEvent`** 的实际逻辑。在我们的例子中，`append`方法将`LogEvent`放入我们的`eventMap. `

## 4.配置

现在我们已经有了我们的`MapAppender `,我们需要一个`lo4j2.xml `配置文件来使用这个 appender 进行日志记录。

下面是我们如何在`log4j2.xml`文件中定义配置部分:

```java
<Configuration xmlns:xi="http://www.w3.org/2001/XInclude" packages="com.baeldung" status="WARN">
```

请注意，packages 属性应该引用包含定制 appender 的包。

接下来，在我们的 appender 部分，我们定义 appender。下面是我们如何在配置中将自定义 appender 添加到 appender 列表中:

```java
<MapAppender name="MapAppender" />
```

最后一部分是在 Loggers 部分实际使用 appender。对于我们的实现，我们使用`MapAppender`作为根日志记录器，并在根部分定义它。

这是如何做到的:

```java
<Root level="DEBUG">
    <AppenderRef ref="MapAppender" />
</Root>
```

## 5.错误处理

为了在记录事件时处理错误，我们可以使用从`AbstractAppender.`继承的`error`方法

例如，如果我们不想记录日志级别低于`WARN.`的事件

我们可以使用`AbstractAppender`的`error `方法来记录错误消息。我们班是这样做的:

```java
public void append(LogEvent event) {
    if (event.getLevel().isLessSpecificThan(Level.WARN)) {
        error("Unable to log less than WARN level.");
        return;
    }
    eventMap.put(Instant.now().toString(), event);
}
```

观察我们的`append`方法现在有了怎样的变化。我们检查`event's`水平是否大于`WARN`，如果小于`WARN`，我们就提前返回。

## 6.结论

在本文中，我们看到了如何为 Log4j2 实现一个定制的 appender。

虽然有许多内置的方法可以通过使用 Log4j2 提供的 appender 来记录我们的数据，但是我们在这个框架中也有一些工具，使我们能够根据应用程序的需要创建自己的 appender。

像往常一样，这个例子可以在 Github 上找到[。](https://web.archive.org/web/20221208143816/https://github.com/eugenp/tutorials/tree/master/logging-modules/log4j2)