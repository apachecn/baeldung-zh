# 在 Java 中测量运行时间

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-measure-elapsed-time>

## 1。概述

在本文中，我们将看看如何在 Java 中测量运行时间。虽然这听起来很容易，但我们必须注意一些陷阱。

我们将探索提供测量运行时间功能的标准 Java 类和外部包。

## 2。简单测量

### 2.1。`currentTimeMillis()`

当我们在 Java 中遇到测量运行时间的需求时，我们可以尝试这样做:

```
long start = System.currentTimeMillis();
// ...
long finish = System.currentTimeMillis();
long timeElapsed = finish - start;
```

如果我们看代码，它非常有意义。我们在开始时得到一个时间戳，当代码结束时得到另一个时间戳。经过的时间是这两个值之间的差值。

然而，**结果可能会不准确，因为`System.currentTimeMillis()`测量的是`wall-clock`时间。**挂钟时间可能因多种原因而改变，例如，改变系统时间会影响结果，或者闰秒会破坏结果。

### 2.2。`nanoTime()`

`java.lang.System`类中的另一个方法是`nanoTime()`。如果我们看一下[的 Java 文档](https://web.archive.org/web/20221031165730/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/System.html#nanoTime())，我们会发现下面的语句:

`“This method can only be used to measure elapsed time and is not related to any other notion of system or wall-clock time.”`

让我们使用它:

```
long start = System.nanoTime();
// ...
long finish = System.nanoTime();
long timeElapsed = finish - start;
```

代码基本和以前一样。唯一的区别是获取时间戳的方法——`nanoTime()`而不是`currentTimeMillis()`。

我们还要注意，`nanoTime()`显然以纳秒为单位返回时间。因此，如果用不同的时间单位来测量经过的时间，我们必须相应地进行转换。

例如，要转换成毫秒，我们必须用纳秒的结果除以 1.000.000。

`nanoTime()`的另一个缺陷是**即使提供了纳秒精度，也不能保证纳秒分辨率**(即值更新的频率)。

但是，它确实保证分辨率至少会和`currentTimeMillis()`一样好。

## 3。Java 8

如果我们使用 Java 8——我们可以尝试新的`[java.time.Instant](https://web.archive.org/web/20221031165730/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/Instant.html)`和 [`java.time.Duration`](https://web.archive.org/web/20221031165730/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/Duration.html) 类。两者都是不可变的、线程安全的，并且使用它们自己的时标，即 *Java 时标，*新的`java.time` API 中的所有类也是如此。

### 3.1。Java 时标

传统的计时方式是将一天分为 24 小时 60 分钟 60 秒，这样一天就是 86.400 秒。然而，太阳日并不总是一样长。

UTC 时间刻度实际上允许一天有 86.399 或 86.401 SI 秒。一秒是科学上的“标准国际秒”,由铯 33 原子的辐射周期来定义。这是保持白昼与太阳一致所必需的。

Java 时间刻度将每个日历日精确地分成 86.400 个小部分，称为秒。没有闰秒。

### 3.2。`Instant`阶级

`Instant`类代表时间轴上的一个瞬间。基本上，从标准的 Java 纪元`1970-01-01T00:00:00Z`开始，它就是一个数字时间戳。

为了获得当前时间戳，我们可以使用`Instant.now()`静态方法。这个方法允许传入一个可选的 [`Clock`](https://web.archive.org/web/20221031165730/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/Clock.html) 参数。如果省略，则使用默认时区的系统时钟。

和前面的例子一样，我们可以将开始和结束时间存储在两个变量中。接下来，我们可以计算两个瞬间之间经过的时间。

我们还可以使用`Duration`类及其`between()`方法来获得两个 `Instant`对象之间的持续时间。最后，我们需要将`Duration`转换成毫秒:

```
Instant start = Instant.now();
// CODE HERE        
Instant finish = Instant.now();
long timeElapsed = Duration.between(start, finish).toMillis();
```

## 4。`StopWatch`

继续看库，Apache Commons Lang 提供了可以用来测量运行时间的`StopWatch`类。

### 4.1。Maven 依赖关系

我们可以通过更新 pom.xml 获得最新版本:

```
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

依赖关系的最新版本可以在[这里](https://web.archive.org/web/20221031165730/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.commons%22%20AND%20a%3A%22commons-lang3%22)查看。

### 4.2。用`StopWatch` 测量经过的时间

首先，我们需要获得该类的一个实例，然后我们可以简单地测量运行时间:

```
StopWatch watch = new StopWatch();
watch.start();
```

一旦我们运行了一个观察器，我们就可以执行我们想要进行基准测试的代码，然后在最后，我们简单地调用`stop()`方法。最后，为了得到实际的结果，我们调用`getTime()`:

```
watch.stop();
System.out.println("Time Elapsed: " + watch.getTime()); // Prints: Time Elapsed: 2501
```

**`StopWatch`有一些额外的辅助方法，我们可以使用它们来暂停或恢复我们的测量。**如果我们需要使我们的基准更加复杂，这可能会有所帮助。

最后，让我们注意这个类不是线程安全的。

## 5。结论

在 Java 中有很多方法来测量时间。我们已经通过使用`currentTimeMillis()`涵盖了一种非常“传统”(并且不准确)的方式。此外，我们检查了 Apache Common 的`StopWatch`,并查看了 Java 8 中可用的新类。

总的来说，对于简单和正确的时间测量，`nanoTime()`方法就足够了。打字也比`currentTimeMillis()`短。

但是，让我们注意，为了进行适当的基准测试，我们可以使用像 Java 微基准测试工具(JMH)这样的框架，而不是手工测量时间。这个话题超出了本文的范围，但是我们在这里探讨了一下。

最后，和往常一样，讨论中使用的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221031165730/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-time-measurements)