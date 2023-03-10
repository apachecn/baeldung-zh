# 意识简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/awaitility-testing>

## 1。简介

异步系统的一个常见问题是，很难为它们编写可读的测试，这些测试侧重于业务逻辑，并且没有被同步、超时和并发控制所污染。

在这篇文章中，我们将看一看**[awaity](https://web.archive.org/web/20221012100327/http://www.awaitility.org/)——一个为异步系统测试**提供简单的领域特定语言(DSL)的库。

有了易用性，我们可以用一种易读的 DSL 来表达我们对系统的期望。

## 2。依赖性

我们需要将可用性依赖添加到我们的`pom.xml.`

对于大多数用例来说, `awaitility`库就足够了。如果我们想使用基于代理的条件`,`，我们还需要提供`awaitility-proxy` 库:

```java
<dependency>
    <groupId>org.awaitility</groupId>
    <artifactId>awaitility</artifactId>
    <version>3.0.0</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.awaitility</groupId>
    <artifactId>awaitility-proxy</artifactId>
    <version>3.0.0</version>
    <scope>test</scope>
</dependency>
```

你可以在 Maven Central 上找到最新版本的 [`awaitility`](https://web.archive.org/web/20221012100327/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.awaitility%22%20AND%20a%3A%22awaitility%22) 和`[awaitility-proxy](https://web.archive.org/web/20221012100327/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.awaitility%22%20AND%20a%3A%22awaitility-proxy%22)`库。

## 3。创建异步服务

让我们编写一个简单的异步服务并测试它:

```java
public class AsyncService {
    private final int DELAY = 1000;
    private final int INIT_DELAY = 2000;

    private AtomicLong value = new AtomicLong(0);
    private Executor executor = Executors.newFixedThreadPool(4);
    private volatile boolean initialized = false;

    void initialize() {
        executor.execute(() -> {
            sleep(INIT_DELAY);
            initialized = true;
        });
    }

    boolean isInitialized() {
        return initialized;
    }

    void addValue(long val) {
        throwIfNotInitialized();
        executor.execute(() -> {
            sleep(DELAY);
            value.addAndGet(val);
        });
    }

    public long getValue() {
        throwIfNotInitialized();
        return value.longValue();
    }

    private void sleep(int delay) {
        try {
            Thread.sleep(delay);
        } catch (InterruptedException e) {
        }
    }

    private void throwIfNotInitialized() {
        if (!initialized) {
            throw new IllegalStateException("Service is not initialized");
        }
    }
}
```

## 4。有意识测试

现在，让我们创建测试类:

```java
public class AsyncServiceLongRunningManualTest {
    private AsyncService asyncService;

    @Before
    public void setUp() {
        asyncService = new AsyncService();
    }

    //...
}
```

我们的测试检查在调用`initialize`方法后，服务的初始化是否在指定的超时时间内发生(默认为 10s)。

这个测试用例仅仅等待服务初始化状态改变，或者如果状态没有改变，抛出一个`ConditionTimeoutException` 。

在指定的初始延迟(默认为 100 毫秒)之后，通过以定义的间隔(默认为 100 毫秒)轮询我们的服务的`Callable`来获得状态。这里我们使用超时、间隔和延迟的默认设置:

```java
asyncService.initialize();
await()
  .until(asyncService::isInitialized);
```

这里，我们使用`await`——`Awaitility` 类的静态方法之一。它返回一个`ConditionFactory` 类的实例。为了增加可读性，我们也可以使用其他方法，比如`given` 。

默认的定时参数可以使用`Awaitility`类中的静态方法来更改:

```java
Awaitility.setDefaultPollInterval(10, TimeUnit.MILLISECONDS);
Awaitility.setDefaultPollDelay(Duration.ZERO);
Awaitility.setDefaultTimeout(Duration.ONE_MINUTE);
```

这里我们可以看到`Duration`类的使用，它为最常用的时间段提供了有用的常量。

我们还可以**为每个`await`调用**提供定制的计时值。在这里，我们预计初始化最多在 5 秒钟后发生，至少在 100 毫秒后发生，轮询间隔为 100 毫秒:

```java
asyncService.initialize();
await()
    .atLeast(Duration.ONE_HUNDRED_MILLISECONDS)
    .atMost(Duration.FIVE_SECONDS)
  .with()
    .pollInterval(Duration.ONE_HUNDRED_MILLISECONDS)
    .until(asyncService::isInitialized);
```

值得一提的是，`ConditionFactory` 包含了额外的方法，比如`with`、`then`、`and`、`given.` ，这些方法不做任何事情，只是返回`this`，但是它们可以用来增强测试条件的可读性。

## 5。使用匹配器

可用性还允许使用`hamcrest`匹配器来检查表达式的结果。例如，在调用`addValue`方法后，我们可以检查我们的`long`值是否如预期的那样改变了:

```java
asyncService.initialize();
await()
  .until(asyncService::isInitialized);
long value = 5;
asyncService.addValue(value);
await()
  .until(asyncService::getValue, equalTo(value));
```

注意，在这个例子中，我们使用第一个`await` 调用来等待服务初始化。否则，`getValue`方法将抛出一个`IllegalStateException`。

## 6。忽略异常

有时，我们会遇到这样的情况:在异步作业完成之前，方法会抛出异常。在我们的服务中，它可以是在服务初始化之前对`getValue` 方法的调用。

可用性提供了忽略这个异常而不使测试失败的可能性。

例如，让我们在初始化后立即检查`getValue` 结果是否等于零，忽略`IllegalStateException`:

```java
asyncService.initialize();
given().ignoreException(IllegalStateException.class)
  .await().atMost(Duration.FIVE_SECONDS)
  .atLeast(Duration.FIVE_HUNDRED_MILLISECONDS)
  .until(asyncService::getValue, equalTo(0L));
```

## 7 .**。使用代理**

如第 2 节所述，我们需要包含`awaitility-proxy` 来使用基于代理的条件。代理的思想是为条件提供真正的方法调用，而不需要实现`Callable`或 lambda 表达式。

让我们使用`AwaitilityClassProxy.to` 静态方法来检查`AsyncService` 是否被初始化:

```java
asyncService.initialize();
await()
  .untilCall(to(asyncService).isInitialized(), equalTo(true));
```

## 8。访问字段

可用性甚至可以访问私有字段来对它们执行断言。在下面的示例中，我们可以看到获取服务初始化状态的另一种方法:

```java
asyncService.initialize();
await()
  .until(fieldIn(asyncService)
  .ofType(boolean.class)
  .andWithName("initialized"), equalTo(true));
```

## 9。结论

在这个快速教程中，我们介绍了 Awaitility 库，熟悉了用于测试异步系统的基本 DSL，并看到了一些高级特性，这些特性使这个库在实际项目中变得灵活和易于使用。

和往常一样，所有代码示例都可以在 Github 的[上获得。](https://web.archive.org/web/20221012100327/https://github.com/eugenp/tutorials/tree/master/libraries-testing)