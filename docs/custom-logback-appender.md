# 创建一个定制的日志回溯 Appender

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/custom-logback-appender>

## 1。简介

在本文中，我们将探索如何创建一个定制的日志回溯 appender。如果你在找 Java 登录入门，请看看[这篇文章](/web/20220625233345/https://www.baeldung.com/java-logging-intro)。

Logback 附带了许多内置的附加器，可以写入标准输出、文件系统或数据库。这个框架架构的美妙之处在于它的模块化，这意味着我们可以轻松地对它进行定制。

在本教程中，我们将关注`logback-classic`，它需要以下 Maven 依赖关系:

```java
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.2.6</version>
</dependency>
```

该依赖关系的最新版本可在 [Maven Central](https://web.archive.org/web/20220625233345/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22ch.qos.logback%22%20AND%20a%3A%22logback-classic%22) 上获得。

## 2。基本回溯追加器

Logback 提供了基类，我们可以扩展这些基类来创建定制的 appender。

`Appender` 是所有 appenders 必须实现的通用接口。泛型类型要么是*ILoggingEvent*要么是*access event*，这取决于我们是使用*log back-classic*还是log back-access。

我们的自定义 appender 应该扩展***Appender base*****或*****unsynchronized Appender base***，它们都实现了*Appender*并处理过滤器和状态消息等功能。

*AppenderBase* 是线程安全的；*unsynchronized appenderbase*子类负责管理自己的线程安全。

正如*console Pender*和*file Appender*都扩展了*OutputStream appender*并调用了超方法*【setOutputStream()*****自定义 appender 应该子类****

 **## 3。自定义附加器

对于我们的自定义示例，我们将创建一个名为`MapAppender`的玩具附加器。这个 appender 将把所有的日志事件插入到一个`Concurrent` `HashMap`中，并带有键的时间戳。首先，我们将子类化`AppenderBase`并使用`ILoggingEvent`作为泛型类型:

```java
public class MapAppender extends AppenderBase<ILoggingEvent> {

    private ConcurrentMap<String, ILoggingEvent> eventMap 
      = new ConcurrentHashMap<>();

    @Override
    protected void append(ILoggingEvent event) {
        eventMap.put(System.currentTimeMillis(), event);
    }

    public Map<String, ILoggingEvent> getEventMap() {
        return eventMap;
    }
}
```

接下来，为了使`MapAppender`能够开始接收日志事件，让我们在配置文件`logback.xml`中将它添加为一个追加器:

```java
<configuration>
    <appender name="map" class="com.baeldung.logback.MapAppender"/>
    <root level="info">
        <appender-ref ref="map"/>
    </root>
</configuration>
```

## 4。设置属性

Logback 使用 JavaBeans 自省来分析 appender 上的属性集。我们的定制 appender 将需要 getter 和 setter 方法来允许内省器找到并设置这些属性。

让我们给`MapAppender`添加一个属性，为`eventMap` 的键添加一个前缀:

```java
public class MapAppender extends AppenderBase<ILoggingEvent> {

    //...

    private String prefix;

    @Override
    protected void append(ILoggingEvent event) {
        eventMap.put(prefix + System.currentTimeMillis(), event);
    }

    public String getPrefix() {
        return prefix;
    }

    public void setPrefix(String prefix) {
        this.prefix = prefix;
    }

    //...

}
```

接下来，在我们的配置中添加一个属性来设置这个前缀:

```java
<configuration debug="true">

    <appender name="map" class="com.baeldung.logback.MapAppender">
        <prefix>test</prefix>
    </appender>

    //...

</configuration>
```

## 5。错误处理

为了在创建和配置自定义 appender 的过程中处理错误，我们可以使用从`AppenderBase`继承的方法。

例如，当前缀属性为 null 或空字符串时，`MapAppender` 可以调用`addError()`并提前返回:

```java
public class MapAppender extends AppenderBase<ILoggingEvent> {

    //...

    @Override
    protected void append(final ILoggingEvent event) {
        if (prefix == null || "".equals(prefix)) {
            addError("Prefix is not set for MapAppender.");
            return;
        }

        eventMap.put(prefix + System.currentTimeMillis(), event);
    }

    //...

}
```

当调试标志在我们的配置中打开时，我们将在控制台中看到一个错误，警告我们 prefix 属性尚未设置:

```java
<configuration debug="true">

    //...

</configuration>
```

## 6。结论

在这个快速教程中，我们重点介绍了如何为 Logback 实现我们的定制 appender。

像往常一样，这个例子可以在 Github 上找到[。](https://web.archive.org/web/20220625233345/https://github.com/eugenp/tutorials/tree/master/logging-modules/logback)**