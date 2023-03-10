# 石英简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/quartz>

## 1。概述

**[Quartz](https://web.archive.org/web/20220626204627/http://www.quartz-scheduler.org/)** 是一个完全用 Java 编写的开源作业调度框架，设计用于`J2SE`和`J2EE`应用程序。**它在不牺牲简单性的前提下提供了极大的灵活性。**

您可以为执行任何作业创建复杂的计划。例如，每天、每隔一周的星期五下午 7:30 或仅在每月的最后一天运行的任务。

在本文中，我们将了解使用 Quartz API 构建作业的元素。对于结合 Spring 的介绍，我们推荐用石英在 Spring 中[调度。](/web/20220626204627/https://www.baeldung.com/spring-quartz-schedule)

## 2。Maven 依赖关系

我们需要将下面的依赖项添加到`pom.xml:`

```java
<dependency>
    <groupId>org.quartz-scheduler</groupId>
    <artifactId>quartz</artifactId>
    <version>2.3.0</version>
</dependency>
```

最新版本可以在 [Maven 中央存储库](https://web.archive.org/web/20220626204627/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.quartz-scheduler%22%20AND%20a%3A%22quartz%22)中找到。

## 3。石英 API

框架的核心是`Scheduler`。它负责管理我们应用程序的运行时环境。

为了确保可扩展性，Quartz 基于多线程架构。**启动时，框架初始化一组工人线程**，由`Scheduler`用来执行`Jobs`。

这就是框架可以并发运行许多`Jobs`的方式。它还依赖一组松散耦合的`ThreadPool`管理组件来管理线程环境。

API 的关键接口是:

*   用于与框架的调度程序交互的主要 API
*   由我们希望执行的组件实现的接口
*   `JobDetail –`用于定义`Job`的实例
*   `Trigger –`确定给定的`Job`将被执行的时间表的组件
*   `JobBuilder –`用于构建`JobDetail`实例，定义`Jobs`的实例
*   `TriggerBuilder –`用于构建`Trigger`实例

让我们来看看其中的每一个组件。

## 4。调度器

在我们使用`Scheduler`之前，它需要被实例化。为此，我们可以使用工厂`SchedulerFactory**:**`

```java
SchedulerFactory schedulerFactory = new StdSchedulerFactory();
Scheduler scheduler = schedulerFactory.getScheduler();
```

通过一个`SchedulerFactory` 和对其`shutdown()`方法的调用，`Scheduler`的生命周期受到其创建的限制。一旦创建了`Scheduler`接口，就可以用来添加、删除和列出`Jobs`和`Triggers`，并执行其他与调度相关的操作(比如暂停一个触发器)。

然而，**`Scheduler`不会对任何触发器起作用，直到它被`start()`方法**启动:

```java
scheduler.start();
```

## 5。乔布斯

一个`Job` 是实现`Job` 接口的类。它只有一个简单的方法:

```java
public class SimpleJob implements Job {
    public void execute(JobExecutionContext arg0) throws JobExecutionException {
        System.out.println("This is a quartz job!");
    }
}
```

当`Job’s`触发器触发时，`execute()`方法被调度器的一个工作线程调用。

传递给该方法的`JobExecutionContext`对象为作业实例提供了关于其运行时环境的信息、执行它的`Scheduler`的句柄、触发执行的`Trigger`的句柄、作业的`JobDetail`对象和一些其他项目。

在将`Job`添加到`Scheduler.` 时，Quartz 客户端创建了`JobDetail`对象，它本质上是作业实例 *:* 的定义

```java
JobDetail job = JobBuilder.newJob(SimpleJob.class)
  .withIdentity("myJob", "group1")
  .build();
```

这个对象可能还包含各种针对`Job`的属性设置，以及一个`JobDataMap`，它可以用来存储我们的作业类的给定实例的状态信息。

### 5.1。`JobDataMap`

`JobDataMap`用于保存任何数量的数据对象，当作业实例执行时，我们希望这些数据对象对作业实例可用。`JobDataMap`是 Java `Map`接口的一个实现，增加了一些方便的方法来存储和检索原始类型的数据。

下面是一个例子，在将任务添加到调度程序之前，在构建`JobDetail`时将数据放入`JobDataMap`:

```java
JobDetail job = newJob(SimpleJob.class)
  .withIdentity("myJob", "group1")
  .usingJobData("jobSays", "Hello World!")
  .usingJobData("myFloatValue", 3.141f)
  .build();
```

下面是一个在作业执行期间如何访问这些数据的示例:

```java
public class SimpleJob implements Job { 
    public void execute(JobExecutionContext context) throws JobExecutionException {
        JobDataMap dataMap = context.getJobDetail().getJobDataMap();

        String jobSays = dataMap.getString("jobSays");
        float myFloatValue = dataMap.getFloat("myFloatValue");

        System.out.println("Job says: " + jobSays + ", and val is: " + myFloatValue);
    } 
}
```

上面的例子将打印“Job says Hello World！，val 为 3.141”。

我们还可以将 setter 方法添加到我们的作业类中，该作业类对应于`JobDataMap.` 中的键名

如果我们这样做，Quartz 的默认`JobFactory`实现会在作业被实例化时自动调用这些 setters，从而避免了在我们的 execute 方法中从映射中显式获取值的需要。

## 6。触发器

`Trigger`对象用于触发`Jobs`的执行。

当我们希望调度一个`Job`时，我们需要实例化一个触发器并调整其属性来配置我们的调度需求:

```java
Trigger trigger = TriggerBuilder.newTrigger()
  .withIdentity("myTrigger", "group1")
  .startNow()
  .withSchedule(SimpleScheduleBuilder.simpleSchedule()
    .withIntervalInSeconds(40)
    .repeatForever())
  .build();
```

一个`Trigger`也可能有一个`JobDataMap`与之相关联。这对于将特定于触发器执行的参数传递给`Job`非常有用。

不同的调度需求有不同类型的触发器。每个人都有不同的`TriggerKey`属性来跟踪他们的身份。但是，其他一些属性是所有触发器类型共有的:

*   `jobKey`属性表示当触发器触发时应该执行的作业的身份。
*   `startTime`属性表示触发器的时间表何时开始生效。该值是一个`java.util.Date`对象，它为给定的日历日期定义了一个时刻。对于某些触发器类型，触发器在给定的开始时间触发。对于其他人来说，它只是标记了时间表应该开始的时间。
*   `endTime`属性指示何时应该取消触发器的调度。

石英船有少数不同的触发器类型，但最常用的是`SimpleTrigger`和`CronTrigger`T3**。**

### 6.1。优先级

有时，当我们有许多触发器时，Quartz 可能没有足够的资源来立即触发所有计划在同一时间触发的任务。在这种情况下，我们可能希望控制哪个触发器先可用。这正是触发器上的`priority`属性的用途。

**例如**，当十个触发器被设置为同时触发，而只有四个工作线程可用时，优先级最高的前四个触发器将首先执行。当我们不对触发器设置优先级时，它使用默认的优先级 5。任何整数值都允许作为优先级，无论是正数还是负数。

在下面的例子中，我们有两个优先级不同的触发器。如果没有足够的资源同时触发所有触发器，`triggerA` 将是第一个被触发的:

```java
Trigger triggerA = TriggerBuilder.newTrigger()
  .withIdentity("triggerA", "group1")
  .startNow()
  .withPriority(15)
  .withSchedule(SimpleScheduleBuilder.simpleSchedule()
    .withIntervalInSeconds(40)
    .repeatForever())
  .build();

Trigger triggerB = TriggerBuilder.newTrigger()
  .withIdentity("triggerB", "group1")
  .startNow()
  .withPriority(10)
  .withSchedule(SimpleScheduleBuilder.simpleSchedule()
    .withIntervalInSeconds(20)
    .repeatForever())
  .build();
```

### 6.2。失火指示

**如果由于`Scheduler`被关闭，或者在 Quartz 的线程池中没有可用线程的情况下，持续触发器`misses`达到其触发时间，则会发生失火。**

不同的触发类型有不同的缺火指示。默认情况下，它们使用智能策略指令。当调度程序启动时，它会搜索任何未启动的持久性触发器。之后，它会根据各自配置的失火指令更新它们。

让我们看看下面的例子:

```java
Trigger misFiredTriggerA = TriggerBuilder.newTrigger()
  .startAt(DateUtils.addSeconds(new Date(), -10))
  .build();

Trigger misFiredTriggerB = TriggerBuilder.newTrigger()
  .startAt(DateUtils.addSeconds(new Date(), -10))
  .withSchedule(SimpleScheduleBuilder.simpleSchedule()
    .withMisfireHandlingInstructionFireNow())
  .build();
```

我们已经安排触发器在 10 秒钟前运行(因此它的创建时间晚了 10 秒钟)来模拟一个失败，例如，因为调度程序关闭或没有足够数量的工作线程可用。当然，在现实世界中，我们永远不会像这样安排触发器。

在第一次触发(`misFiredTriggerA`)中，没有设置失火处理指令。因此，在这种情况下使用了一个称为*的智能策略*，称为:`withMisfireHandlingInstructionFireNow().`这意味着在调度程序发现失火后立即执行作业。

第二个触发器明确地定义了当缺火发生时我们所期望的行为。在这个例子中，它恰好是相同的智能策略。

### 6.3。`SimpleTrigger`

**`SimpleTrigger`用于我们需要在特定时刻执行作业的场景。**这可以是一次，也可以是每隔一段时间重复一次。

例如，在 2018 年 1 月 13 日上午 12:20:00 启动作业执行。同样，我们可以从那时开始，然后再进行五次，每十秒钟一次。

在下面的代码中，日期`myStartTime` 已经预先定义好了，用于为一个特定的时间戳 **:** 构建触发器

```java
SimpleTrigger trigger = (SimpleTrigger) TriggerBuilder.newTrigger()
  .withIdentity("trigger1", "group1")
  .startAt(myStartTime)
  .forJob("job1", "group1")
  .build();
```

接下来，让我们为特定的时刻建立一个触发器，然后每十秒钟重复十次:

```java
SimpleTrigger trigger = (SimpleTrigger) TriggerBuilder.newTrigger()
  .withIdentity("trigger2", "group1")
  .startAt(myStartTime)
  .withSchedule(simpleSchedule()
    .withIntervalInSeconds(10)
    .withRepeatCount(10))
  .forJob("job1") 
  .build();
```

### 6.4。`CronTrigger`

**当我们需要基于类似日历的报表的时间表时，使用`CronTrigger`。**例如，我们可以指定发射计划，如`every Friday at noon`或`every weekday at 9:30 am`。

Cron 表达式用于配置`CronTrigger`的实例。这些表达式由七个子表达式组成的`Strings`组成。我们可以在这里阅读更多关于克隆表情[的内容。](https://web.archive.org/web/20220626204627/https://docs.oracle.com/cd/E12058_01/doc/doc.1014/e12030/cron_expressions.htm)

在下面的示例中，我们构建了一个触发器，在每天上午 8 点到下午 5 点之间每隔一分钟触发一次:

```java
CronTrigger trigger = TriggerBuilder.newTrigger()
  .withIdentity("trigger3", "group1")
  .withSchedule(CronScheduleBuilder.cronSchedule("0 0/2 8-17 * * ?"))
  .forJob("myJob", "group1")
  .build();
```

## 7。结论

在本文中，我们展示了如何构建一个`Scheduler`来触发一个`Job`。我们还看到了一些最常用的触发选项:`SimpleTrigger`和`CronTrigger`。

Quartz 可以用来创建简单或复杂的调度，以执行几十个、几百个甚至更多的任务。关于该框架的更多信息可以在主[网站](https://web.archive.org/web/20220626204627/http://www.quartz-scheduler.org/)上找到。

这些例子的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220626204627/https://github.com/eugenp/tutorials/tree/master/libraries/src/main/java/com/baeldung/quartz)