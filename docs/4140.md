# Spring Boot 应用程序的正常关闭

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-graceful-shutdown>

## 1.概观

在关机时，默认情况下，Spring 的`TaskExecutor`只是中断所有正在运行的任务，但是让它等待所有正在运行的任务完成可能也不错。这个**为每个任务提供了一个采取措施确保安全关机的机会。**

在这个快速教程中，我们将学习当涉及到使用线程池执行任务时，如何更优雅地关闭 Spring Boot 应用程序。

## 2.简单的例子

让我们考虑一个简单的 Spring Boot 应用程序。我们将自动连接默认的`TaskExecutor` bean:

```
@Autowired
private TaskExecutor taskExecutor;
```

在应用程序启动时，让我们使用线程池中的一个线程执行一个 1 分钟长的进程:

```
taskExecutor.execute(() -> {
    Thread.sleep(60_000);
});
```

当[关闭](/web/20221208143847/https://www.baeldung.com/spring-boot-shutdown)被启动时，例如，启动后 20 秒，示例中的线程被中断，应用程序立即关闭。

## 3.等待任务完成

让我们通过创建一个定制的`ThreadPoolTaskExecutor` bean 来改变任务执行器的默认行为。

这个类提供了一个标志`setWaitForTasksToCompleteOnShutdown` 来防止中断正在运行的任务。我们把它设置为`true`:

```
@Bean
public TaskExecutor taskExecutor() {
    ThreadPoolTaskExecutor taskExecutor = new ThreadPoolTaskExecutor();
    taskExecutor.setCorePoolSize(2);
    taskExecutor.setMaxPoolSize(2);
    taskExecutor.setWaitForTasksToCompleteOnShutdown(true);
    taskExecutor.initialize();
    return taskExecutor;
}
```

我们将重写前面的逻辑，创建 3 个线程，每个线程执行一个 1 分钟长的任务。

```
@PostConstruct
public void runTaskOnStartup() {
    for (int i = 0; i < 3; i++) {
        taskExecutor.execute(() -> {
            Thread.sleep(60_000);
        });
    }
}
```

现在，让我们在启动后的前 60 秒内启动关机。

我们看到应用程序在启动 120 秒后就关闭了。池大小为 2 只允许同时执行两个任务，因此第三个任务排队等候。

设置标志确保当前执行的任务和排队的任务都完成。

注意，当收到关闭请求时，**任务执行器关闭队列** **，这样就不能添加新任务。**

## 4.终止前的最长等待时间

尽管我们已经配置为等待正在进行的和排队的任务完成，Spring **继续关闭容器**的其余部分。这可能会释放任务执行器所需的资源，导致任务失败。

为了阻止容器其余部分的关闭，我们可以在`ThreadPoolTaskExecutor:` 上指定最大等待时间

```
taskExecutor.setAwaitTerminationSeconds(30);
```

这确保了在指定的时间段内，容器级别的**关闭过程将被阻塞**。

当我们将`setWaitForTasksToCompleteOnShutdown`标志设置为`true`时，我们需要指定一个明显更高的超时，以便队列中所有剩余的任务也被执行。

## 5.结论

在这个快速教程中，我们看到了如何通过配置任务执行器 bean 来安全地关闭 Spring Boot 应用程序，以完成正在运行和提交的任务，直到结束。这保证了所有任务都有指定的时间来完成工作。

一个明显的副作用是，它也可能导致更长的关闭阶段。因此，我们需要根据应用程序的性质来决定是否使用它。

和往常一样，GitHub 上的[提供了这篇文章中的例子。](https://web.archive.org/web/20221208143847/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-deployment)