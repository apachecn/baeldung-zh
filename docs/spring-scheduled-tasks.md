# Spring 中的@Scheduled 注释

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-scheduled-tasks>

## 1。概述

在本教程中，我们将说明如何使用**Spring`@Scheduled`注解**来配置和调度任务。

用`@Scheduled`注释方法时，我们需要遵循的简单规则是:

*   该方法通常应该有一个 void 返回类型(如果没有，返回值将被忽略)
*   该方法不应该有任何参数

## 延伸阅读:

## [春天如何做@ Async](/web/20220628081929/https://www.baeldung.com/spring-async)

How to enable and use @Async in Spring - from the very simple config and basic usage to the more complex executors and exception handling strategies.[Read more](/web/20220628081929/https://www.baeldung.com/spring-async) →

## Spring 任务调度器指南

A quick and practical guide to scheduling in Spring with Task Scheduler[Read more](/web/20220628081929/https://www.baeldung.com/spring-task-scheduler) →

## [石英春季调度](/web/20220628081929/https://www.baeldung.com/spring-quartz-schedule)

Quick introduction to working with Quartz in Spring.[Read more](/web/20220628081929/https://www.baeldung.com/spring-quartz-schedule) →

## 2。启用调度支持

为了在 Spring 中启用对调度任务和`@Scheduled`注释的支持，我们可以使用 Java enable-style 注释:

```java
@Configuration
@EnableScheduling
public class SpringConfig {
    ...
}
```

相反，我们可以在 XML 中做同样的事情:

```java
<task:annotation-driven>
```

## 3。以固定延迟调度任务

让我们从配置一个在固定延迟后运行的任务开始:

```java
@Scheduled(fixedDelay = 1000)
public void scheduleFixedDelayTask() {
    System.out.println(
      "Fixed delay task - " + System.currentTimeMillis() / 1000);
}
```

在这种情况下，上一次执行结束和下一次执行开始之间的持续时间是固定的。任务总是等到前一个任务完成。

当必须在再次运行之前完成上一次执行时，应该使用此选项。

## 4。以固定的速率安排任务

现在让我们以固定的时间间隔执行一项任务:

```java
@Scheduled(fixedRate = 1000)
public void scheduleFixedRateTask() {
    System.out.println(
      "Fixed rate task - " + System.currentTimeMillis() / 1000);
}
```

当任务的每次执行都是独立的时，应该使用此选项。

请注意，默认情况下，计划任务不会并行运行。所以即使我们使用了`fixedRate`，下一个任务也不会被调用，直到前一个任务完成。

**如果我们想在调度任务中支持并行行为，我们需要添加`@Async`注释:**

```java
@EnableAsync
public class ScheduledFixedRateExample {
    @Async
    @Scheduled(fixedRate = 1000)
    public void scheduleFixedRateTaskAsync() throws InterruptedException {
        System.out.println(
          "Fixed rate task async - " + System.currentTimeMillis() / 1000);
        Thread.sleep(2000);
    }

}
```

现在这个异步任务将每秒被调用一次，即使前一个任务没有完成。

## 5.固定速率与固定延迟

我们可以使用 Spring 的`@Scheduled`注释来运行一个调度任务，但是基于属性`fixedDelay`和`fixedRate,`，执行的性质会发生变化。

**`fixedDelay`属性确保在一个任务执行的结束时间和下一个任务执行的开始时间之间有一个`n`毫秒的延迟。**

当我们需要确保始终只有一个任务实例运行时，这个属性特别有用。对于依赖型的工作，还是挺有帮助的。

**`fixedRate`属性每隔`n`毫秒运行一次计划任务。**它不检查任务之前的任何执行。

当任务的所有执行都是独立的时，这很有用。如果我们不期望超过内存和线程池的大小，`fixedRate`应该是相当方便的。

尽管如此，如果传入的任务没有很快完成，它们可能会以“内存不足异常”结束。

## 6。安排一个有初始延迟的任务

接下来，让我们调度一个延迟任务(以毫秒为单位):

```java
@Scheduled(fixedDelay = 1000, initialDelay = 1000)
public void scheduleFixedRateWithInitialDelayTask() {

    long now = System.currentTimeMillis() / 1000;
    System.out.println(
      "Fixed rate task with one second initial delay - " + now);
}
```

注意在这个例子中我们是如何同时使用`fixedDelay`和`initialDelay`的。任务将在`initialDelay`值后第一次执行，并根据`fixedDelay`继续执行。

当任务有需要完成的设置时，此选项很方便。

## 7。使用 Cron 表达式调度任务

有时延迟和速率是不够的，我们需要 cron 表达式的灵活性来控制我们任务的时间表:

```java
@Scheduled(cron = "0 15 10 15 * ?")
public void scheduleTaskUsingCronExpression() {

    long now = System.currentTimeMillis() / 1000;
    System.out.println(
      "schedule tasks using cron jobs - " + now);
}
```

请注意，在本例中，我们计划在每月 15 日上午 10:15 执行一个任务。

默认情况下，Spring 将对 cron 表达式使用服务器的本地时区。然而，**我们可以使用`zone`属性来改变这个时区**:

```java
@Scheduled(cron = "0 15 10 15 * ?", zone = "Europe/Paris")
```

有了这个配置，Spring 将安排带注释的方法在巴黎时间每月 15 日上午 10:15 运行。

## 8。参数化时间表

硬编码这些时间表很简单，但我们通常需要能够控制时间表，而无需重新编译和重新部署整个应用程序。

我们将利用 Spring 表达式来具体化任务的配置，并将它们存储在属性文件中。

一个`fixedDelay`任务:

```java
@Scheduled(fixedDelayString = "${fixedDelay.in.milliseconds}")
```

一个`fixedRate` 任务:

```java
@Scheduled(fixedRateString = "${fixedRate.in.milliseconds}")
```

基于`cron`表达式的任务:

```java
@Scheduled(cron = "${cron.expression}")
```

## 9。使用 XML 配置计划任务

Spring 还提供了一种配置调度任务的 XML 方式。下面是设置这些的 XML 配置:

```java
<!-- Configure the scheduler -->
<task:scheduler id="myScheduler" pool-size="10" />

<!-- Configure parameters -->
<task:scheduled-tasks scheduler="myScheduler">
    <task:scheduled ref="beanA" method="methodA" 
      fixed-delay="5000" initial-delay="1000" />
    <task:scheduled ref="beanB" method="methodB" 
      fixed-rate="5000" />
    <task:scheduled ref="beanC" method="methodC" 
      cron="*/5 * * * * MON-FRI" />
</task:scheduled-tasks>
```

## 10.运行时动态设置延迟或速率

通常，`@Scheduled`注释的所有属性只在 Spring 上下文启动时被解析和初始化一次。

因此，当我们在 Spring 中使用`@Scheduled`注释时，**不可能在运行时改变`fixedDelay`或`fixedRate`的值。**

但是，有一个解决方法。**使用 Spring 的`SchedulingConfigurer`提供了一种更加可定制的方式，让我们有机会动态设置延迟或速率**。

让我们创建一个弹簧配置`DynamicSchedulingConfig`，并实现`SchedulingConfigurer`接口:

```java
@Configuration
@EnableScheduling
public class DynamicSchedulingConfig implements SchedulingConfigurer {

    @Autowired
    private TickService tickService;

    @Bean
    public Executor taskExecutor() {
        return Executors.newSingleThreadScheduledExecutor();
    }

    @Override
    public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
        taskRegistrar.setScheduler(taskExecutor());
        taskRegistrar.addTriggerTask(
          new Runnable() {
              @Override
              public void run() {
                  tickService.tick();
              }
          },
          new Trigger() {
              @Override
              public Date nextExecutionTime(TriggerContext context) {
                  Optional<Date> lastCompletionTime =
                    Optional.ofNullable(context.lastCompletionTime());
                  Instant nextExecutionTime =
                    lastCompletionTime.orElseGet(Date::new).toInstant()
                      .plusMillis(tickService.getDelay());
                  return Date.from(nextExecutionTime);
              }
          }
        );
    }

}
```

正如我们注意到的，借助于`ScheduledTaskRegistrar#addTriggerTask`方法，我们可以添加一个`Runnable`任务和一个`Trigger`实现，以便在每次执行结束后重新计算`nextExecutionTime`。

此外，我们用`@EnableScheduling`来注释我们的`DynamicSchedulingConfig`,以便进行调度。

因此，我们安排`TickService#tick`方法在每次延迟后运行，这是由`getDelay`方法在运行时动态确定的。

## 11.并行运行任务

默认情况下， **Spring 使用本地单线程调度程序来运行任务**。因此，即使我们有多个`@Scheduled`方法，它们都需要等待线程完成执行前一个任务。

如果我们的任务是真正独立的，那么并行运行它们会更方便。为此，我们需要提供一个更适合我们需求的 [`TaskScheduler`](/web/20220628081929/https://www.baeldung.com/spring-task-scheduler) :

```java
@Bean
public TaskScheduler  taskScheduler() {
    ThreadPoolTaskScheduler threadPoolTaskScheduler = new ThreadPoolTaskScheduler();
    threadPoolTaskScheduler.setPoolSize(5);
    threadPoolTaskScheduler.setThreadNamePrefix("ThreadPoolTaskScheduler");
    return threadPoolTaskScheduler;
}
```

在上面的例子中，我们将`TaskScheduler`的池大小配置为 5，但是请记住，实际的配置应该根据个人的特定需求进行微调。

### 11.1.使用 Spring Boot

如果我们使用 Spring Boot，我们可以利用一种更方便的方法来增加调度程序的池大小。

设置`spring.task.scheduling.pool.size`属性:
`spring.task.scheduling.pool.size=5`就够了

## 12。结论

在本文中，我们讨论了配置和使用**注释**的方法。

我们讨论了启用调度的过程，以及配置调度任务模式的各种方法。我们还展示了动态配置延迟和速率的解决方法。

上面显示的例子可以在 GitHub 上找到[。](https://web.archive.org/web/20220628081929/https://github.com/eugenp/tutorials/tree/master/spring-scheduling)