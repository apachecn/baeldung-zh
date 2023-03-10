# 雅加达的日程安排 EE

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/scheduling-in-java-enterprise-edition>

## 1.概观

在之前的[文章](/web/20220815030036/https://www.baeldung.com/spring-scheduled-tasks)中，我们演示了如何使用`@Scheduled` *注释在 Spring 中调度任务。在本文中，我们将展示如何通过在 Jakarta EE 应用程序中使用**计时器服务来实现相同的目的，这是上一篇文章中介绍的每种情况。***

 *## 2.启用对计划的支持

在 Jakarta EE 应用程序中，不需要启用对定时任务的支持。计时器服务是一个容器管理的服务，它允许应用程序调用为基于时间的事件安排的方法。例如，应用程序可能必须在某个小时运行一些每日报告，以便生成统计数据。

有两种类型的计时器:

*   **编程定时器**:定时器服务可以注入到任何 bean 中(除了有状态会话 bean)，业务逻辑应该放在用`@Timeout`注释的方法中。计时器可以通过 beans 的注释方法`@PostConstruct`来初始化，也可以通过调用方法来初始化。

*   **自动定时器**:业务逻辑放在任何标注有`@Schedule`或`@Schedules`的方法中。一旦应用程序启动，这些定时器就会被初始化。

让我们从第一个例子开始。

## 3.以固定延迟调度任务

在 Spring 中，这可以简单地通过使用`@Scheduled(fixedDelay = 1000)` 注释来完成。在这种情况下，上一次执行结束和下一次执行开始之间的持续时间是固定的。任务总是等到前一个任务完成。

在 Jakarta EE 中做完全相同的事情有点困难，因为没有提供类似的内置机制，然而，通过一点额外的编码就可以实现类似的场景。让我们来看看这是如何做到的:

```java
@Singleton
public class FixedTimerBean {

    @EJB
    private WorkerBean workerBean;

    @Lock(LockType.READ)
    @Schedule(second = "*/5", minute = "*", hour = "*", persistent = false)
    public void atSchedule() throws InterruptedException {
        workerBean.doTimerWork();
    }
} 
```

```java
@Singleton
public class WorkerBean {

    private AtomicBoolean busy = new AtomicBoolean(false);

    @Lock(LockType.READ)
    public void doTimerWork() throws InterruptedException {
        if (!busy.compareAndSet(false, true)) {
            return;
        }
        try {
            Thread.sleep(20000L);
        } finally {
            busy.set(false);
        }
    }
}
```

正如你所看到的，计时器被安排为每五秒触发一次。然而，在我们的例子中触发的方法通过调用当前`Thread`上的`sleep()`模拟了 20 秒的响应时间。

因此，容器将继续每五秒钟调用一次`doTimerWork()`,但是如果前一次调用没有完成，那么方法开始处的条件`busy.compareAndSet(false, true),`将立即返回。在这种情况下，我们确保只有在前一个任务完成后，才会执行下一个任务。

## 4.以固定速率调度任务

一种方法是使用[定时器服务](https://web.archive.org/web/20220815030036/https://docs.oracle.com/javaee/6/api/javax/ejb/TimerService.html)，它是通过使用`@Resource`注入的，并在方法注释`@PostConstruct`中配置。用`@Timeout`标注的方法将在定时器到期时被调用。

正如在上一篇文章中提到的，任务执行的开始并不等待上一次执行的完成。当任务的每次执行都是独立的时，应该使用此选项。下面的代码片段创建了一个每秒触发一次的计时器:

```java
@Startup
@Singleton
public class ProgrammaticAtFixedRateTimerBean {

    @Inject
    Event<TimerEvent> event;

    @Resource
    TimerService timerService;

    @PostConstruct
    public void initialize() {
        timerService.createTimer(0,1000, "Every second timer with no delay");
    }

    @Timeout
    public void programmaticTimout(Timer timer) {
        event.fire(new TimerEvent(timer.getInfo().toString()));
    }
}
```

另一种方法是使用`@Scheduled`标注。在下面的代码片段中，我们每五秒钟触发一次计时器:

```java
@Startup
@Singleton
public class ScheduleTimerBean {

    @Inject
    Event<TimerEvent> event;

    @Schedule(hour = "*", minute = "*", second = "*/5", info = "Every 5 seconds timer")
    public void automaticallyScheduled(Timer timer) {
        fireEvent(timer);
    }

    private void fireEvent(Timer timer) {
        event.fire(new TimerEvent(timer.getInfo().toString()));
    }
}
```

## 5.调度初始延迟的任务

如果您的用例场景要求定时器延迟启动，我们也可以这样做。在这种情况下，Jakarta EE 允许使用[定时服务](https://web.archive.org/web/20220815030036/https://docs.oracle.com/javaee/6/api/javax/ejb/TimerService.html)。让我们看一个例子，计时器初始延迟 10 秒，然后每隔 5 秒触发一次:

```java
@Startup
@Singleton
public class ProgrammaticWithInitialFixedDelayTimerBean {

    @Inject
    Event<TimerEvent> event;

    @Resource
    TimerService timerService;

    @PostConstruct
    public void initialize() {
        timerService.createTimer(10000, 5000, "Delay 10 seconds then every 5 seconds timer");
    }

    @Timeout
    public void programmaticTimout(Timer timer) {
        event.fire(new TimerEvent(timer.getInfo().toString()));
    }
}
```

我们示例中使用的`createTimer`方法使用下面的签名 c `reateTimer(long initialDuration, long intervalDuration, java.io.Serializable info)`，其中`initialDuration`是第一个定时器到期通知之前必须经过的毫秒数，`intervalDuration`是定时器到期通知之间必须经过的毫秒数。

在这个例子中，我们使用 10 秒的`initialDuration`和 5 秒的`intervalDuration`。任务将在`initialDuration`值后第一次执行，并根据`intervalDuration`继续执行。

## 6.使用 Cron 表达式调度任务

我们看到的所有调度器，无论是编程的还是自动的，都允许使用 cron 表达式。让我们看一个例子:

```java
@Schedules ({
   @Schedule(dayOfMonth="Last"),
   @Schedule(dayOfWeek="Fri", hour="23")
})
public void doPeriodicCleanup() { ... }
```

在这个例子中，方法`doPeriodicCleanup()`将在每个星期五的 23:00 和这个月的最后一天被调用。

## 7.结论

在本文中，我们研究了在 Jakarta EE 环境中调度任务的各种方法，以前的一篇文章使用 Spring 完成了一些示例。

代码示例可以在 [GitHub 库](https://web.archive.org/web/20220815030036/https://github.com/eugenp/tutorials/tree/master/jee-7)中找到。*