# 网飞伺服系统简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/netflix-servo>

## 1。概述

[网飞伺服](https://web.archive.org/web/20220625174936/https://github.com/Netflix/servo)是一个 Java 应用的度量工具。伺服类似于 [Dropwizard Metrics](https://web.archive.org/web/20220625174936/http://metrics.dropwizard.io/) ，然而要简单得多。它只利用 JMX 来提供一个简单的接口来公开和发布应用程序指标。

在本文中，我们将介绍 Servo 提供了什么，以及我们如何使用它来收集和发布应用程序指标。

## 2。Maven 依赖关系

在我们深入实际实现之前，让我们将[伺服](https://web.archive.org/web/20220625174936/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22servo-core%22)依赖项添加到`pom.xml`文件中:

```java
<dependency>
    <groupId>com.netflix.servo</groupId>
    <artifactId>servo-core</artifactId>
    <version>0.12.16</version>
</dependency>
```

此外，还有许多扩展可用，如[伺服阿帕奇](https://web.archive.org/web/20220625174936/https://search.maven.org/classic/#search%7Cga%7C1%7Cservo-apache)、[伺服 AWS](https://web.archive.org/web/20220625174936/https://search.maven.org/classic/#search%7Cga%7C1%7Cservo-aws) 等。我们以后可能需要它们。这些扩展的最新版本也可以在 [Maven Central](https://web.archive.org/web/20220625174936/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.netflix.servo%22) 上找到。

## 3。收集指标

首先，让我们看看如何从我们的应用程序中收集指标。

伺服提供了四种主要的公制类型:[`Counter`](https://web.archive.org/web/20220625174936/https://netflix.github.io/servo/current/servo-core/docs/javadoc/com/netflix/servo/monitor/Counter.html)[`Gauge`](https://web.archive.org/web/20220625174936/https://netflix.github.io/servo/current/servo-core/docs/javadoc/com/netflix/servo/monitor/Gauge.html)[`Timer,`](https://web.archive.org/web/20220625174936/https://netflix.github.io/servo/current/servo-core/docs/javadoc/com/netflix/servo/monitor/Timer.html) 和`[Informational](https://web.archive.org/web/20220625174936/https://netflix.github.io/servo/current/servo-core/docs/javadoc/com/netflix/servo/monitor/Informational.html)`。

### 3.1。公制类型—**`Counter`**

`[Counters](https://web.archive.org/web/20220625174936/https://netflix.github.io/servo/current/servo-core/docs/javadoc/com/netflix/servo/monitor/Counter.html)`用于记录增量。常用的实现有`[BasicCounter](https://web.archive.org/web/20220625174936/https://netflix.github.io/servo/current/servo-core/docs/javadoc/com/netflix/servo/monitor/BasicCounter.html)`、`[StepCounter](https://web.archive.org/web/20220625174936/https://netflix.github.io/servo/current/servo-core/docs/javadoc/com/netflix/servo/monitor/StepCounter.html)`和`[PeakRateCounter](https://web.archive.org/web/20220625174936/https://netflix.github.io/servo/current/servo-core/docs/javadoc/com/netflix/servo/monitor/PeakRateCounter.html)`。

简单明了地做一个柜台应该做的事情:

```java
Counter counter = new BasicCounter(MonitorConfig.builder("test").build());
assertEquals("counter should start with 0", 0, counter.getValue().intValue());

counter.increment();

assertEquals("counter should have increased by 1", 1, counter.getValue().intValue());

counter.increment(-1);

assertEquals("counter should have decreased by 1", 0, counter.getValue().intValue());
```

`PeakRateCounter`返回轮询间隔期间给定秒的最大计数:

```java
Counter counter = new PeakRateCounter(MonitorConfig.builder("test").build());
assertEquals(
  "counter should start with 0", 
  0, counter.getValue().intValue());

counter.increment();
SECONDS.sleep(1);

counter.increment();
counter.increment();

assertEquals("peak rate should have be 2", 2, counter.getValue().intValue());
```

与其他计数器不同， `StepCounter`记录上一轮询间隔的每秒速率:

```java
System.setProperty("servo.pollers", "1000");
Counter counter = new StepCounter(MonitorConfig.builder("test").build());

assertEquals("counter should start with rate 0.0", 0.0, counter.getValue());

counter.increment();
SECONDS.sleep(1);

assertEquals(
  "counter rate should have increased to 1.0", 
  1.0, counter.getValue());
```

注意，我们在上面的代码中将`servo.pollers`设置为`1000`。那就是将轮询间隔设置为 1 秒，而不是默认的 60 秒和 10 秒。稍后我们将对此进行更多的讨论。

### 3.2。公制类型—**`Gauge`**

[`Gauge`](https://web.archive.org/web/20220625174936/https://netflix.github.io/servo/current/servo-core/docs/javadoc/com/netflix/servo/monitor/Gauge.html) 是一个简单的监视器，返回当前值。提供了[`BasicGauge`](https://web.archive.org/web/20220625174936/https://netflix.github.io/servo/current/servo-core/docs/javadoc/com/netflix/servo/monitor/BasicGauge.html)[`MinGauge`](https://web.archive.org/web/20220625174936/https://netflix.github.io/servo/current/servo-core/docs/javadoc/com/netflix/servo/monitor/MinGauge.html)[`MaxGauge`](https://web.archive.org/web/20220625174936/https://netflix.github.io/servo/current/servo-core/docs/javadoc/com/netflix/servo/monitor/MaxGauge.html)[`NumberGauges`](https://web.archive.org/web/20220625174936/https://netflix.github.io/servo/current/servo-core/docs/javadoc/com/netflix/servo/monitor/NumberGauge.html)。

`BasicGauge`调用一个`Callable`来获取当前值。我们可以得到一个集合的大小，一个`BlockingQueue`的最新值或者任何需要小计算的值。

```java
Gauge<Double> gauge = new BasicGauge<>(MonitorConfig.builder("test")
  .build(), () -> 2.32);

assertEquals(2.32, gauge.getValue(), 0.01);
```

`MaxGauge`和`MinGauge`分别用于记录最大值和最小值:

```java
MaxGauge gauge = new MaxGauge(MonitorConfig.builder("test").build());
assertEquals(0, gauge.getValue().intValue());

gauge.update(4);
assertEquals(4, gauge.getCurrentValue(0));

gauge.update(1);
assertEquals(4, gauge.getCurrentValue(0));
```

`NumberGauge` ( [`LongGauge`](https://web.archive.org/web/20220625174936/https://netflix.github.io/servo/current/servo-core/docs/javadoc/com/netflix/servo/monitor/LongGauge.html) ， [`DoubleGauge`](https://web.archive.org/web/20220625174936/https://netflix.github.io/servo/current/servo-core/docs/javadoc/com/netflix/servo/monitor/DoubleGauge.html) )裹甲提供`Number` ( `Long`，`Double`)。为了使用这些量规收集指标，我们必须确保`Number`是线程安全的。

### 3.3。公制类型—**`Timer`**

[`Timers`](https://web.archive.org/web/20220625174936/https://netflix.github.io/servo/current/servo-core/docs/javadoc/com/netflix/servo/monitor/Timer.html) 帮助测量特定事件的持续时间。默认实现有 [`BasicTimer`](https://web.archive.org/web/20220625174936/https://netflix.github.io/servo/current/servo-core/docs/javadoc/com/netflix/servo/monitor/BasicTimer.html) 、 [`StatsTimer`](https://web.archive.org/web/20220625174936/https://netflix.github.io/servo/current/servo-core/docs/javadoc/com/netflix/servo/monitor/StatsTimer.html) 、 [`BucketTimer`](https://web.archive.org/web/20220625174936/https://netflix.github.io/servo/current/servo-core/docs/javadoc/com/netflix/servo/monitor/BucketTimer.html) 。

`BasicTimer`记录总时间、计数和其他简单统计:

```java
BasicTimer timer = new BasicTimer(MonitorConfig.builder("test").build(), SECONDS);
Stopwatch stopwatch = timer.start();

SECONDS.sleep(1);
timer.record(2, SECONDS);
stopwatch.stop();

assertEquals("timer should count 1 second", 1, timer.getValue().intValue());
assertEquals("timer should count 3 seconds in total", 
  3.0, timer.getTotalTime(), 0.01);
assertEquals("timer should record 2 updates", 2, timer.getCount().intValue());
assertEquals("timer should have max 2", 2, timer.getMax(), 0.01);
```

`StatsTimer`通过轮询间隔之间的采样提供更丰富的统计数据:

```java
System.setProperty("netflix.servo", "1000");
StatsTimer timer = new StatsTimer(MonitorConfig
  .builder("test")
  .build(), new StatsConfig.Builder()
  .withComputeFrequencyMillis(2000)
  .withPercentiles(new double[] { 99.0, 95.0, 90.0 })
  .withPublishMax(true)
  .withPublishMin(true)
  .withPublishCount(true)
  .withPublishMean(true)
  .withPublishStdDev(true)
  .withPublishVariance(true)
  .build(), SECONDS);
Stopwatch stopwatch = timer.start();

SECONDS.sleep(1);
timer.record(3, SECONDS);
stopwatch.stop();

stopwatch = timer.start();
timer.record(6, SECONDS);
SECONDS.sleep(2);
stopwatch.stop();

assertEquals("timer should count 12 seconds in total", 
  12, timer.getTotalTime());
assertEquals("timer should count 12 seconds in total", 
  12, timer.getTotalMeasurement());
assertEquals("timer should record 4 updates", 4, timer.getCount());
assertEquals("stats timer value time-cost/update should be 2", 
  3, timer.getValue().intValue());

final Map<String, Number> metricMap = timer.getMonitors().stream()
  .collect(toMap(monitor -> getMonitorTagValue(monitor, "statistic"),
    monitor -> (Number) monitor.getValue()));

assertThat(metricMap.keySet(), containsInAnyOrder(
  "count", "totalTime", "max", "min", "variance", "stdDev", "avg", 
  "percentile_99", "percentile_95", "percentile_90"));
```

`BucketTimer`提供了一种通过分时段取值范围获得样本分布的方法:

```java
BucketTimer timer = new BucketTimer(MonitorConfig
  .builder("test")
  .build(), new BucketConfig.Builder()
  .withBuckets(new long[] { 2L, 5L })
  .withTimeUnit(SECONDS)
  .build(), SECONDS);

timer.record(3);
timer.record(6);

assertEquals(
  "timer should count 9 seconds in total",
  9, timer.getTotalTime().intValue());

Map<String, Long> metricMap = timer.getMonitors().stream()
  .filter(monitor -> monitor.getConfig().getTags().containsKey("servo.bucket"))
  .collect(toMap(
    m -> getMonitorTagValue(m, "servo.bucket"),
    m -> (Long) m.getValue()));

assertThat(metricMap, allOf(hasEntry("bucket=2s", 0L), hasEntry("bucket=5s", 1L),
  hasEntry("bucket=overflow", 1L)));
```

为了跟踪可能持续数小时的长时间操作，我们可以使用复合监视器 [`DurationTimer`](https://web.archive.org/web/20220625174936/https://netflix.github.io/servo/current/servo-core/docs/javadoc/com/netflix/servo/monitor/DurationTimer.html) 。

### 3.4。公制类型—**`Informational`**

此外，我们可以利用 [`Informational`](https://web.archive.org/web/20220625174936/https://netflix.github.io/servo/current/servo-core/docs/javadoc/com/netflix/servo/monitor/Informational.html) 监视器来记录描述性信息，以帮助调试和诊断。唯一的实现是 [`BasicInformational`](https://web.archive.org/web/20220625174936/https://netflix.github.io/servo/current/servo-core/docs/javadoc/com/netflix/servo/monitor/BasicInformational.html) ，它的用法再简单不过了:

```java
BasicInformational informational = new BasicInformational(
  MonitorConfig.builder("test").build());
informational.setValue("information collected");
```

### 3.5。`MonitorRegistry`

度量类型都是类型 [`Monitor`](https://web.archive.org/web/20220625174936/https://netflix.github.io/servo/current/servo-core/docs/javadoc/com/netflix/servo/monitor/Monitor.html) ，这是`Servo`的基础。我们现在知道各种工具收集原始指标，但是为了报告数据，我们需要注册这些监视器。

请注意，每个单独配置的监视器应该注册一次，并且只能注册一次，以确保指标的正确性。所以我们可以使用单例模式注册监视器。

很多时候，我们可以用 [`DefaultMonitorRegistry`](https://web.archive.org/web/20220625174936/https://netflix.github.io/servo/current/servo-core/docs/javadoc/com/netflix/servo/DefaultMonitorRegistry.html) 来注册显示器:

```java
Gauge<Double> gauge = new BasicGauge<>(MonitorConfig.builder("test")
  .build(), () -> 2.32);
DefaultMonitorRegistry.getInstance().register(gauge);
```

如果要动态注册一个显示器，可以使用 [`DynamicTimer`](https://web.archive.org/web/20220625174936/https://netflix.github.io/servo/current/servo-core/docs/javadoc/com/netflix/servo/monitor/DynamicTimer.html) 、 [`DynamicCounter`](https://web.archive.org/web/20220625174936/https://netflix.github.io/servo/current/servo-core/docs/javadoc/com/netflix/servo/monitor/DynamicCounter.html) :

```java
DynamicCounter.increment("monitor-name", "tag-key", "tag-value");
```

**注意，每次更新值时，动态注册会导致昂贵的查找操作。**

Servo 还提供了几个助手方法来注册对象中声明的监视器:

```java
Monitors.registerObject("testObject", this);
assertTrue(Monitors.isObjectRegistered("testObject", this));
```

方法 [`registerObject`](https://web.archive.org/web/20220625174936/https://netflix.github.io/servo/current/servo-core/docs/javadoc/com/netflix/servo/monitor/Monitors.html#registerObject(java.lang.String, java.lang.Object)) 将使用反射添加注释 [`@Monitor`](https://web.archive.org/web/20220625174936/https://netflix.github.io/servo/current/servo-core/docs/javadoc/com/netflix/servo/annotations/Monitor.html) 声明的`Monitors`的所有实例，并添加 [`@MonitorTags`](https://web.archive.org/web/20220625174936/https://netflix.github.io/servo/current/servo-core/docs/javadoc/com/netflix/servo/annotations/MonitorTags.html) 声明的标签:

```java
@Monitor(
  name = "integerCounter",
  type = DataSourceType.COUNTER,
  description = "Total number of update operations.")
private AtomicInteger updateCount = new AtomicInteger(0);

@MonitorTags
private TagList tags = new BasicTagList(
  newArrayList(new BasicTag("tag-key", "tag-value")));

@Test
public void givenAnnotatedMonitor_whenUpdated_thenDataCollected() throws Exception {
    System.setProperty("servo.pollers", "1000");
    Monitors.registerObject("testObject", this);
    assertTrue(Monitors.isObjectRegistered("testObject", this));

    updateCount.incrementAndGet();
    updateCount.incrementAndGet();
    SECONDS.sleep(1);

    List<List<Metric>> metrics = observer.getObservations();

    assertThat(metrics, hasSize(greaterThanOrEqualTo(1)));

    Iterator<List<Metric>> metricIterator = metrics.iterator();
    metricIterator.next(); //skip first empty observation

    while (metricIterator.hasNext()) {
        assertThat(metricIterator.next(), hasItem(
          hasProperty("config", 
          hasProperty("name", is("integerCounter")))));
    }
}
```

## 4。发布指标

有了收集到的指标，我们可以将其以任何格式发布到，比如在各种数据可视化平台上呈现时间序列图。为了发布指标，我们需要定期从监视器观察中轮询数据。

### 4.1。`MetricPoller`

[`MetricPoller`](https://web.archive.org/web/20220625174936/https://netflix.github.io/servo/current/servo-core/docs/javadoc/com/netflix/servo/publish/MetricPoller.html) 用作度量取器。我们可以获取[`MonitorRegistries`](https://web.archive.org/web/20220625174936/https://netflix.github.io/servo/current/servo-core/docs/javadoc/com/netflix/servo/publish/MonitorRegistryMetricPoller.html)[JVM](https://web.archive.org/web/20220625174936/https://netflix.github.io/servo/current/servo-core/docs/javadoc/com/netflix/servo/publish/JvmMetricPoller.html)[JMX](https://web.archive.org/web/20220625174936/https://netflix.github.io/servo/current/servo-core/docs/javadoc/com/netflix/servo/publish/JmxMetricPoller.html)的度量。在扩展的帮助下，我们可以轮询像 [Apache 服务器状态](https://web.archive.org/web/20220625174936/https://github.com/Netflix/servo/tree/master/servo-apache)和 [Tomcat 指标](https://web.archive.org/web/20220625174936/https://github.com/Netflix/servo/tree/master/servo-tomcat)这样的指标。

```java
MemoryMetricObserver observer = new MemoryMetricObserver();
PollRunnable pollRunnable = new PollRunnable(new JvmMetricPoller(),
  new BasicMetricFilter(true), observer);
PollScheduler.getInstance().start();
PollScheduler.getInstance().addPoller(pollRunnable, 1, SECONDS);

SECONDS.sleep(1);
PollScheduler.getInstance().stop();
List<List<Metric>> metrics = observer.getObservations();

assertThat(metrics, hasSize(greaterThanOrEqualTo(1)));
List<String> keys = extractKeys(metrics);

assertThat(keys, hasItems("loadedClassCount", "initUsage", "maxUsage", "threadCount"));
```

这里我们创建了一个`[JvmMetricPoller](https://web.archive.org/web/20220625174936/https://netflix.github.io/servo/current/servo-core/docs/javadoc/com/netflix/servo/publish/JvmMetricPoller.html)`来轮询 JVM 的指标。将轮询器添加到调度程序时，我们让轮询任务每秒运行一次。系统默认轮询器配置在`[Pollers](https://web.archive.org/web/20220625174936/https://netflix.github.io/servo/current/servo-core/docs/javadoc/com/netflix/servo/monitor/Pollers.html)`中定义，但是我们可以通过系统属性`servo.pollers`指定轮询器。

### 4.2。`MetricObserver`

当轮询度量时，注册的 [`MetricObservers`](https://web.archive.org/web/20220625174936/https://netflix.github.io/servo/current/servo-core/docs/javadoc/com/netflix/servo/publish/MetricObserver.html) 的观测值将被更新。

`MetricObservers`默认提供的有[`MemoryMetricObserver`](https://web.archive.org/web/20220625174936/https://netflix.github.io/servo/current/servo-core/docs/javadoc/com/netflix/servo/publish/MemoryMetricObserver.html)[`FileMetricObserver`](https://web.archive.org/web/20220625174936/https://netflix.github.io/servo/current/servo-core/docs/javadoc/com/netflix/servo/publish/FileMetricObserver.html)[`AsyncMetricObserver`](https://web.archive.org/web/20220625174936/https://netflix.github.io/servo/current/servo-core/docs/javadoc/com/netflix/servo/publish/AsyncMetricObserver.html)。我们已经在前面的代码示例中展示了如何使用`MemoryMetricObserver`。

目前，有几个有用的扩展可用:

*   `[AtlasMetricObserver](https://web.archive.org/web/20220625174936/https://github.com/Netflix/servo/tree/master/servo-atlas)`:将指标发布到[网飞地图集](https://web.archive.org/web/20220625174936/https://github.com/Netflix/atlas)，以在内存中生成用于分析的时间序列数据
*   [`CloudWatchMetricObserver`](https://web.archive.org/web/20220625174936/https://github.com/Netflix/servo/tree/master/servo-aws) :将指标推送到[亚马逊 CloudWatch](https://web.archive.org/web/20220625174936/https://aws.amazon.com/cloudwatch/) 进行指标监控和跟踪
*   `[GraphiteObserver](https://web.archive.org/web/20220625174936/https://github.com/Netflix/servo/tree/master/servo-graphite)`:将指标发布到 [Graphite](https://web.archive.org/web/20220625174936/http://graphiteapp.org/) 进行存储和图形化

我们可以实现一个定制的`MetricObserver`来将应用程序指标发布到我们认为合适的地方。唯一需要关心的是处理更新的指标:

```java
public class CustomObserver extends BaseMetricObserver {

    //...

    @Override
    public void updateImpl(List<Metric> metrics) {
        //TODO
    }
}
```

### 4.3。发布到网飞图集

[`Atlas`](https://web.archive.org/web/20220625174936/https://github.com/Netflix/atlas) 是另一个来自网飞的度量相关工具。这是一个管理多维时间序列数据的工具，是发布我们收集的指标的完美地方。

现在，我们将演示如何将我们的指标发布到网飞地图集。

首先，让我们将 [`servo-atlas`](https://web.archive.org/web/20220625174936/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.netflix.servo%22%20AND%20a%3A%22servo-atlas%22) 依赖项附加到`pom.xml`:

```java
<dependency>
      <groupId>com.netflix.servo</groupId>
      <artifactId>servo-atlas</artifactId>
      <version>${netflix.servo.ver}</version>
</dependency>

<properties>
    <netflix.servo.ver>0.12.17</netflix.servo.ver>
</properties>
```

这个依赖项包括一个`AtlasMetricObserver`来帮助我们向`Atlas`发布指标。

然后，我们将建立一个 Atlas 服务器:

```java
$ curl -LO 'https://github.com/Netflix/atlas/releases/download/v1.4.4/atlas-1.4.4-standalone.jar'
$ curl -LO 'https://raw.githubusercontent.com/Netflix/atlas/v1.4.x/conf/memory.conf'
$ java -jar atlas-1.4.4-standalone.jar memory.conf
```

为了节省测试时间，让我们在`memory.conf`中将步长设置为 1 秒，这样我们就可以生成一个包含足够多指标细节的时间序列图。

`AtlasMetricObserver`需要一个简单的配置和一列标签。给定标签的指标将被推送到 Atlas:

```java
System.setProperty("servo.pollers", "1000");
System.setProperty("servo.atlas.batchSize", "1");
System.setProperty("servo.atlas.uri", "http://localhost:7101/api/v1/publish");
AtlasMetricObserver observer = new AtlasMetricObserver(
  new BasicAtlasConfig(), BasicTagList.of("servo", "counter"));

PollRunnable task = new PollRunnable(
  new MonitorRegistryMetricPoller(), new BasicMetricFilter(true), observer);
```

在使用`PollRunnable`任务启动`PollScheduler`之后，我们可以自动向 Atlas 发布指标:

```java
Counter counter = new BasicCounter(MonitorConfig
  .builder("test")
  .withTag("servo", "counter")
  .build());
DefaultMonitorRegistry
  .getInstance()
  .register(counter);
assertThat(atlasValuesOfTag("servo"), not(containsString("counter")));

for (int i = 0; i < 3; i++) {
    counter.increment(RandomUtils.nextInt(10));
    SECONDS.sleep(1);
    counter.increment(-1 * RandomUtils.nextInt(10));
    SECONDS.sleep(1);
}

assertThat(atlasValuesOfTag("servo"), containsString("counter"));
```

基于这些指标，我们可以使用 Atlas 的 [graph API](https://web.archive.org/web/20220625174936/https://github.com/Netflix/atlas/wiki/Graph) 生成一个线图:

[![graph](img/249ed4352517784c289c8b6fba6d345d.png)](/web/20220625174936/https://www.baeldung.com/wp-content/uploads/2017/06/graph.png)

## 5。总结

在本文中，我们介绍了如何使用网飞伺服收集和发布应用程序的指标。

如果你没有读过我们对 Dropwizard 指标的介绍，请在这里查看[与 Servo 的快速比较。](/web/20220625174936/https://www.baeldung.com/dropwizard-metrics)

和往常一样，本文的完整实现代码可以在 Github 上找到[。](https://web.archive.org/web/20220625174936/https://github.com/eugenp/tutorials/tree/master/metrics)