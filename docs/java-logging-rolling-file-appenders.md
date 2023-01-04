# 滚动文件附加器指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-logging-rolling-file-appenders>

## 1。概述

虽然日志文件通常传递有用的信息，但随着时间的推移，它们会自然变大。如果任由其无限增长，它们的规模可能会成为一个问题。

日志库使用**滚动文件附加器来解决这个问题，滚动文件附加器自动“滚动”或归档当前日志文件，并在某些预定义的条件发生时在新文件**中恢复日志记录，从而防止不必要的停机。

在本教程中，我们将学习如何在一些最广泛使用的日志库中配置滚动文件附加器:Log4j、Log4j2 和 Slf4j。

我们将演示如何基于大小、日期/时间以及大小和日期/时间的组合来滚动日志文件。我们还将探索如何配置每个库来自动压缩并在以后删除旧的日志文件，从而使我们免于编写繁琐的内务代码。

## 2。我们的示例应用程序

让我们从记录一些消息的示例应用程序开始。这段代码是基于 Log4j 的，但是我们可以很容易地修改它，使其适用于 Log4j2 或 Slf4j:

```java
import org.apache.log4j.Logger;

public class Log4jRollingExample {

    private static Logger logger = Logger.getLogger(Log4jRollingExample.class);

    public static void main(String[] args) throws InterruptedException {
        for(int i = 0; i < 2000; i++) {
            logger.info("This is the " + i + " time I say 'Hello World'.");
            Thread.sleep(100);
        }
    }
}
```

应用相当幼稚；它在一个循环中写一些消息，在迭代之间有一个短暂的延迟。由于要运行 2，000 个循环，并且每个循环中有 100 毫秒的暂停，应用程序应该需要三分多钟才能完成。

我们将使用这个示例来演示不同类型的滚动文件附加器的几个特性。

## 3。Log4j 中的滚动文件追加器

### 3.1。Maven 依赖关系

首先，为了在我们的应用程序中使用 Log4j，我们将把这个依赖项添加到项目的 *pom.xml* 文件中:

```java
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency> 
```

对于我们将在接下来的例子中使用的由`apache-log-extras`提供的附加 appenders，我们将添加以下依赖项，确保使用我们为 Log4j 声明的相同版本，以确保完全兼容:

```java
<dependency>
    <groupId>log4j</groupId>
    <artifactId>apache-log4j-extras</artifactId>
    <version>1.2.17</version>
</dependency> 
```

我们可以在 Maven Central 上找到最新发布的 [Log4j](https://web.archive.org/web/20220812051731/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22log4j%22%20AND%20a%3A%22log4j%22) 和 [Apache Log4j Extras](https://web.archive.org/web/20220812051731/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22log4j%22%20AND%20a%3A%22apache-log4j-extras%22) 。

### 3.2。基于文件大小的滚动

在 Log4j 中，和在其他日志库中一样，文件滚动被委托给 appender。让我们看看 Log4j 中基于文件大小滚动的滚动文件附加器的配置:

```java
<appender name="roll-by-size" class="org.apache.log4j.RollingFileAppender">
    <param name="file" value="target/log4j/roll-by-size/app.log" />
    <param name="MaxFileSize" value="5KB" />
    <param name="MaxBackupIndex" value="2" />
        <layout class="org.apache.log4j.PatternLayout">
            <param name="ConversionPattern" value="%d{yyyy-MM-dd HH:mm:ss} %-5p %m%n" />
        </layout>
</appender>
```

这里，我们使用`MaxFileSize`参数将 Log4j 配置为当日志文件的大小达到 5KB 时滚动日志文件。我们还使用`MaxBackupIndex`参数指示 Log4j 最多保留两个滚动日志文件。

当我们运行示例应用程序时，我们获得以下文件:

```java
27/11/2016  10:28    138 app.log
27/11/2016  10:28  5.281 app.log.1
27/11/2016  10:28  5.281 app.log.2 
```

发生了什么事？Log4j 开始写入`app.log`文件。当文件大小超过 5KB 限制时，Log4j 将`app.log`移动到`app.log.1`，创建一个新的空`app.log`，并继续向`app.log`写入新的日志消息。

然后，在新的`app.log`超过 5KB 限制后，重复这个滚动过程。这一次，`app.log.1` 被移到了`app.log.2,` ，为另一个新的空`app.log`腾出了空间。

这个滚动过程在运行过程中重复了几次，但是因为我们配置了 appender 来最多保存两个滚动的文件，所以没有名为`app.log.3`的文件。

所以我们解决了最初的一个问题，因为现在我们可以对生成的日志文件的大小设置一个限制。

当我们检查`app.log.2`的第一行时，它包含与第 700 次迭代相关的消息，这意味着所有以前的日志消息都丢失了:

```java
2016-11-27 10:28:34 INFO  This is the 700 time I say 'Hello World'. 
```

现在，让我们看看是否可以设计一个更适合生产环境的设置，在生产环境中，丢失日志消息被认为不是最好的方法。

为此，我们将使用其他更强大、更灵活、更可配置的 Log4j appenders，它们包含在一个名为`apache-log4j-extras`的专用包中。

这个工件中包含的附加器提供了许多选项来微调日志滚动，并且它们引入了不同的概念`triggering policy`和`rolling policy`。`triggering policy`描述了何时应该进行滚动，而`rolling policy`描述了应该如何进行滚动。这两个概念是滚动日志文件的关键，其他库也或多或少地使用它们。

### 3.3。自动压缩滚动

让我们回到 Log4j 示例，通过添加自动压缩滚动文件以节省空间来改进我们的设置:

```java
<appender name="roll-by-size" class="org.apache.log4j.rolling.RollingFileAppender">
    <rollingPolicy class="org.apache.log4j.rolling.FixedWindowRollingPolicy">
        <param name="ActiveFileName" value="target/log4j/roll-by-size/app.log" />
        <param name="FileNamePattern" value="target/log4j/roll-by-size/app.%i.log.gz" />
    <param name="MinIndex" value="7" />
    <param name="MaxIndex" value="17" /> 
    </rollingPolicy>
    <triggeringPolicy class="org.apache.log4j.rolling.SizeBasedTriggeringPolicy">
        <param name="MaxFileSize" value="5120" />
    </triggeringPolicy>
    <layout class="org.apache.log4j.PatternLayout">
        <param name="ConversionPattern" value="%d{yyyy-MM-dd HH:mm:ss} %-5p %m%n" />
    </layout>
</appender>
```

使用`triggering policy`元素，我们声明当日志超过 5120 字节大小时应该进行滚动。

在`rolling policy`标签`,` 中，`ActiveFileName`参数指定包含最新消息的主日志文件的路径，而`FileNamePattern`参数指定描述滚动文件路径的模板。请注意，这确实是一种模式，因为特殊的占位符`%i`将被替换为滚动文件的索引。

我们还要注意的是，`FileNamePattern`以一个“`.gz”`扩展名结尾。每当我们使用与受支持的压缩格式相关联的扩展名时，我们都会压缩旧的滚动文件，而无需我们付出任何额外的努力。

现在，当我们运行应用程序时，我们获得了一组不同的日志文件:

```java
03/12/2016 19:24 88 app.1.log.gz
...
03/12/2016 19:26 88 app.2.log.gz
03/12/2016 19:26 88 app.3.log.gz
03/12/2016 19:27 70 app.current.log 
```

文件`app.current.log`是最后一个日志出现的地方。当以前的日志的大小达到设置的限制时，会对其进行滚动和压缩。

### 3.4。基于日期和时间的滚动

在其他场景中，我们可能希望将 Log4j 配置为基于日志消息的日期和时间而不是文件的大小来滚动文件。例如，在 web 应用程序中，我们可能希望在同一天在同一个日志文件中发布所有日志消息。

为此，我们可以使用`TimeBasedRollingPolicy`。使用此策略，必须为包含时间相关占位符的日志文件路径指定一个模板。每次发出日志消息时，appender 都会验证生成的日志路径。如果它不同于最后使用的路径，那么将发生滚动。这里有一个配置这种 appender 的简单例子:

```java
<appender name="roll-by-time"
    class="org.apache.log4j.rolling.RollingFileAppender">
    <rollingPolicy class="org.apache.log4j.rolling.TimeBasedRollingPolicy">
        <param name="FileNamePattern" value="target/log4j/roll-by-time/app.%d{HH-mm}.log.gz" />
    </rollingPolicy>
    <layout class="org.apache.log4j.PatternLayout">
        <param name="ConversionPattern" value="%d{yyyy-MM-dd HH:mm:ss} %-5p - %m%n" />
    </layout>
</appender> 
```

### 3.5。基于尺寸和时间的滚动

结合`SizeBasedTriggeringPolicy`和`TimeBasedRollingPolicy,`我们可以获得一个基于日期/时间滚动的 appender，当文件的大小达到设定的限制时，它也基于大小滚动:

```java
<appender name="roll-by-time-and-size" class="org.apache.log4j.rolling.RollingFileAppender">
    <rollingPolicy class="org.apache.log4j.rolling.TimeBasedRollingPolicy">
        <param name="ActiveFileName" value="log4j/roll-by-time-and-size/app.log" />
        <param name="FileNamePattern" value="log4j/roll-by-time-and-size/app.%d{HH-mm}.%i.log.gz" />
    </rollingPolicy>
    <triggeringPolicy
        class="org.apache.log4j.rolling.SizeBasedTriggeringPolicy">
        <param name="MaxFileSize" value="100" />
    </triggeringPolicy>
    <layout class="org.apache.log4j.PatternLayout">
        <param name="ConversionPattern" value="%d{yyyy-MM-dd HH:mm:ss} %-5p - %m%n" />
    </layout>
</appender>
```

当我们用这个设置运行我们的应用程序时，我们获得以下日志文件:

```java
03/12/2016 19:25 234 app.19-25.1481393432120.log.gz
03/12/2016 19:25 234 app.19-25.1481393438939.log.gz
03/12/2016 19:26 244 app.19-26.1481393441940.log.gz
03/12/2016 19:26 240 app.19-26.1481393449152.log.gz
03/12/2016 19:26 3.528 app.19-26.1481393470902.log
```

文件 `app.19-26.1481393470902.log`是当前日志记录发生的地方。正如我们所看到的，19:25 到 19:26 之间的所有日志都存储在多个压缩日志文件中，文件名以“`app.19-25″.`开头，占位符`“%i”`被一个不断增加的数字所取代。

## 4。Log4j2 中的滚动文件附加器

### 4.1。Maven 依赖关系

要使用 Log4j2 作为我们首选的日志库，我们需要用以下依赖项更新我们项目的 POM:

```java
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.7</version>
</dependency>
```

和往常一样，我们可以在 Maven Central 上找到[的最新版本](https://web.archive.org/web/20220812051731/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.logging.log4j%22%20AND%20a%3A%22log4j-core%22)。

### 4.2。基于文件大小的滚动

让我们将示例应用程序改为使用 Log4j2 日志库。我们将根据`log4j2.xml`配置文件中日志文件的大小设置文件滚动:

```java
<RollingFile 
  name="roll-by-size" 
  fileName="target/log4j2/roll-by-size/app.log"
  filePattern="target/log4j2/roll-by-size/app.%i.log.gz" 
  ignoreExceptions="false">
    <PatternLayout>
        <Pattern>%d{yyyy-MM-dd HH:mm:ss} %p %m%n</Pattern>
    </PatternLayout>
    <Policies>
        <OnStartupTriggeringPolicy />
        <SizeBasedTriggeringPolicy size="5 KB" />
    </Policies>
</RollingFile> 
```

在`Policies`标签中，我们指定了我们想要应用的所有触发策略。`OnStartupTriggeringPolicy`每次应用程序启动时触发一次滚动，这对独立应用程序很有用。我们还指定了一个`SizeBasedTriggeringPolicy,`,声明每当日志文件达到 5KB 时就应该进行一次回滚。

### 4.3。基于日期和时间的滚动

使用 Log4j2 提供的策略，我们将设置一个 appender 来根据时间滚动和压缩日志文件:

```java
<RollingFile name="roll-by-time" 
  fileName="target/log4j2/roll-by-time/app.log"
  filePattern="target/log4j2/roll-by-time/app.%d{MM-dd-yyyy-HH-mm}.log.gz"
  ignoreExceptions="false">
    <PatternLayout>
        <Pattern>%d{yyyy-MM-dd HH:mm:ss} %p %m%n</Pattern>
    </PatternLayout>
    <TimeBasedTriggeringPolicy />
</RollingFile> 
```

这里的关键是`TimeBasedTriggeringPolicy,` ，它允许我们在滚动文件名的模板中使用与时间相关的占位符。注意，由于我们只需要一个触发策略，我们不需要像前面的例子那样使用`Policies`标签。

### 4.4。基于尺寸和时间的滚动

如前所述，更有说服力的方案是基于时间和大小来滚动和压缩日志文件。下面是我们如何为这个任务设置 Log4j2 的一个例子:

```java
<RollingFile name="roll-by-time-and-size" 
  fileName="target/log4j2/roll-by-time-and-size/app.log"
  filePattern="target/log4j2/roll-by-time-and-size/app.%d{MM-dd-yyyy-HH-mm}.%i.log.gz" 
  ignoreExceptions="false">
    <PatternLayout>
        <Pattern>%d{yyyy-MM-dd HH:mm:ss} %p %m%n</Pattern>
    </PatternLayout>
    <Policies>
        <OnStartupTriggeringPolicy />
        <SizeBasedTriggeringPolicy size="5 KB" />
        <TimeBasedTriggeringPolicy />
    </Policies>
    <DefaultRolloverStrategy>
        <Delete basePath="${baseDir}" maxDepth="2">
            <IfFileName glob="target/log4j2/roll-by-time-and-size/app.*.log.gz" />
            <IfLastModified age="20d" />
        </Delete>
    </DefaultRolloverStrategy>
</RollingFile>
```

在这种配置下，我们声明应该根据时间和大小进行滚动。appender 能够理解我们所指的时间间隔，因为文件名使用了模式“`app.%d{MM-dd-yyyy-HH-mm}.%i.log.gz”`”，它隐式地设置每分钟发生一次滚动，并压缩滚动的文件。

我们还添加了一个`DefaultRolloverStrategy`来删除符合特定标准的旧滚动文件。我们将我们的配置为删除超过 20 天的符合给定模式的文件。

### 4.5。Maven 依赖关系

要使用 Log4j2 作为我们首选的日志库，我们需要用以下依赖项更新我们项目的 POM:

```java
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.7</version>
</dependency>
```

和往常一样，我们可以在 Maven Central 上找到[的最新版本](https://web.archive.org/web/20220812051731/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.logging.log4j%22%20AND%20a%3A%22log4j-core%22)。

## 5。Slf4j 中的滚动文件追加器

### 5.1。Maven 依赖关系

当我们想要使用 Slf4j2 和 Logback 后端作为我们的日志库时，我们将把这个依赖项添加到我们的`pom.xml`:

```java
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.2.6</version>
</dependency>
```

和往常一样，我们可以在 Maven Central 上找到[的最新版本](https://web.archive.org/web/20220812051731/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22ch.qos.logback%22%20AND%20a%3A%22logback-classic%22)。

### 5.2。基于文件大小的滚动

现在让我们看看如何使用 Slf4j 的默认后端`Logback`。我们将在配置文件`logback.xml`中设置文件滚动，该文件位于应用程序的类路径中:

```java
<appender name="roll-by-size" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>target/slf4j/roll-by-size/app.log</file>
    <rollingPolicy class="ch.qos.logback.core.rolling.FixedWindowRollingPolicy">
        <fileNamePattern>target/slf4j/roll-by-size/app.%i.log.zip</fileNamePattern>
        <minIndex>1</minIndex>
        <maxIndex>3</maxIndex>
        <totalSizeCap>1MB</totalSizeCap>
    </rollingPolicy>
    <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
       <maxFileSize>5KB</maxFileSize>
    </triggeringPolicy>
    <encoder>
        <pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>
    </encoder>
</appender> 
```

我们再次遇到滚动政策的概念。基本机制与 Log4j 和 Log4j2 使用的相同。`FixedWindowRollingPolicy`允许我们在滚动文件的名称模式中使用索引占位符。

当日志文件的大小超过配置的限制时，会分配一个新文件。旧内容被存储为列表中的第一个文件，将现有内容进一步移动一个位置。

### 5.3。基于时间的滚动

在 Slf4j 中，我们可以使用提供的`TimeBasedRollingPolicy`基于时间滚动日志文件。此策略允许我们使用与时间和日期相关的占位符来指定滚动文件的模板名称:

```java
<appender name="roll-by-time" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>target/slf4j/roll-by-time/app.log</file>
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
        <fileNamePattern>target/slf4j/roll-by-time/app.%d{yyyy-MM-dd-HH-mm}.log.zip
        </fileNamePattern>
        <maxHistory>20</maxHistory>
        <totalSizeCap>1MB</totalSizeCap>
    </rollingPolicy>
    <encoder>
        <pattern>%d{yyyy-MM-dd HH:mm:ss} %p %m%n</pattern>
    </encoder>
</appender>
```

### 5.4。基于尺寸和时间的滚动

如果我们需要基于时间和大小滚动文件，我们可以使用提供的`SizeAndTimeBasedRollingPolicy`。使用此策略时，我们必须指定一个与时间相关的占位符和一个索引占位符。

每当特定时间间隔内日志文件的大小超过配置的大小限制时，就会创建另一个日志文件，该日志文件具有与时间相关的占位符相同的值，但具有递增的索引:

```java
<appender name="roll-by-time-and-size"
    class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>target/slf4j/roll-by-time-and-size/app.log</file>
    <rollingPolicy
      class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
        <fileNamePattern>
            target/slf4j/roll-by-time-and-size/app.%d{yyyy-MM-dd-mm}.%i.log.zip
        </fileNamePattern>
        <maxFileSize>5KB</maxFileSize>
        <maxHistory>20</maxHistory>
        <totalSizeCap>1MB</totalSizeCap>
    </rollingPolicy>
    <encoder>
        <pattern>%d{yyyy-MM-dd HH:mm:ss} %p %m%n</pattern>
    </encoder>
</appender>
```

## 6。结论

在本文中，我们了解到利用日志库来滚动文件可以减轻我们手动管理日志文件的负担。相反，我们可以专注于业务逻辑的发展。滚动文件附加器是一个有价值的工具，应该放在每个开发人员的工具箱中。

和往常一样，可以在 GitHub 上获得这些源代码[，在这里，本文中给出的示例应用程序被配置为使用几种不同的滚动设置来记录日志，以便让我们找到一个好的基本配置，进一步调整以满足我们的需求。](https://web.archive.org/web/20220812051731/https://github.com/eugenp/tutorials/tree/master/logging-modules/log4j)