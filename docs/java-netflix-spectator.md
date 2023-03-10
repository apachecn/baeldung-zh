# 网飞观察家指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-netflix-spectator>

## 1.概观

**[旁观者](https://web.archive.org/web/20220628091209/https://github.com/Netflix/spectator)是一个为维度时间序列后端系统插装代码和收集数据的库。** Spectator 起源于网飞进行各种指标收集，与之配套使用的相应后端系统主要是 [Atlas](https://web.archive.org/web/20220628091209/https://github.com/Netflix/atlas) 。

在本教程中，我们将了解旁观者提供了什么，以及我们如何使用它来收集指标。

## 2.Maven 依赖性

在我们深入实际实现之前，让我们将[旁观者](https://web.archive.org/web/20220628091209/https://search.maven.org/artifact/com.netflix.spectator/spectator-api)依赖项添加到`pom.xml`文件中:

```java
<dependency>
    <groupId>com.netflix.spectator</groupId>
    <artifactId>spectator-api</artifactId>
    <version>1.0.11</version>
</dependency>
```

`spectator-api`是核心的观众图书馆。

## 3.`Registry`、`Meter`和指标的基础知识

在我们开始深入研究这个库之前，我们应该首先理解`Registry, Meter,` 和度量`.`的基础知识

*   [`Registry`](https://web.archive.org/web/20220628091209/https://www.javadoc.io/doc/com.netflix.spectator/spectator-api/0.70.2/com/netflix/spectator/api/Registry.html) 是我们维护一套米的地方
*   [`Meter`](https://web.archive.org/web/20220628091209/https://www.javadoc.io/static/com.netflix.spectator/spectator-api/0.38.1/com/netflix/spectator/api/Meter.html) 用于收集一组关于我们应用的测量值，如`Counter, Timer, Gauge,` 等。
*   指标是我们在`Meter,`上显示的单个测量值，例如计数、持续时间、最大值、平均值等。

让我们进一步探索这些，并了解它们是如何在观众图书馆中使用的。

## 4.`Registry`

旁观者库自带`Registry`作为接口，有一些内置的实现，比如`DefaultRegistry and` `NoopRegistry`。我们还可以根据自己的需求创建定制的`Registry`实现。

可以使用如下的`Registry`实现:

```java
Registry registry = new DefaultRegistry();
```

## 5.`Meter`

**`Meter`主要有主动和被动两种。**

### 5.1.活动仪表

这些仪表用于测量某些事件的发生。我们有三种这样的仪表:

*   [T2`Counter`](https://web.archive.org/web/20220628091209/https://www.javadoc.io/static/com.netflix.spectator/spectator-api/0.38.1/com/netflix/spectator/api/Counter.html)
*   倍
*   [T2`DistributionSummary`](https://web.archive.org/web/20220628091209/https://www.javadoc.io/static/com.netflix.spectator/spectator-api/0.38.1/com/netflix/spectator/api/DistributionSummary.html)

### 5.2.无源仪表

这些度量工具用于在需要时获取指标值。例如，运行线程的数量可能是我们想要测量的指标。我们只有一种这样的仪表，`Gauge.`

接下来，我们来详细探讨一下这些不同类型的仪表。

## 6.`Counter`

这些仪表测量事件发生的频率。例如，假设我们想测量元素从列表中插入或删除的速率。

让我们首先在初始化时向`Registry`对象注册一个计数器:

```java
insertCounter = registry.counter("list.insert.count");
removeCounter = registry.counter("list.remove.count");
```

在这里，我们可以允许用户使用任何使用依赖注入的`Registry`实现。

现在，我们可以增加或减少`Counter`计数器，分别用于添加到列表或从列表中删除:

```java
requestList.add(element);
insertCounter.increment();

requestList.remove(0);
removeCounter.increment();
```

这样，我们可以生成两个度量，稍后，我们可以将度量推送到`Atlas`进行可视化。

## 7.定时器

这些仪表测量在某项活动上花费的时间。spectator 支持两种类型的计时器:

*   [T2`Timer`](https://web.archive.org/web/20220628091209/https://www.javadoc.io/static/com.netflix.spectator/spectator-api/0.38.1/com/netflix/spectator/api/Timer.html)
*   [T2`LongTaskTimer`](https://web.archive.org/web/20220628091209/https://www.javadoc.io/static/com.netflix.spectator/spectator-api/0.38.1/com/netflix/spectator/api/LongTaskTimer.html)

### 7.1.`Timer`

这些定时器主要用于测量短期事件。因此，他们通常测量事件完成后花费的时间。

首先，我们需要在`Registry`中注册该仪表:

```java
requestLatency = registry.timer("app.request.latency");
```

接下来，我们可以调用`Timer`的`record()` 方法来测量处理请求所花费的时间:

```java
requestLatency.record(() -> handleRequest(input));
```

### 7.2.`LongTaskTimer`

这些计时器主要用于测量长时间运行任务的持续时间。因此，即使事件正在进行中，我们也可以查询这些计时器。这也是一种类型的`Gauge.`，当事件正在进行时，我们可以看到像`duration`和`activeTasks`这样的指标。

同样，作为第一步，我们需要注册该仪表:

```java
refreshDuration = LongTaskTimer.get(registry, registry.createId("metadata.refreshDuration"));
```

接下来，我们可以使用`LongTaskTimer`开始和停止围绕长期运行任务的测量:

```java
long taskId = refreshDuration.start();
try {
    Thread.sleep(input);
    return "Done";
} catch (InterruptedException e) {
    e.printStackTrace();
    throw e;
} finally {
    refreshDuration.stop(taskId);
}
```

## 8.估计

正如我们前面讨论的，仪表是无源仪表。因此，它们给出了在运行任务的任何时间点采样的值。因此，举例来说，如果我们想知道 JVM 中正在运行的线程的数量或者任何时候堆内存的使用情况，我们可以使用它。

我们有两种类型的压力表:

*   轮询仪表
*   活动仪表

### 8.1.轮询仪表

这种类型的仪表在后台轮询正在运行的任务的值。它在它监视的任务上创建一个钩子。因此，不需要更新该量表中的值。

现在，让我们看看如何使用这个标尺来监控`List`的尺寸:

```java
PolledMeter.using(registry)
  .withName("list.size")
  .monitorValue(listSize);
```

这里，`PolledMeter `是允许使用`monitorValue()` 方法在`listSize`上进行后台轮询的类。此外，`listSize`是跟踪样本列表大小的变量。

### 8.2.活动仪表

这种类型的计量器需要定期手动更新与监控任务中的更新相关的值。这是一个使用主动仪表的例子`:`

```java
gauge = registry.gauge("list.size");
```

我们首先在`Registry`中注册这个量规。然后，我们在列表中添加或删除元素时手动更新它:

```java
list.add(element);
gauge.set(listSize);
list.remove(0);
gauge.set(listSize);
```

## 9.`DistributionSummary`

现在，我们将研究另一个被称为`DistributionSummary.`的指标，它跟踪事件的分布。该计量器可以测量请求有效负载的大小。例如，我们将使用`DistributionSummary`来度量请求的大小。

首先，像往常一样，我们在`Registry`中注册这个仪表:

```java
distributionSummary = registry.distributionSummary("app.request.size");
```

现在，我们可以使用这个类似于`Timer`的计量器来记录请求的大小:

```java
distributionSummary.record((long) input.length());
handleRequest();
```

## 10.观众对伺服对千分尺

[Servo](/web/20220628091209/https://www.baeldung.com/netflix-servo) 也是一个测量不同代码度量的库。旁观者是伺服的继承者，由网飞建造。旁观者最初是为 Java 8 推出的，从未来支持的角度来看，它是一个更好的选择。

这些网飞库是市场上可用于测量不同指标的各种选项之一。我们总是可以单独使用它们，或者我们可以选择像[千分尺](/web/20220628091209/https://www.baeldung.com/micrometer)这样的外观。Micrometer 允许用户在不同的度量测量库之间轻松切换。因此，它还允许选择不同的后端监控系统。

## 11.结论

在本文中，我们介绍了 Spectator，一个来自网飞的度量测量库。此外，我们还调查了各种有源和无源仪表的使用情况。我们可以将插桩数据推送并发布到时序数据库`Atlas`。

和往常一样，本文的完整实现代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220628091209/https://github.com/eugenp/tutorials/tree/master/metrics)