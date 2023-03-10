# Spring 任务调度器指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-task-scheduler>

## 1.概观

在本文中，我们将讨论 **Spring 任务调度机制**–`TaskScheduler` 及其预构建的实现，以及要使用的不同触发器。如果你想了解更多关于春季日程安排的信息，请查看`[@Async](/web/20220524031354/https://www.baeldung.com/spring-async)`和`[@Scheduled](/web/20220524031354/https://www.baeldung.com/spring-scheduled-tasks)`的文章。

`TaskScheduler`是在 Spring 3.0 中引入的，有多种方法可以在未来的某个时刻运行，它还返回了一个`ScheduledFuture`接口的表示对象，可以用来取消预定的任务或者检查任务是否完成。

我们所要做的就是选择一个可运行的任务进行调度，然后选择一个合适的调度策略。

## 2.`ThreadPoolTaskScheduler`

`ThreadPoolTaskScheduler` 非常适合内部线程管理，因为它将任务委托给了`ScheduledExecutorService`并实现了`TaskExecutor` 接口——因此它的单个实例能够处理异步潜在执行以及`@Scheduled`注释。

现在让我们在`ThreadPoolTaskSchedulerConfig` :定义`ThreadPoolTaskScheduler` bean

```java
@Configuration
@ComponentScan(
  basePackages="com.baeldung.taskscheduler",
  basePackageClasses={ThreadPoolTaskSchedulerExamples.class})
public class ThreadPoolTaskSchedulerConfig {

    @Bean
    public ThreadPoolTaskScheduler threadPoolTaskScheduler(){
        ThreadPoolTaskScheduler threadPoolTaskScheduler
          = new ThreadPoolTaskScheduler();
        threadPoolTaskScheduler.setPoolSize(5);
        threadPoolTaskScheduler.setThreadNamePrefix(
          "ThreadPoolTaskScheduler");
        return threadPoolTaskScheduler;
    }
}
```

配置的 bean `threadPoolTaskScheduler` 可以基于配置的池大小 5 异步执行任务。

注意，所有与`ThreadPoolTaskScheduler` 相关的线程名称都将以`ThreadPoolTaskScheduler`为前缀。

让我们实现一个可以调度的简单任务:

```java
class RunnableTask implements Runnable{
    private String message;

    public RunnableTask(String message){
        this.message = message;
    }

    @Override
    public void run() {
        System.out.println(new Date()+" Runnable Task with "+message
          +" on thread "+Thread.currentThread().getName());
    }
} 
```

我们现在可以简单地安排这个任务由调度程序执行:

```java
taskScheduler.schedule(
  new Runnabletask("Specific time, 3 Seconds from now"),
  new Date(System.currentTimeMillis + 3000)
); 
```

`taskScheduler`将在一个已知的日期安排这个可运行的任务，正好在当前时间之后 3 秒。

现在让我们更深入地了解一下`ThreadPoolTaskScheduler` 调度机制。

## 3.调度具有固定延迟的可运行任务

具有固定延迟的调度可以通过两种简单的机制来完成:

### 3.1.在最后一次调度执行的固定延迟后进行调度

让我们将任务配置为在 1000 毫秒的固定延迟后运行:

```java
taskScheduler.scheduleWithFixedDelay(
  new RunnableTask("Fixed 1 second Delay"), 1000);
```

从一个执行的完成到下一个执行的开始，`RunnableTask` 将总是在 1000 毫秒后运行。

### 3.2.特定日期固定延迟后的调度

让我们将任务配置为在给定开始时间的固定延迟后运行:

```java
taskScheduler.scheduleWithFixedDelay(
  new RunnableTask("Current Date Fixed 1 second Delay"),
  new Date(),
  1000);
```

`RunnableTask` 将在指定的执行时间被调用，该时间主要是`@PostConstruct`方法开始的时间，随后有 1000 毫秒的延迟。

## 4.以固定速率调度

有两种简单的机制用于以固定速率调度可运行任务:

### 4.1.以固定速率调度`RunnableTask`

让我们安排一个任务以固定的毫秒速率**运行:**

```java
taskScheduler.scheduleAtFixedRate(
  new RunnableTask("Fixed Rate of 2 seconds") , 2000);
```

**下一个`RunnableTask` 将总是在 2000 毫秒后运行，而不管可能仍在运行的上一次执行的状态。**

### 4.2.从给定日期开始以固定的速率调度`RunnableTask`

```java
taskScheduler.scheduleAtFixedRate(new RunnableTask(
  "Fixed Rate of 2 seconds"), new Date(), 3000);
```

`RunnableTask` 将在当前时间后 3000 毫秒运行。

## 5.使用`CronTrigger`进行调度

`CronTrigger` 用于根据 cron 表达式调度任务:

```java
CronTrigger cronTrigger 
  = new CronTrigger("10 * * * * ?"); 
```

提供的触发器可用于根据特定的节奏或时间表运行任务:

```java
taskScheduler.schedule(new RunnableTask("Cron Trigger"), cronTrigger);
```

在这种情况下，`RunnableTask` 将在每分钟的第 10 秒执行。

## 6.使用`PeriodicTrigger`进行调度

让我们使用`PeriodicTrigger`来调度一个具有 2000 毫秒的**固定延迟**的任务:

```java
PeriodicTrigger periodicTrigger 
  = new PeriodicTrigger(2000, TimeUnit.MICROSECONDS);
```

配置的`PeriodicTrigger` bean 将用于在 2000 毫秒的固定延迟后运行任务。

现在让我们用`PeriodicTrigger`来安排`RunnableTask`:

```java
taskScheduler.schedule(
  new RunnableTask("Periodic Trigger"), periodicTrigger);
```

我们还可以配置`PeriodicTrigger` 以固定速率而不是固定延迟进行初始化，我们还可以为第一个调度任务设置一个给定毫秒的初始延迟。

我们需要做的就是在`periodicTrigger` bean **:** 的 return 语句前添加两行代码

```java
periodicTrigger.setFixedRate(true);
periodicTrigger.setInitialDelay(1000);
```

我们使用`setFixedRate`方法以固定速率而不是固定延迟来调度任务，然后使用`setInitialDelay`方法仅为第一个要运行的可运行任务设置初始延迟。

## 7.结论

在这篇简短的文章中，我们展示了如何使用 Spring 对任务的支持来调度一个可运行的任务。

我们研究了以固定的延迟、固定的速率和根据特定的触发器来运行任务。

和往常一样，这些代码在 GitHub 中作为 Maven 项目[提供。](https://web.archive.org/web/20220524031354/https://github.com/eugenp/tutorials/tree/master/spring-scheduling)