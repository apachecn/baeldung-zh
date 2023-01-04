# Java 中基于优先级的作业调度

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-priority-job-schedule>

## 1。简介

在多线程环境中，有时我们需要根据自定义标准调度任务，而不仅仅是创建时间。

让我们看看如何在 Java 中实现这一点——使用一个`PriorityBlockingQueue`。

## 2。概述

假设我们有一些希望根据优先级执行的作业:

```
public class Job implements Runnable {
    private String jobName;
    private JobPriority jobPriority;

    @Override
    public void run() {
        System.out.println("Job:" + jobName +
          " Priority:" + jobPriority);
        Thread.sleep(1000); // to simulate actual execution time
    }

    // standard setters and getters
}
```

出于演示的目的，我们在`run()`方法中打印作业名称和优先级。

我们还添加了`sleep()` ，以便模拟一个运行时间更长的作业；当作业正在执行时，更多的作业将在优先级队列中累积。

最后，`JobPriority`是一个简单的枚举:

```
public enum JobPriority {
    HIGH,
    MEDIUM,
    LOW
}
```

## 3。`Comparator`风俗

我们需要编写一个定义自定义标准的比较器；并且，在 [Java 8 中，它是微不足道的](/web/20221126224409/https://www.baeldung.com/java-8-comparator-comparing):

```
Comparator.comparing(Job::getJobPriority);
```

## 4。优先任务调度器

完成所有设置后，现在让我们实现一个简单的作业调度器——它使用一个单线程执行器在`PriorityBlockingQueue`中寻找作业并执行它们:

```
public class PriorityJobScheduler {

    private ExecutorService priorityJobPoolExecutor;
    private ExecutorService priorityJobScheduler 
      = Executors.newSingleThreadExecutor();
    private PriorityBlockingQueue<Job> priorityQueue;

    public PriorityJobScheduler(Integer poolSize, Integer queueSize) {
        priorityJobPoolExecutor = Executors.newFixedThreadPool(poolSize);
        priorityQueue = new PriorityBlockingQueue<Job>(
          queueSize, 
          Comparator.comparing(Job::getJobPriority));
        priorityJobScheduler.execute(() -> {
            while (true) {
                try {
                    priorityJobPoolExecutor.execute(priorityQueue.take());
                } catch (InterruptedException e) {
                    // exception needs special handling
                    break;
                }
            }
        });
    }

    public void scheduleJob(Job job) {
        priorityQueue.add(job);
    }
}
```

**这里的关键是用一个定制的比较器创建一个`Job`类型的`PriorityBlockingQueue`实例。**使用`take()`方法从队列中选择下一个要执行的任务，该方法检索并删除队列头。

客户端代码现在只需要调用`scheduleJob()`——将作业添加到队列中。`priorityQueue.add()`使用`JobExecutionComparator`将作业排列在队列中与现有作业相比合适的位置。

注意，实际的作业是使用一个单独的`ExecutorService`和一个专用的线程池来执行的。

## 5。演示

最后，这里有一个调度程序的快速演示:

```
private static int POOL_SIZE = 1;
private static int QUEUE_SIZE = 10;

@Test
public void whenMultiplePriorityJobsQueued_thenHighestPriorityJobIsPicked() {
    Job job1 = new Job("Job1", JobPriority.LOW);
    Job job2 = new Job("Job2", JobPriority.MEDIUM);
    Job job3 = new Job("Job3", JobPriority.HIGH);
    Job job4 = new Job("Job4", JobPriority.MEDIUM);
    Job job5 = new Job("Job5", JobPriority.LOW);
    Job job6 = new Job("Job6", JobPriority.HIGH);

    PriorityJobScheduler pjs = new PriorityJobScheduler(
      POOL_SIZE, QUEUE_SIZE);

    pjs.scheduleJob(job1);
    pjs.scheduleJob(job2);
    pjs.scheduleJob(job3);
    pjs.scheduleJob(job4);
    pjs.scheduleJob(job5);
    pjs.scheduleJob(job6);

    // clean up
}
```

为了演示作业是按照优先级顺序执行的，我们将`POOL_SIZE`保持为 1，尽管`QUEUE_SIZE`是 10。我们为调度程序提供不同优先级的作业。

这是我们在其中一次运行中得到的示例输出:

```
Job:Job3 Priority:HIGH
Job:Job6 Priority:HIGH
Job:Job4 Priority:MEDIUM
Job:Job2 Priority:MEDIUM
Job:Job1 Priority:LOW
Job:Job5 Priority:LOW
```

输出可能因运行而异。然而，我们不应该出现这样的情况:即使队列中包含一个优先级较高的作业，也执行一个优先级较低的作业。

## 6。结论

在这个快速教程中，我们看到了如何使用`PriorityBlockingQueue`按照自定义的优先级顺序执行任务。

像往常一样，源文件可以在 GitHub 上找到[。](https://web.archive.org/web/20221126224409/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-advanced-2)