# 弗洛格流畅测井

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/flogger-logging>

## 1.概观

在本教程中，我们将讨论 [Flogger](https://web.archive.org/web/20221207002508/https://google.github.io/flogger/) 框架，这是一个由 Google 设计的流畅的 Java 日志 API。

## 2.为什么要用弗洛格？

有了目前市场上所有的日志框架，比如 Log4j 和 Logback，为什么我们还需要另一个日志框架呢？

事实证明，与其他框架相比，Flogger 有几个优势——让我们来看看。

### 2.1.可读性

Flogger API 的流畅特性大大提高了它的可读性。

让我们看一个例子，我们希望每十次迭代记录一条消息。

对于传统的日志框架，我们会看到这样的情况:

```
int i = 0;

// ...

if (i % 10 == 0) {
    logger.info("This log shows every 10 iterations");
    i++;
}
```

但是现在，有了弗洛格，上述内容可以简化为:

```
logger.atInfo().every(10).log("This log shows every 10 iterations");
```

虽然有人会说 logger 版本的 logger 语句看起来比传统版本更加冗长，但是它确实允许更强大的功能，并最终导致更具可读性和表达性的日志语句。

### 2.2.表演

只要我们避免在记录的对象上调用`toString`,记录对象就会得到优化:

```
User user = new User();
logger.atInfo().log("The user is: %s", user);
```

如果我们记录日志，如上所示，后端有机会优化日志。另一方面，如果我们直接调用`toString `，或者连接字符串，那么这个机会就失去了:

```
logger.atInfo().log("Ths user is: %s", user.toString());
logger.atInfo().log("Ths user is: %s" + user);
```

### 2.3.展开性

Flogger 框架已经涵盖了我们对日志框架的大部分基本功能。

然而，有些情况下我们需要增加功能。在这些情况下，可以扩展 API。

**目前，这需要一个单独的支持类。**例如，我们可以通过编写一个`UserLogger `类来扩展 Flogger API:

```
logger.at(INFO).forUserId(id).withUsername(username).log("Message: %s", param);
```

这在我们想要一致地格式化消息的情况下可能是有用的。然后，`UserLogger `将提供定制方法`forUserId(String id)`和`withUsername(String username).`的实现

为此，**`UserLogger`类必须扩展`AbstractLogger `类，并为 API** 提供一个实现。如果我们看一下`FluentLogger`，它只是一个没有额外方法的记录器，因此，**我们可以从原样复制这个类开始，然后在这个基础上添加方法。**

### 2.4.效率

传统框架大量使用 varargs。这些方法需要在调用之前分配并填充一个新的`Object[]`。此外，任何传入的基本类型都必须自动装箱。

所有这些都需要额外的字节码和调用点的延迟。特别不幸的是**如果日志语句实际上没有被启用。**在循环中经常出现的调试级日志中，成本变得更加明显。Flogger 通过完全避免 varargs 来消除这些成本。

**Flogger 通过使用流畅的调用链来解决这个问题，从调用链中可以构建日志记录语句。**这允许框架只对`log`方法有少量的覆盖，因此能够避免像 varargs 和自动装箱这样的事情。**这意味着 API 可以容纳各种新特性，而不会出现组合爆炸。**

典型的日志记录框架会有这些方法:

```
level(String, Object)
level(String, Object...)
```

其中`level`可以是大约七个日志级别名称中的一个(例如`severe`)，并且有一个接受附加日志级别的规范日志方法:

```
log(Level, Object...)
```

除此之外，通常还有方法的变体，这些方法采用与日志语句相关联的原因(一个`Throwable`实例):

```
level(Throwable, String, Object)
level(Throwable, String, Object...)
```

很明显，API 将三个关注点结合到一个方法调用中:

1.  它试图指定日志级别(方法选择)
2.  试图将元数据附加到日志语句`(Throwable`原因)
3.  此外，还指定日志消息和参数。

这种方法迅速增加了满足这些独立问题所需的不同日志记录方法的数量。

我们现在可以看到为什么在链中有两个方法很重要:

```
logger.atInfo().withCause(e).log("Message: %s", arg);
```

现在让我们看看如何在我们的代码库中使用它。

## 3.属国

设置 Flogger 非常简单。我们只需要将`[flogger](https://web.archive.org/web/20221207002508/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.google.flogger%22)` 和`[flogger-system-backend](https://web.archive.org/web/20221207002508/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.google.flogger%22)`添加到我们的`pom:`中

```
<dependencies>
    <dependency>
        <groupId>com.google.flogger</groupId>
        <artifactId>flogger</artifactId>
        <version>0.4</version>
    </dependency>
    <dependency>
        <groupId>com.google.flogger</groupId>
        <artifactId>flogger-system-backend</artifactId>
        <version>0.4</version>
        <scope>runtime</scope>
    </dependency>
</dependencies>
```

建立了这些依赖关系后，我们现在可以继续探索我们可以使用的 API。

## 4.探索 Fluent API

首先，让我们为我们的记录器声明一个`static`实例:

```
private static final FluentLogger logger = FluentLogger.forEnclosingClass();
```

现在我们可以开始伐木了。我们从简单的开始:

```
int result = 45 / 3;
logger.atInfo().log("The result is %d", result);
```

日志消息可以使用任何 Java 的`printf`格式说明符，比如 `%s, %d`或 `%016x`。

### 4.1.避免在日志站点工作

Flogger 创建者建议我们避免在日志站点工作。

假设我们有以下长期运行的方法来总结组件的当前状态:

```
public static String collectSummaries() {
    longRunningProcess();
    int items = 110;
    int s = 30;
    return String.format("%d seconds elapsed so far. %d items pending processing", s, items);
}
```

很容易在我们的日志语句中直接调用`collectSummaries`:

```
logger.atFine().log("stats=%s", collectSummaries());
```

**不管配置的日志级别或速率限制如何，现在每次都会调用`collectSummaries`方法。**

使禁用日志语句的成本几乎为零是日志框架的核心。反过来，这意味着它们中的更多部分可以原封不动地保留在代码中，而不会造成伤害。像我们刚才所做的那样编写日志语句会失去这个优势。

**相反，我们应该使用`LazyArgs.lazy `方法**:

```
logger.atFine().log("stats=%s", LazyArgs.lazy(() -> collectSummaries()));
```

**现在，日志站点**几乎不做任何工作，只是为 lambda 表达式创建实例。 **Flogger 只有打算真正记录消息时才会评估这个 lambda。**

虽然允许使用`isEnabled`来保护日志语句:

```
if (logger.atFine().isEnabled()) {
    logger.atFine().log("summaries=%s", collectSummaries());
}
```

这是不必要的，我们应该避免这样做，因为 Flogger 为我们做了这些检查。这种方法也只能按级别保护日志语句，对速率受限的日志语句没有帮助。

### 4.2.处理异常

异常呢，我们如何处理它们？

Flogger 附带了一个`withStackTrace`方法，我们可以用它来记录一个`Throwable` 实例:

```
try {
    int result = 45 / 0;
} catch (RuntimeException re) {
    logger.atInfo().withStackTrace(StackSize.FULL).withCause(re).log("Message");
}
```

其中`withStackTrace`将具有常量值`SMALL, MEDIUM, LARGE` 或`FULL`的`StackSize`枚举作为参数。由`withStackTrace()`生成的堆栈跟踪将在默认的`java.util.logging`后端显示为`LogSiteStackTrace`异常。但是其他后端可能会选择不同的处理方式。

### 4.3.日志配置和级别

到目前为止，我们已经在大多数例子中使用了`logger.atInfo`,但是 Flogger 确实支持许多其他级别。我们将研究这些，但是首先，让我们介绍如何配置日志选项。

为了配置日志记录，我们使用了`LoggerConfig`类。

例如，当我们想要将日志记录级别设置为`FINE`时:

```
LoggerConfig.of(logger).setLevel(Level.FINE);
```

Flogger 支持各种日志记录级别:

```
logger.atInfo().log("Info Message");
logger.atWarning().log("Warning Message");
logger.atSevere().log("Severe Message");
logger.atFine().log("Fine Message");
logger.atFiner().log("Finer Message");
logger.atFinest().log("Finest Message");
logger.atConfig().log("Config Message");
```

### 4.4.限速

限速的问题怎么样？我们如何处理不想记录每次迭代的情况呢？

**弗洛格用`every(int n) `方法**来解救我们:

```
IntStream.range(0, 100).forEach(value -> {
    logger.atInfo().every(40).log("This log shows every 40 iterations => %d", value);
});
```

当我们运行上面的代码时，我们得到以下输出:

```
Sep 18, 2019 5:04:02 PM com.baeldung.flogger.FloggerUnitTest lambda$givenAnInterval_shouldLogAfterEveryTInterval$0
INFO: This log shows every 40 iterations => 0 [CONTEXT ratelimit_count=40 ]
Sep 18, 2019 5:04:02 PM com.baeldung.flogger.FloggerUnitTest lambda$givenAnInterval_shouldLogAfterEveryTInterval$0
INFO: This log shows every 40 iterations => 40 [CONTEXT ratelimit_count=40 ]
Sep 18, 2019 5:04:02 PM com.baeldung.flogger.FloggerUnitTest lambda$givenAnInterval_shouldLogAfterEveryTInterval$0
INFO: This log shows every 40 iterations => 80 [CONTEXT ratelimit_count=40 ]
```

如果我们想每 10 秒记录一次呢？那么，**我们可以用`atMostEvery(int n, TimeUnit unit)`** :

```
IntStream.range(0, 1_000_0000).forEach(value -> {
    logger.atInfo().atMostEvery(10, TimeUnit.SECONDS).log("This log shows [every 10 seconds] => %d", value);
});
```

这样，现在的结果变成了:

```
Sep 18, 2019 5:08:06 PM com.baeldung.flogger.FloggerUnitTest lambda$givenATimeInterval_shouldLogAfterEveryTimeInterval$1
INFO: This log shows [every 10 seconds] => 0 [CONTEXT ratelimit_period="10 SECONDS" ]
Sep 18, 2019 5:08:16 PM com.baeldung.flogger.FloggerUnitTest lambda$givenATimeInterval_shouldLogAfterEveryTimeInterval$1
INFO: This log shows [every 10 seconds] => 3545373 [CONTEXT ratelimit_period="10 SECONDS [skipped: 3545372]" ]
Sep 18, 2019 5:08:26 PM com.baeldung.flogger.FloggerUnitTest lambda$givenATimeInterval_shouldLogAfterEveryTimeInterval$1
INFO: This log shows [every 10 seconds] => 7236301 [CONTEXT ratelimit_period="10 SECONDS [skipped: 3690927]" ]
```

## 5.将 Flogger 与其他后端一起使用

那么，如果我们想将 Flogger 添加到我们现有的应用程序中，比如说已经使用了 Slf4j 或 Log4jT5 的应用程序，该怎么办呢？在我们希望利用现有配置的情况下，这可能很有用。正如我们将看到的，Flogger 支持多个后端。

### 5.1。Slf4j 打蛋器

配置 Slf4j 后端很简单。首先，我们需要将`[flogger-slf4j-backend](https://web.archive.org/web/20221207002508/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.google.flogger%22)`依赖项添加到我们的`pom`中:

```
<dependency>
    <groupId>com.google.flogger</groupId>
    <artifactId>flogger-slf4j-backend</artifactId>
    <version>0.4</version>
</dependency>
```

接下来，我们需要告诉 Flogger，我们希望使用一个不同于默认后端的后端。我们通过系统属性注册一个 Flogger 工厂来实现这一点:

```
System.setProperty(
  "flogger.backend_factory", "com.google.common.flogger.backend.slf4j.Slf4jBackendFactory#getInstance");
```

现在我们的应用程序将使用现有的配置。

### 5.2。带 Log4j 的 Flogger】

我们遵循类似的步骤来配置 Log4j 后端。让我们将`[flogger-log4j-backend](https://web.archive.org/web/20221207002508/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A"com.google.flogger")`依赖项添加到`pom`中:

```
<dependency>
    <groupId>com.google.flogger</groupId>
    <artifactId>flogger-log4j-backend</artifactId>
    <version>0.4</version>
    <exclusions>
        <exclusion>
            <groupId>com.sun.jmx</groupId>
            <artifactId>jmxri</artifactId>
        </exclusion>
        <exclusion>
            <groupId>com.sun.jdmk</groupId>
            <artifactId>jmxtools</artifactId>
        </exclusion>
        <exclusion>
            <groupId>javax.jms</groupId>
            <artifactId>jms</artifactId>
        </exclusion>
    </exclusions>
</dependency>

<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
<dependency>
    <groupId>log4j</groupId>
    <artifactId>apache-log4j-extras</artifactId>
    <version>1.2.17</version>
</dependency>
```

我们还需要为 Log4j 注册一个 Flogger 后端工厂:

```
System.setProperty(
  "flogger.backend_factory", "com.google.common.flogger.backend.log4j.Log4jBackendFactory#getInstance");
```

就这样，我们的应用程序现在设置为使用现有的 Log4j 配置！

## 6.结论

在本教程中，我们看到了如何使用 Flogger 框架作为传统日志框架的替代方案。我们已经看到了一些强大的特性，在使用这个框架时我们可以从中受益。

我们还看到了如何通过注册 Slf4j 和 Log4j 等不同的后端来利用我们现有的配置。

像往常一样，本教程的源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221207002508/https://github.com/eugenp/tutorials/tree/master/logging-modules/flogger)