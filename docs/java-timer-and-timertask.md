# Java 计时器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-timer-and-timertask>

## 1。计时器——基础知识

*Timer* 和`TimerTask`是 java util 类，我们用它们在后台线程中调度任务。基本上， **[*TimerTask*](https://web.archive.org/web/20220902075847/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/TimerTask.html "The Javadoc") 是要执行的任务， [*Timer*](https://web.archive.org/web/20220902075847/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Timer.html "The Javadoc") 是调度器**。

## 2。安排一次任务

### 2.1.在给定的延迟之后

让我们从简单的**在`Timer`的帮助下运行单个任务**开始:

```java
@Test
public void givenUsingTimer_whenSchedulingTaskOnce_thenCorrect() {
    TimerTask task = new TimerTask() {
        public void run() {
            System.out.println("Task performed on: " + new Date() + "n" +
              "Thread's name: " + Thread.currentThread().getName());
        }
    };
    Timer timer = new Timer("Timer");

    long delay = 1000L;
    timer.schedule(task, delay);
}
```

**这在一定的延迟**后执行任务，这是我们给定的`schedule()`方法的第二个参数。在下一节中，我们将了解如何在给定的日期和时间安排任务。

注意，如果我们将此作为 Junit 测试运行，我们应该添加一个 *Thread.sleep(delay * 2)* 调用，以允许计时器的线程在 JUnit 测试停止执行之前运行任务。

### 2.2.在给定的日期和时间

现在让我们看一下`Timer#schedule(TimerTask, Date)`方法，它采用一个`Date`而不是一个`long`作为它的第二个参数。这允许我们在某个时刻安排任务，而不是在延迟之后。

这一次，让我们假设我们有一个旧的遗留数据库，我们希望将它的数据迁移到一个具有更好模式的新数据库中。

我们可以创建一个`DatabaseMigrationTask`类来处理这个迁移:

```java
public class DatabaseMigrationTask extends TimerTask {
    private List<String> oldDatabase;
    private List<String> newDatabase;

    public DatabaseMigrationTask(List<String> oldDatabase, List<String> newDatabase) {
        this.oldDatabase = oldDatabase;
        this.newDatabase = newDatabase;
    }

    @Override
    public void run() {
        newDatabase.addAll(oldDatabase);
    }
}
```

为了简单起见，我们用一个`String`的`List`来表示这两个数据库。简单地说，我们的迁移包括将第一个列表中的数据放入第二个列表中。

为了在期望的时刻执行迁移，**我们必须使用重载版本的`schedule` `()`方法**:

```java
List<String> oldDatabase = Arrays.asList("Harrison Ford", "Carrie Fisher", "Mark Hamill");
List<String> newDatabase = new ArrayList<>();

LocalDateTime twoSecondsLater = LocalDateTime.now().plusSeconds(2);
Date twoSecondsLaterAsDate = Date.from(twoSecondsLater.atZone(ZoneId.systemDefault()).toInstant());

new Timer().schedule(new DatabaseMigrationTask(oldDatabase, newDatabase), twoSecondsLaterAsDate);
```

正如我们所看到的，我们将迁移任务以及执行日期交给了`schedule()`方法。

然后在由`twoSecondsLater`指示的时间执行迁移:

```java
while (LocalDateTime.now().isBefore(twoSecondsLater)) {
    assertThat(newDatabase).isEmpty();
    Thread.sleep(500);
}
assertThat(newDatabase).containsExactlyElementsOf(oldDatabase);
```

在那之前，迁移不会发生。

## 3.计划可重复的任务

既然我们已经介绍了如何调度任务的单次执行，那么让我们看看如何处理可重复的任务。

再一次，`Timer`类提供了多种可能性。我们可以设置重复来观察固定延迟或固定速率。

**固定延迟意味着执行将在最后一次执行开始后的一段时间后开始，即使它被延迟(因此它本身被延迟)**。

假设我们希望每两秒调度一个任务，第一次执行用了一秒，第二次执行用了两秒，但是延迟了一秒。然后第三次执行在第五秒开始:

```java
0s     1s    2s     3s           5s
|--T1--|
|-----2s-----|--1s--|-----T2-----|
|-----2s-----|--1s--|-----2s-----|--T3--|
```

另一方面，**固定的速率意味着每次执行都将遵守初始时间表，而不管前一次执行是否被延迟**。

让我们重复使用之前的例子。在固定速率下，第二个任务将在三秒钟后开始(由于延迟)，但第三个任务将在四秒钟后开始(考虑到每两秒钟执行一次的初始计划):

```java
0s     1s    2s     3s    4s
|--T1--|       
|-----2s-----|--1s--|-----T2-----|
|-----2s-----|-----2s-----|--T3--|
```

现在我们已经讨论了这两个原则，让我们看看如何使用它们。

**为了使用固定延迟调度，`schedule()`方法还有两个重载，每个重载都带有一个额外的参数，以毫秒为单位表示周期。**

为什么两次超载？因为仍然有可能在某个时刻或某个延迟之后开始任务。

**对于固定速率调度，我们有两个`scheduleAtFixedRate()`方法，也是以毫秒为单位取周期性。**同样，我们有一种方法在给定的日期和时间启动任务，另一种方法在给定的延迟后启动任务。

同样值得一提的是，如果一个任务的执行时间超过了周期，那么它会延迟整个执行链，不管我们使用的是固定延迟还是固定速率。

### 3.1.具有固定的延迟

现在，让我们假设我们想要实现一个简讯系统，每周向我们的追随者发送一封电子邮件。在这种情况下，重复的任务似乎是理想的。

因此，让我们计划每一秒的简讯，这基本上是垃圾邮件，但由于发送是假的，我们准备好了。

首先，我们将设计一个`NewsletterTask`:

```java
public class NewsletterTask extends TimerTask {
    @Override
    public void run() {
        System.out.println("Email sent at: " 
          + LocalDateTime.ofInstant(Instant.ofEpochMilli(scheduledExecutionTime()), 
          ZoneId.systemDefault()));
    }
}
```

每次执行时，任务将打印它的预定时间，这是我们使用`TimerTask#scheduledExecutionTime()`方法收集的。

那么如果我们想在固定延迟模式下每秒调度一次这个任务呢？我们将不得不使用我们之前提到的`schedule()`的重载版本:

```java
new Timer().schedule(new NewsletterTask(), 0, 1000);

for (int i = 0; i < 3; i++) {
    Thread.sleep(1000);
}
```

当然，我们只对少数情况进行测试:

```java
Email sent at: 2020-01-01T10:50:30.860
Email sent at: 2020-01-01T10:50:31.860
Email sent at: 2020-01-01T10:50:32.861
Email sent at: 2020-01-01T10:50:33.861
```

正如我们所见，每次执行之间至少有一秒钟，但有时会延迟一毫秒。这种现象是由于我们决定使用固定延迟重复。

### 3.2.采用固定汇率

如果我们使用固定速率的重复会怎么样？那么我们将不得不使用`scheduledAtFixedRate()`方法:

```java
new Timer().scheduleAtFixedRate(new NewsletterTask(), 0, 1000);

for (int i = 0; i < 3; i++) {
    Thread.sleep(1000);
}
```

这一次，**的执行不会被之前的**延迟:

```java
Email sent at: 2020-01-01T10:55:03.805
Email sent at: 2020-01-01T10:55:04.805
Email sent at: 2020-01-01T10:55:05.805
Email sent at: 2020-01-01T10:55:06.805
```

### 3.3.安排日常任务

接下来，让我们**每天运行一次任务**:

```java
@Test
public void givenUsingTimer_whenSchedulingDailyTask_thenCorrect() {
    TimerTask repeatedTask = new TimerTask() {
        public void run() {
            System.out.println("Task performed on " + new Date());
        }
    };
    Timer timer = new Timer("Timer");

    long delay = 1000L;
    long period = 1000L * 60L * 60L * 24L;
    timer.scheduleAtFixedRate(repeatedTask, delay, period);
}
```

## 4。取消`Timer`和`TimerTask`和

有几种方法可以取消任务的执行。

### 4.1。取消`Run`内的`TimerTask`和

第一个选项是调用 *run()* 方法实现 *TimerTask* 本身内部的 *TimerTask.cancel()* 方法:

```java
@Test
public void givenUsingTimer_whenCancelingTimerTask_thenCorrect()
  throws InterruptedException {
    TimerTask task = new TimerTask() {
        public void run() {
            System.out.println("Task performed on " + new Date());
            cancel();
        }
    };
    Timer timer = new Timer("Timer");

    timer.scheduleAtFixedRate(task, 1000L, 1000L);

    Thread.sleep(1000L * 2);
}
```

### 4.2。取消`Timer`

另一个选项是在一个*定时器*对象上调用*定时器. cancel()* 方法:

```java
@Test
public void givenUsingTimer_whenCancelingTimer_thenCorrect() 
  throws InterruptedException {
    TimerTask task = new TimerTask() {
        public void run() {
            System.out.println("Task performed on " + new Date());
        }
    };
    Timer timer = new Timer("Timer");

    timer.scheduleAtFixedRate(task, 1000L, 1000L);

    Thread.sleep(1000L * 2); 
    timer.cancel(); 
}
```

### 4.3。停止`Run`内`TimerTask`的线程

我们还可以停止任务的`run`方法中的线程，从而取消整个任务:

```java
@Test
public void givenUsingTimer_whenStoppingThread_thenTimerTaskIsCancelled() 
  throws InterruptedException {
    TimerTask task = new TimerTask() {
        public void run() {
            System.out.println("Task performed on " + new Date());
            // TODO: stop the thread here
        }
    };
    Timer timer = new Timer("Timer");

    timer.scheduleAtFixedRate(task, 1000L, 1000L);

    Thread.sleep(1000L * 2); 
}
```

注意`run`实现中的 TODO 指令；为了运行这个简单的例子，我们需要实际停止线程。

在真实的自定义线程实现中，应该支持停止线程，但在这种情况下，我们可以忽略这种反对，并在线程类本身上使用简单的`stop` API。

## 5。`Timer`vs`ExecutorService`

我们还可以很好地利用 ExecutorService 来调度定时器任务，而不是使用定时器。

下面是如何以指定的时间间隔运行重复任务的快速示例:

```java
@Test
public void givenUsingExecutorService_whenSchedulingRepeatedTask_thenCorrect() 
  throws InterruptedException {
    TimerTask repeatedTask = new TimerTask() {
        public void run() {
            System.out.println("Task performed on " + new Date());
        }
    };
    ScheduledExecutorService executor = Executors.newSingleThreadScheduledExecutor();
    long delay  = 1000L;
    long period = 1000L;
    executor.scheduleAtFixedRate(repeatedTask, delay, period, TimeUnit.MILLISECONDS);
    Thread.sleep(delay + period * 3);
    executor.shutdown();
}
```

那么`Timer`和`ExecutorService`解决方案的主要区别是什么:

*   *定时器*可以对系统时钟的变化敏感；*ScheduledThreadPoolExecutor*不是。
*   *定时器*只有一个执行线程；`ScheduledThreadPoolExecutor`可以配置任意数量的线程。
*   在 *TimerTask* 内部抛出的运行时异常会杀死线程，所以下面的调度任务不会进一步运行；使用 *ScheduledThreadExecutor，*当前任务将被取消，但其余任务将继续运行。

## 6。结论

在本文中，我们展示了利用 Java 内置的简单而灵活的`Timer`和`TimerTask` 基础设施快速调度任务的许多方法。当然，如果我们需要的话，Java 世界中还有更复杂、更完整的解决方案，比如[石英库](https://web.archive.org/web/20220902075847/http://quartz-scheduler.org/ "The Quartz Scheduler library")，但这是一个非常好的起点。

这些例子的实现可以在 GitHub 上的[中找到。](https://web.archive.org/web/20220902075847/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-time-measurements "The Timer and TimerTask examples on github")