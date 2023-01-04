# Dropwizard 指标简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/dropwizard-metrics>

## 1。简介

[Metrics](https://web.archive.org/web/20220812052521/http://metrics.dropwizard.io/3.1.0/) 是一个 Java 库，为 Java 应用程序提供测量工具。

它有几个模块，在本文中，我们将详细阐述 metrics-core 模块、metrics-healthchecks 模块、metrics-servlet 模块和 metrics-servlet 模块，并概述其余模块，供您参考。

## 2。模块`metrics-core`

### 2.1。Maven 依赖关系

要使用`metrics-core`模块，只需要将一个依赖项添加到`pom.xml`文件中:

```java
<dependency>
    <groupId>io.dropwizard.metrics</groupId>
    <artifactId>metrics-core</artifactId>
    <version>3.1.2</version>
</dependency> 
```

你可以在这里找到它的最新版本。

### 2.2。`MetricRegistry`

简单地说，我们将使用`MetricRegistry`类来注册一个或多个指标。

我们可以为所有的指标使用一个指标注册中心，但是如果我们想要为不同的指标使用不同的报告方法，我们也可以将我们的指标分成组，并为每个组使用不同的指标注册中心。

现在让我们创建一个`MetricRegistry`:

```java
MetricRegistry metricRegistry = new MetricRegistry();
```

然后我们可以用这个`MetricRegistry`注册一些指标:

```java
Meter meter1 = new Meter();
metricRegistry.register("meter1", meter1);

Meter meter2 = metricRegistry.meter("meter2"); 
```

有两种创建新指标的基本方法:自己实例化一个或者从指标注册中心获得一个。正如你所看到的，我们在上面的例子中使用了它们，我们正在实例化`Meter`对象“meter1 ”,并且我们正在获取由`metricRegistry`创建的另一个`Meter`对象“meter2”。

在指标注册中心，每个指标都有一个唯一的名称，就像我们在上面使用“meter1”和“meter2”作为指标名称一样。`MetricRegistry`还提供了一组静态助手方法来帮助我们创建合适的指标名称:

```java
String name1 = MetricRegistry.name(Filter.class, "request", "count");
String name2 = MetricRegistry.name("CustomFilter", "response", "count"); 
```

如果我们需要管理一组度量注册表，我们可以使用`SharedMetricRegistries`类，它是单例的和线程安全的。我们可以在其中添加一个度量寄存器，从其中检索这个度量寄存器，然后删除它:

```java
SharedMetricRegistries.add("default", metricRegistry);
MetricRegistry retrievedMetricRegistry = SharedMetricRegistries.getOrCreate("default");
SharedMetricRegistries.remove("default"); 
```

## 3。度量概念

metrics-core 模块提供了几种常用的度量类型:`Meter`、`Gauge`、`Counter`、`Histogram`、`Timer`、 `Reporter` 输出度量值`.`

### 3.1。 `Meter`

A `Meter`测量事件发生次数和比率:

```java
Meter meter = new Meter();
long initCount = meter.getCount();
assertThat(initCount, equalTo(0L));

meter.mark();
assertThat(meter.getCount(), equalTo(1L));

meter.mark(20);
assertThat(meter.getCount(), equalTo(21L));

double meanRate = meter.getMeanRate();
double oneMinRate = meter.getOneMinuteRate();
double fiveMinRate = meter.getFiveMinuteRate();
double fifteenMinRate = meter.getFifteenMinuteRate(); 
```

`getCount()`方法返回事件发生次数计数，而`mark()`方法将事件发生次数计数加 1 或 n。`Meter`对象提供了四个速率，分别代表整个`Meter`生命周期、最近一分钟、最近五分钟和最近一个季度的平均速率。

### 3.2。 `Gauge`

`Gauge`是一个简单用于返回特定值的接口。metrics-core 模块提供了它的几种实现:[`RatioGauge`](https://web.archive.org/web/20220812052521/http://metrics.dropwizard.io/3.1.0/apidocs/com/codahale/metrics/RatioGauge.html)[`CachedGauge`](https://web.archive.org/web/20220812052521/http://metrics.dropwizard.io/3.1.0/apidocs/com/codahale/metrics/annotation/CachedGauge.html)[`DerivativeGauge`](https://web.archive.org/web/20220812052521/http://metrics.dropwizard.io/3.1.0/apidocs/com/codahale/metrics/DerivativeGauge.html)。

是一个抽象类，它测量一个值与另一个值的比率。

让我们看看如何使用它。首先，我们实现一个类`AttendanceRatioGauge`:

```java
public class AttendanceRatioGauge extends RatioGauge {
    private int attendanceCount;
    private int courseCount;

    @Override
    protected Ratio getRatio() {
        return Ratio.of(attendanceCount, courseCount);
    }

    // standard constructors
} 
```

然后我们测试它:

```java
RatioGauge ratioGauge = new AttendanceRatioGauge(15, 20);

assertThat(ratioGauge.getValue(), equalTo(0.75)); 
```

`CachedGauge`是另一个可以缓存值的抽象类，因此，当值的计算代价很高时，它非常有用。为了使用它，我们需要实现一个类`ActiveUsersGauge`:

```java
public class ActiveUsersGauge extends CachedGauge<List<Long>> {

    @Override
    protected List<Long> loadValue() {
        return getActiveUserCount();
    }

    private List<Long> getActiveUserCount() {
        List<Long> result = new ArrayList<Long>();
        result.add(12L);
        return result;
    }

    // standard constructors
}
```

然后，我们对其进行测试，看它是否如预期的那样工作:

```java
Gauge<List<Long>> activeUsersGauge = new ActiveUsersGauge(15, TimeUnit.MINUTES);
List<Long> expected = new ArrayList<>();
expected.add(12L);

assertThat(activeUsersGauge.getValue(), equalTo(expected)); 
```

在实例化`ActiveUsersGauge`时，我们将缓存的到期时间设置为 15 分钟。

`DerivativeGauge`也是一个抽象类，它允许你从其他`Gauge`中获取一个值作为它的值。

让我们看一个例子:

```java
public class ActiveUserCountGauge extends DerivativeGauge<List<Long>, Integer> {

    @Override
    protected Integer transform(List<Long> value) {
        return value.size();
    }

    // standard constructors
}
```

这个`Gauge`从一个`ActiveUsersGauge`中得到它的值，所以我们期望它是来自基本列表大小的值:

```java
Gauge<List<Long>> activeUsersGauge = new ActiveUsersGauge(15, TimeUnit.MINUTES);
Gauge<Integer> activeUserCountGauge = new ActiveUserCountGauge(activeUsersGauge);

assertThat(activeUserCountGauge.getValue(), equalTo(1)); 
```

当我们需要访问通过 JMX 公开的其他库的指标时，使用`JmxAttributeGauge`。

### 3.3。 `Counter`

`Counter`用于记录增量和减量；

```java
Counter counter = new Counter();
long initCount = counter.getCount();
assertThat(initCount, equalTo(0L));

counter.inc();
assertThat(counter.getCount(), equalTo(1L));

counter.inc(11);
assertThat(counter.getCount(), equalTo(12L));

counter.dec();
assertThat(counter.getCount(), equalTo(11L));

counter.dec(6);
assertThat(counter.getCount(), equalTo(5L));
```

### 3.4。 `Histogram`

`Histogram`用于跟踪一串`Long`值，并分析它们的统计特征，如`max, min, mean, median, standard deviation, 75th percentile`等；

```java
Histogram histogram = new Histogram(new UniformReservoir());
histogram.update(5);
long count1 = histogram.getCount();
assertThat(count1, equalTo(1L));

Snapshot snapshot1 = histogram.getSnapshot();
assertThat(snapshot1.getValues().length, equalTo(1));
assertThat(snapshot1.getValues()[0], equalTo(5L));

histogram.update(20);
long count2 = histogram.getCount();
assertThat(count2, equalTo(2L));

Snapshot snapshot2 = histogram.getSnapshot();
assertThat(snapshot2.getValues().length, equalTo(2));
assertThat(snapshot2.getValues()[1], equalTo(20L));
assertThat(snapshot2.getMax(), equalTo(20L));
assertThat(snapshot2.getMean(), equalTo(12.5));
assertEquals(10.6, snapshot2.getStdDev(), 0.1);
assertThat(snapshot2.get75thPercentile(), equalTo(20.0));
assertThat(snapshot2.get999thPercentile(), equalTo(20.0)); 
```

`Histogram`使用油藏采样对数据进行采样，当我们实例化一个`Histogram`对象时，需要显式设置它的油藏。

`Reservoir`是一个接口，metrics-core 提供了其中的四个实现:[`ExponentiallyDecayingReservoir`](https://web.archive.org/web/20220812052521/http://metrics.dropwizard.io/3.1.0/apidocs/com/codahale/metrics/ExponentiallyDecayingReservoir.html)[`UniformReservoir`](https://web.archive.org/web/20220812052521/http://metrics.dropwizard.io/3.1.0/apidocs/com/codahale/metrics/UniformReservoir.html)[`SlidingTimeWindowReservoir`](https://web.archive.org/web/20220812052521/http://metrics.dropwizard.io/3.1.0/apidocs/com/codahale/metrics/SlidingTimeWindowReservoir.html)`[SlidingWindowReservoir](https://web.archive.org/web/20220812052521/http://metrics.dropwizard.io/3.1.0/apidocs/com/codahale/metrics/SlidingWindowReservoir.html).`

在上一节中，我们提到除了使用构造函数`.` 之外，指标也可以由`MetricRegistry,` 创建。当我们使用`metricRegistry.histogram()`时，它返回一个带有`ExponentiallyDecayingReservoir` 实现的`Histogram`实例。

### 3.5。 `Timer`

`Timer`用于跟踪由`Context`对象表示的多个计时持续时间，并提供它们的统计数据:

```java
Timer timer = new Timer();
Timer.Context context1 = timer.time();
TimeUnit.SECONDS.sleep(5);
long elapsed1 = context1.stop();

assertEquals(5000000000L, elapsed1, 1000000);
assertThat(timer.getCount(), equalTo(1L));
assertEquals(0.2, timer.getMeanRate(), 0.1);

Timer.Context context2 = timer.time();
TimeUnit.SECONDS.sleep(2);
context2.close();

assertThat(timer.getCount(), equalTo(2L));
assertEquals(0.3, timer.getMeanRate(), 0.1); 
```

### 3.6。`Reporter`

当我们需要输出我们的测量值时，我们可以使用`Reporter`。这是一个接口，metrics-core 模块提供了它的几种实现，比如[`ConsoleReporter`](https://web.archive.org/web/20220812052521/http://metrics.dropwizard.io/3.1.0/apidocs/com/codahale/metrics/ConsoleReporter.html)[`CsvReporter`](https://web.archive.org/web/20220812052521/http://metrics.dropwizard.io/3.1.0/apidocs/com/codahale/metrics/CsvReporter.html)[`Slf4jReporter`](https://web.archive.org/web/20220812052521/http://metrics.dropwizard.io/3.1.0/apidocs/com/codahale/metrics/Slf4jReporter.html)[`JmxReporter`](https://web.archive.org/web/20220812052521/http://metrics.dropwizard.io/3.1.0/apidocs/com/codahale/metrics/JmxReporter.html)等等。

这里我们以`ConsoleReporter`为例:

```java
MetricRegistry metricRegistry = new MetricRegistry();

Meter meter = metricRegistry.meter("meter");
meter.mark();
meter.mark(200);
Histogram histogram = metricRegistry.histogram("histogram");
histogram.update(12);
histogram.update(17);
Counter counter = metricRegistry.counter("counter");
counter.inc();
counter.dec();

ConsoleReporter reporter = ConsoleReporter.forRegistry(metricRegistry).build();
reporter.start(5, TimeUnit.MICROSECONDS);
reporter.report(); 
```

这里是`ConsoleReporter:`的输出示例

```java
-- Histograms ------------------------------------------------------------------
histogram
count = 2
min = 12
max = 17
mean = 14.50
stddev = 2.50
median = 17.00
75% <= 17.00
95% <= 17.00
98% <= 17.00
99% <= 17.00
99.9% <= 17.00

-- Meters ----------------------------------------------------------------------
meter
count = 201
mean rate = 1756.87 events/second
1-minute rate = 0.00 events/second
5-minute rate = 0.00 events/second
15-minute rate = 0.00 events/second 
```

## 4。模块`metrics-healthchecks`

Metrics 有一个扩展 metrics-healthchecks 模块，用于处理健康检查。

### 4.1。Maven 依赖关系

为了使用 metrics-healthchecks 模块，我们需要将这个依赖项添加到`pom.xml`文件中:

```java
<dependency>
    <groupId>io.dropwizard.metrics</groupId>
    <artifactId>metrics-healthchecks</artifactId>
    <version>3.1.2</version>
</dependency>
```

你可以在这里找到它的最新版本。

### 4.2。用途

首先，我们需要几个负责特定健康检查操作的类，这些类必须实现`HealthCheck`。

例如，我们使用`DatabaseHealthCheck`和`UserCenterHealthCheck`:

```java
public class DatabaseHealthCheck extends HealthCheck {

    @Override
    protected Result check() throws Exception {
        return Result.healthy();
    }
} 
```

```java
public class UserCenterHealthCheck extends HealthCheck {

    @Override
    protected Result check() throws Exception {
        return Result.healthy();
    }
} 
```

然后，我们需要一个`HealthCheckRegistry`(就像`MetricRegistry`)，并用它注册`DatabaseHealthCheck`和`UserCenterHealthCheck`:

```java
HealthCheckRegistry healthCheckRegistry = new HealthCheckRegistry();
healthCheckRegistry.register("db", new DatabaseHealthCheck());
healthCheckRegistry.register("uc", new UserCenterHealthCheck());

assertThat(healthCheckRegistry.getNames().size(), equalTo(2)); 
```

我们也可以注销`HealthCheck`:

```java
healthCheckRegistry.unregister("uc");

assertThat(healthCheckRegistry.getNames().size(), equalTo(1)); 
```

我们可以运行所有的`HealthCheck`实例:

```java
Map<String, HealthCheck.Result> results = healthCheckRegistry.runHealthChecks();
for (Map.Entry<String, HealthCheck.Result> entry : results.entrySet()) {
    assertThat(entry.getValue().isHealthy(), equalTo(true));
} 
```

最后，我们可以运行一个特定的`HealthCheck`实例:

```java
healthCheckRegistry.runHealthCheck("db"); 
```

## 5。模块`metrics-servlets`

Metrics 为我们提供了一些有用的 servlets，允许我们通过 HTTP 请求访问与 metrics 相关的数据。

### 5.1。Maven 依赖关系

要使用 metrics-servlet 模块，我们需要将这个依赖项添加到`pom.xml`文件中:

```java
<dependency>
    <groupId>io.dropwizard.metrics</groupId>
    <artifactId>metrics-servlets</artifactId>
    <version>3.1.2</version>
</dependency>
```

你可以在这里找到它的最新版本。

### 5.2。`HealthCheckServlet`用法

`HealthCheckServlet`提供健康检查结果。首先，我们需要创建一个公开我们的`HealthCheckRegistry`的`ServletContextListener`:

```java
public class MyHealthCheckServletContextListener
  extends HealthCheckServlet.ContextListener {

    public static HealthCheckRegistry HEALTH_CHECK_REGISTRY
      = new HealthCheckRegistry();

    static {
        HEALTH_CHECK_REGISTRY.register("db", new DatabaseHealthCheck());
    }

    @Override
    protected HealthCheckRegistry getHealthCheckRegistry() {
        return HEALTH_CHECK_REGISTRY;
    }
} 
```

然后，我们将这个监听器和`HealthCheckServlet`添加到`web.xml`文件中:

```java
<listener>
    <listener-class>com.baeldung.metrics.servlets.MyHealthCheckServletContextListener</listener-class>
</listener>
<servlet>
    <servlet-name>healthCheck</servlet-name>
    <servlet-class>com.codahale.metrics.servlets.HealthCheckServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>healthCheck</servlet-name>
    <url-pattern>/healthcheck</url-pattern>
</servlet-mapping>
```

现在，我们可以启动 web 应用程序，并向“http://localhost:8080/health check”发送 GET 请求来获取健康检查结果。它的反应应该是这样的:

```java
{
  "db": {
    "healthy": true
  }
}
```

### 5.3。 `ThreadDumpServlet` 用法

提供关于 JVM 中所有活动线程、它们的状态、它们的堆栈跟踪以及它们可能正在等待的任何锁的状态的信息。
如果我们想使用它，我们只需将这些添加到`web.xml`文件中:

```java
<servlet>
    <servlet-name>threadDump</servlet-name>
    <servlet-class>com.codahale.metrics.servlets.ThreadDumpServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>threadDump</servlet-name>
    <url-pattern>/threaddump</url-pattern>
</servlet-mapping>
```

线程转储数据将在“http://localhost:8080/thread dump”中提供。

### 5.4。`PingServlet` 用法

`PingServlet`可用于测试应用程序是否正在运行。我们将这些添加到`web.xml`文件中:

```java
<servlet>
    <servlet-name>ping</servlet-name>
    <servlet-class>com.codahale.metrics.servlets.PingServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>ping</servlet-name>
    <url-pattern>/ping</url-pattern>
</servlet-mapping>
```

然后发送一个 GET 请求到“http://localhost:8080/ping”。响应的状态码是 200，内容是“pong”。

### 5.5。`MetricsServlet` 用法

`MetricsServlet`提供指标数据。首先，我们需要创建一个公开我们的`MetricRegistry`的`ServletContextListener`:

```java
public class MyMetricsServletContextListener
  extends MetricsServlet.ContextListener {
    private static MetricRegistry METRIC_REGISTRY
     = new MetricRegistry();

    static {
        Counter counter = METRIC_REGISTRY.counter("m01-counter");
        counter.inc();

        Histogram histogram = METRIC_REGISTRY.histogram("m02-histogram");
        histogram.update(5);
        histogram.update(20);
        histogram.update(100);
    }

    @Override
    protected MetricRegistry getMetricRegistry() {
        return METRIC_REGISTRY;
    }
} 
```

这个监听器和`MetricsServlet`都需要添加到`web.xml`中:

```java
<listener>
    <listener-class>com.codahale.metrics.servlets.MyMetricsServletContextListener</listener-class>
</listener>
<servlet>
    <servlet-name>metrics</servlet-name>
    <servlet-class>com.codahale.metrics.servlets.MetricsServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>metrics</servlet-name>
    <url-pattern>/metrics</url-pattern>
</servlet-mapping>
```

这将在我们的 web 应用程序“http://localhost:8080/metrics”中公开。它的响应应该包含各种度量数据:

```java
{
  "version": "3.0.0",
  "gauges": {},
  "counters": {
    "m01-counter": {
      "count": 1
    }
  },
  "histograms": {
    "m02-histogram": {
      "count": 3,
      "max": 100,
      "mean": 41.66666666666666,
      "min": 5,
      "p50": 20,
      "p75": 100,
      "p95": 100,
      "p98": 100,
      "p99": 100,
      "p999": 100,
      "stddev": 41.69998667732268
    }
  },
  "meters": {},
  "timers": {}
} 
```

### 5.6。`AdminServlet` 用法

`AdminServlet`聚集`HealthCheckServlet`、`ThreadDumpServlet`、`MetricsServlet`、`PingServlet`。

让我们将这些添加到`web.xml`中:

```java
<servlet>
    <servlet-name>admin</servlet-name>
    <servlet-class>com.codahale.metrics.servlets.AdminServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>admin</servlet-name>
    <url-pattern>/admin/*</url-pattern>
</servlet-mapping>
```

现在可以在“http://localhost:8080/admin”上访问它。我们将得到一个包含四个链接的页面，这四个 servlets 各有一个链接。

注意，如果我们想要进行健康检查和访问度量数据，仍然需要这两个侦听器。

## 6。模块`metrics-servlet`

`metrics-servlet`模块提供了一个`Filter`,它有几个指标:状态代码的计数器、活动请求数量的计数器和请求持续时间的计时器。

### 6.1。Maven 依赖关系

要使用这个模块，让我们首先将依赖关系添加到`pom.xml`中:

```java
<dependency>
    <groupId>io.dropwizard.metrics</groupId>
    <artifactId>metrics-servlet</artifactId>
    <version>3.1.2</version>
</dependency>
```

你可以在这里找到它的最新版本。

### 6.2。用途

要使用它，我们需要创建一个`ServletContextListener`，它将我们的`MetricRegistry`暴露给`InstrumentedFilter`:

```java
public class MyInstrumentedFilterContextListener
  extends InstrumentedFilterContextListener {

    public static MetricRegistry REGISTRY = new MetricRegistry();

    @Override
    protected MetricRegistry getMetricRegistry() {
        return REGISTRY;
    }
} 
```

然后，我们将这些添加到`web.xml`:

```java
<listener>
     <listener-class>
         com.baeldung.metrics.servlet.MyInstrumentedFilterContextListener
     </listener-class>
</listener>

<filter>
    <filter-name>instrumentFilter</filter-name>
    <filter-class>
        com.codahale.metrics.servlet.InstrumentedFilter
    </filter-class>
</filter>
<filter-mapping>
    <filter-name>instrumentFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

现在`InstrumentedFilter`可以工作了。如果我们想访问它的度量数据，我们可以通过它的`MetricRegistry` `REGISTRY`来完成。

## 7 .**。其他模块**

除了我们上面介绍的模块之外，Metrics 还有一些用于不同目的的其他模块:

*   `metrics-jvm`:为检测 JVM 内部提供了几个有用的指标
*   `metrics-ehcache`:提供`InstrumentedEhcache`，一个 Ehcache 缓存的装饰器
*   `metrics-httpclient`:提供用于检测 Apache HttpClient (4.x 版本)的类
*   `metrics-log4j`:提供`InstrumentedAppender`，一个 log4j 1.x 的 Log4j `Appender`实现，它按照事件的日志级别记录日志事件的比率
*   `metrics-log4j2`:类似于 metrics-log4j，只是针对 log4j 2.x
*   `metrics-logback`:提供`InstrumentedAppender`，一个回退`Appender`实现，它按照事件的日志级别记录日志事件的比率
*   `metrics-json`:为杰克森提供`HealthCheckModule`和`MetricsModule`

更重要的是，除了这些主要的项目模块，其他一些第三方库提供了与其他库和框架的集成。

## 8。结论

插装应用程序是一个常见的需求，因此在本文中，我们引入了度量标准，希望它能帮助您解决问题。

与往常一样，该示例的完整源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220812052521/https://github.com/eugenp/tutorials/tree/master/metrics)