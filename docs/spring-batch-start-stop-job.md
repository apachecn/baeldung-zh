# 如何触发和停止计划的春季批处理作业

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-batch-start-stop-job>

## 1。概述

在本教程中，我们将研究和比较不同的方式来**触发和停止任何所需业务案例的预定 Spring 批处理作业**。

如果您需要关于 Spring Batch 和 Scheduler 的介绍，请参考 [Spring-Batch](/web/20220710202406/https://www.baeldung.com/introduction-to-spring-batch) 和 [Spring-Scheduler](/web/20220710202406/https://www.baeldung.com/spring-scheduled-tasks) 文章。

## 2.触发计划的春季批处理作业

首先，我们有一个类`SpringBatchScheduler`来配置调度和批处理作业。方法`launchJob()`将被注册为计划任务。

此外，为了以最直观的方式触发预定的 Spring 批处理作业，让我们添加一个条件标志，仅当该标志设置为 true 时才触发作业:

```java
private AtomicBoolean enabled = new AtomicBoolean(true);

private AtomicInteger batchRunCounter = new AtomicInteger(0);

@Scheduled(fixedRate = 2000)
public void launchJob() throws Exception {
    if (enabled.get()) {
        Date date = new Date();
        JobExecution jobExecution = jobLauncher()
          .run(job(), new JobParametersBuilder()
            .addDate("launchDate", date)
            .toJobParameters());
        batchRunCounter.incrementAndGet();
    }
}

// stop, start functions (changing the flag of enabled)
```

变量`batchRunCounter`将用于集成测试，以验证批处理作业是否已停止。

## 3.停止计划的春季批处理作业

有了上面的条件标志，我们就能够在调度任务有效的情况下触发调度的 Spring 批处理作业。

如果我们不需要恢复作业，那么我们实际上可以停止计划的任务以节省资源。

让我们在接下来的两个小节中看看两个选项。

### 3.1.使用调度程序后处理器

因为我们是通过使用 [`@Scheduled`](https://web.archive.org/web/20220710202406/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/scheduling/annotation/Scheduled.html) 注释来调度一个方法，所以 bean post 处理器 [`ScheduledAnnotationBeanPostProcessor`](https://web.archive.org/web/20220710202406/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/scheduling/annotation/ScheduledAnnotationBeanPostProcessor.html) 会被首先注册。

我们可以显式调用`postProcessBeforeDestruction()`来销毁给定的调度 bean:

```java
@Test
public void stopJobSchedulerWhenSchedulerDestroyed() throws Exception {
    ScheduledAnnotationBeanPostProcessor bean = context
      .getBean(ScheduledAnnotationBeanPostProcessor.class);
    SpringBatchScheduler schedulerBean = context
      .getBean(SpringBatchScheduler.class);
    await().untilAsserted(() -> Assert.assertEquals(
      2, 
      schedulerBean.getBatchRunCounter().get()));
    bean.postProcessBeforeDestruction(
      schedulerBean, "SpringBatchScheduler");
    await().atLeast(3, SECONDS);

    Assert.assertEquals(
      2, 
      schedulerBean.getBatchRunCounter().get());
}
```

考虑到多个调度程序，最好在自己的类中保留一个调度程序，这样我们可以根据需要停止特定的调度程序。

### 3.2.取消预定的`Future`

另一种停止调度程序的方法是手动取消它的`Future`。

这里有一个用于捕获`Future`地图的自定义任务调度程序:

```java
@Bean
public TaskScheduler poolScheduler() {
    return new CustomTaskScheduler();
}

private class CustomTaskScheduler 
  extends ThreadPoolTaskScheduler {

    //

    @Override
    public ScheduledFuture<?> scheduleAtFixedRate(
      Runnable task, long period) {
        ScheduledFuture<?> future = super
          .scheduleAtFixedRate(task, period);

        ScheduledMethodRunnable runnable = (ScheduledMethodRunnable) task;
        scheduledTasks.put(runnable.getTarget(), future);

        return future;
    }
}
```

然后，我们迭代`Future`图，并取消批处理作业调度器的`Future`:

```java
public void cancelFutureSchedulerTasks() {
    scheduledTasks.forEach((k, v) -> {
        if (k instanceof SpringBatchScheduler) {
            v.cancel(false);
        }
    });
}
```

在有多个调度器任务的情况下，我们可以在定制调度器池中维护`Future`映射，但是基于调度器类取消相应的调度`Future`。

## 4.结论

在这篇简短的文章中，我们尝试了三种不同的方式来触发或停止预定的 Spring 批处理作业。

当我们需要重新启动批处理作业时，使用条件标志来管理作业运行将是一个灵活的解决方案。否则，我们可以遵循其他两个选项来完全停止调度程序。

和往常一样，本文中使用的所有代码示例都可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220710202406/https://github.com/eugenp/tutorials/tree/master/spring-batch-2)