# Spring Cloud Sleuth 在 Monolith 应用程序中的应用

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-cloud-sleuth-single-application>

## 1。概述

在本文中，我们将介绍[`Spring Cloud Sleuth`](https://web.archive.org/web/20220812054235/https://cloud.spring.io/spring-cloud-sleuth/)——一个在任何应用程序中增强日志的强大工具，尤其是在由多个服务组成的系统中。

对于这篇文章，我们将重点关注在 monolith 应用程序中使用 Sleuth **，而不是跨微服务**。

我们都有过这样的不幸经历:试图诊断一个调度任务、一个多线程操作或一个复杂的 web 请求的问题。通常，即使有日志记录，也很难判断哪些操作需要关联在一起才能创建一个请求。

这使得**诊断一个复杂的动作变得非常困难**甚至不可能。通常会产生这样的解决方案，比如向请求中的每个方法传递一个惟一的 id 来标识日志。

进来了 `Sleuth`。这个库使得识别属于特定作业、线程或请求的日志成为可能。Sleuth 毫不费力地与日志框架如`Logback`和`SLF4J` 集成，添加独特的标识符，帮助使用日志跟踪和诊断问题。

让我们来看看它是如何工作的。

## 2。设置

我们首先在我们最喜欢的 IDE 中创建一个`Spring Boot` web 项目，并将这个依赖项添加到我们的`pom.xml`文件中:

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
```

我们的应用程序使用`Spring Boot`运行，父 pom 为每个条目提供版本。这个依赖的最新版本可以在这里找到:[spring-cloud-starter-sleuth](https://web.archive.org/web/20220812054235/https://repo.spring.io/libs-milestone-local/org/springframework/cloud/spring-cloud-sleuth/)。要查看整个 POM，请在 [Github](https://web.archive.org/web/20220812054235/https://github.com/eugenp/tutorials/tree/master/spring-sleuth) 上查看该项目。

此外，让我们添加一个应用程序名来指示`Sleuth`识别这个应用程序的日志。

在我们的`application.properties`文件中添加这一行:

```java
spring.application.name=Baeldung Sleuth Tutorial
```

## 3。侦察配置

`Sleuth`能够在许多情况下增强日志。从版本 2.0.0 开始，Spring Cloud Sleuth 使用 [Brave](https://web.archive.org/web/20220812054235/https://github.com/openzipkin/brave) 作为跟踪库，为进入我们应用程序的每个 web 请求添加唯一的 id。此外，Spring 团队增加了对跨线程边界共享这些 id 的支持。

可以将跟踪视为应用程序中触发的单个请求或作业。该请求中的所有不同步骤，甚至跨应用程序和线程边界，都将具有相同的 traceId。

另一方面，跨度可以被认为是作业或请求的一部分。单个跟踪可以由多个跨度组成，每个跨度与请求的特定步骤或部分相关。使用 trace 和 span ids，我们可以在应用程序处理请求时准确定位应用程序所处的时间和位置。使得阅读我们的日志更加容易。

在我们的示例中，我们将在单个应用程序中探索这些功能。

### 3.1。简单的 Web 请求

首先，让我们创建一个控制器类作为使用的入口点:

```java
@RestController
public class SleuthController {

    @GetMapping("/")
    public String helloSleuth() {
        logger.info("Hello Sleuth");
        return "success";
    }
}
```

让我们运行我们的应用程序并导航到“http://localhost:8080”。查看日志，查看如下所示的输出:

```java
2017-01-10 22:36:38.254  INFO 
  [Baeldung Sleuth Tutorial,4e30f7340b3fb631,4e30f7340b3fb631,false] 12516 
  --- [nio-8080-exec-1] c.b.spring.session.SleuthController : Hello Sleuth
```

这看起来像一个普通的日志，除了括号之间的部分。这是`Spring Sleuth`补充的核心信息。该数据遵循以下格式:

**【应用名称，traceId，spanId，导出】**

*   **应用程序名称**–这是我们在属性文件中设置的名称，可用于聚合来自同一应用程序的多个实例的日志。
*   **trace Id**–这是分配给单个请求、作业或操作的 id。类似于每个独特的用户发起的 web 请求都有自己的`traceId`。
*   **SpanId**–跟踪一个工作单元。想象一个由多个步骤组成的请求。每一步都可以有自己的`spanId`并被单独跟踪。默认情况下，任何应用程序流都将以相同的 TraceId 和 SpanId 开始。
*   **Export**–这个属性是一个布尔值，表示这个日志是否被导出到一个聚合器，比如`Zipkin`。`Zipkin`超出了本文的范围，但是它在分析由`Sleuth`创建的日志中起着重要的作用。

到目前为止，您应该对这个库的功能有所了解。让我们来看另一个例子，进一步展示这个库对于日志记录是多么不可或缺。

### 3.2。带有服务访问的简单 Web 请求

让我们首先用一个方法创建一个服务:

```java
@Service
public class SleuthService {

    public void doSomeWorkSameSpan() {
        Thread.sleep(1000L);
        logger.info("Doing some work");
    }
}
```

现在，让我们将服务注入到控制器中，并添加一个访问它的请求映射方法:

```java
@Autowired
private SleuthService sleuthService;

    @GetMapping("/same-span")
    public String helloSleuthSameSpan() throws InterruptedException {
        logger.info("Same Span");
        sleuthService.doSomeWorkSameSpan();
        return "success";
}
```

最后，重新启动应用程序并导航到“http://localhost:8080/same-span”。观察日志输出，如下所示:

```java
2017-01-10 22:51:47.664  INFO 
  [Baeldung Sleuth Tutorial,b77a5ea79036d5b9,b77a5ea79036d5b9,false] 12516 
  --- [nio-8080-exec-3] c.b.spring.session.SleuthController      : Same Span
2017-01-10 22:51:48.664  INFO 
  [Baeldung Sleuth Tutorial,b77a5ea79036d5b9,b77a5ea79036d5b9,false] 12516 
  --- [nio-8080-exec-3] c.baeldung.spring.session.SleuthService  : Doing some work
```

请注意，即使消息来自两个不同的类，两个日志之间的跟踪和 span ids 也是相同的。这使得在请求期间通过搜索请求的`traceId`来识别每个日志变得很简单。

这是默认行为，一个请求得到一个`traceId`和`spanId`。但是我们可以根据需要手动添加跨度。让我们看一个使用这个特性的例子。

### 3.3。手动添加跨度

首先，让我们添加一个新的控制器:

```java
@GetMapping("/new-span")
public String helloSleuthNewSpan() {
    logger.info("New Span");
    sleuthService.doSomeWorkNewSpan();
    return "success";
}
```

现在让我们将新方法添加到我们的服务中:

```java
@Autowired
private Tracer tracer;
// ...
public void doSomeWorkNewSpan() throws InterruptedException {
    logger.info("I'm in the original span");

    Span newSpan = tracer.nextSpan().name("newSpan").start();
    try (SpanInScope ws = tracer.withSpanInScope(newSpan.start())) {
        Thread.sleep(1000L);
        logger.info("I'm in the new span doing some cool work that needs its own span");
    } finally {
        newSpan.finish();
    }

    logger.info("I'm in the original span");
}
```

注意，我们还添加了一个新对象，`Tracer`。`tracer`实例由`Spring Sleuth`在启动时创建，并通过依赖注入提供给我们的类。

必须手动启动和停止跟踪。为了实现这一点，在手动创建的`span`中运行的代码被放置在`try-finally`块中，以确保无论操作成功与否`span`都被关闭。另外，请注意，新的 span 必须放在范围内。

重新启动应用程序并导航到“http://localhost:8080/new-span”。观察日志输出，如下所示:

```java
2017-01-11 21:07:54.924  
  INFO [Baeldung Sleuth Tutorial,9cdebbffe8bbbade,9cdebbffe8bbbade,false] 12516 
  --- [nio-8080-exec-6] c.b.spring.session.SleuthController      : New Span
2017-01-11 21:07:54.924  
  INFO [Baeldung Sleuth Tutorial,9cdebbffe8bbbade,9cdebbffe8bbbade,false] 12516 
  --- [nio-8080-exec-6] c.baeldung.spring.session.SleuthService  : 
  I'm in the original span
2017-01-11 21:07:55.924  
  INFO [Baeldung Sleuth Tutorial,9cdebbffe8bbbade,1e706f252a0ee9c2,false] 12516 
  --- [nio-8080-exec-6] c.baeldung.spring.session.SleuthService  : 
  I'm in the new span doing some cool work that needs its own span
2017-01-11 21:07:55.924  
  INFO [Baeldung Sleuth Tutorial,9cdebbffe8bbbade,9cdebbffe8bbbade,false] 12516 
  --- [nio-8080-exec-6] c.baeldung.spring.session.SleuthService  : 
  I'm in the original span
```

我们可以看到，第三个日志与其他日志共享`traceId`，但是它有一个唯一的`spanId`。这可用于定位单个请求中的不同部分，以便进行更细粒度的跟踪。

现在我们来看看`Sleuth's`对线程的支持。

### 3.4。跨越 Runnables

为了演示`Sleuth`的线程能力，让我们首先添加一个配置类来设置一个线程池:

```java
@Configuration
public class ThreadConfig {

    @Autowired
    private BeanFactory beanFactory;

    @Bean
    public Executor executor() {
        ThreadPoolTaskExecutor threadPoolTaskExecutor
         = new ThreadPoolTaskExecutor();
        threadPoolTaskExecutor.setCorePoolSize(1);
        threadPoolTaskExecutor.setMaxPoolSize(1);
        threadPoolTaskExecutor.initialize();

        return new LazyTraceExecutor(beanFactory, threadPoolTaskExecutor);
    }
}
```

这里需要注意的是`LazyTraceExecutor`的使用。这个类来自于`Sleuth`库，是一种特殊的执行器，它将我们的`traceId`传播到新的线程，并在进程中创建新的`spanId`。

现在让我们将这个执行器连接到我们的控制器中，并在一个新的请求映射方法中使用它:

```java
@Autowired
private Executor executor;

    @GetMapping("/new-thread")
    public String helloSleuthNewThread() {
        logger.info("New Thread");
        Runnable runnable = () -> {
            try {
                Thread.sleep(1000L);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            logger.info("I'm inside the new thread - with a new span");
        };
        executor.execute(runnable);

        logger.info("I'm done - with the original span");
        return "success";
}
```

有了 runnable 之后，让我们重新启动应用程序并导航到“http://localhost:8080/new-thread”。观察日志输出，如下所示:

```java
2017-01-11 21:18:15.949  
  INFO [Baeldung Sleuth Tutorial,96076a78343c364d,96076a78343c364d,false] 12516 
  --- [nio-8080-exec-9] c.b.spring.session.SleuthController      : New Thread
2017-01-11 21:18:15.950  
  INFO [Baeldung Sleuth Tutorial,96076a78343c364d,96076a78343c364d,false] 12516 
  --- [nio-8080-exec-9] c.b.spring.session.SleuthController      : 
  I'm done - with the original span
2017-01-11 21:18:16.953  
  INFO [Baeldung Sleuth Tutorial,96076a78343c364d,e3b6a68013ddfeea,false] 12516 
  --- [lTaskExecutor-1] c.b.spring.session.SleuthController      : 
  I'm inside the new thread - with a new span
```

与前面的例子非常相似，我们可以看到所有的日志都共享同一个`traceId`。但是来自 runnable 的日志有一个独特的跨度，可以跟踪该线程中完成的工作。请记住，这是因为`LazyTraceExecutor`而发生的，如果我们使用普通的执行器，我们将继续看到在新线程中使用相同的`spanId`。

现在让我们看看`Sleuth's`对`@Async`方法的支持。

### 3.5。`@Async`支持

要添加异步支持，让我们首先修改我们的`ThreadConfig`类来启用这个特性:

```java
@Configuration
@EnableAsync
public class ThreadConfig extends AsyncConfigurerSupport {

    //...
    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor threadPoolTaskExecutor = new ThreadPoolTaskExecutor();
        threadPoolTaskExecutor.setCorePoolSize(1);
        threadPoolTaskExecutor.setMaxPoolSize(1);
        threadPoolTaskExecutor.initialize();

        return new LazyTraceExecutor(beanFactory, threadPoolTaskExecutor);
    }
}
```

注意，我们扩展了`AsyncConfigurerSupport`来指定我们的异步执行器，并使用`LazyTraceExecutor`来确保 traceIds 和 spanIds 被正确传播。我们还把`@EnableAsync`加到了我们班的最前面。

现在让我们为我们的服务添加一个异步方法:

```java
@Async
public void asyncMethod() {
    logger.info("Start Async Method");
    Thread.sleep(1000L);
    logger.info("End Async Method");
}
```

现在让我们从控制器调用这个方法:

```java
@GetMapping("/async")
public String helloSleuthAsync() {
    logger.info("Before Async Method Call");
    sleuthService.asyncMethod();
    logger.info("After Async Method Call");

    return "success";
}
```

最后，让我们重新启动服务并导航到“http://localhost:8080/async”。观察日志输出，如下所示:

```java
2017-01-11 21:30:40.621  
  INFO [Baeldung Sleuth Tutorial,c187f81915377fff,c187f81915377fff,false] 10072 
  --- [nio-8080-exec-2] c.b.spring.session.SleuthController      : 
  Before Async Method Call
2017-01-11 21:30:40.622  
  INFO [Baeldung Sleuth Tutorial,c187f81915377fff,c187f81915377fff,false] 10072 
  --- [nio-8080-exec-2] c.b.spring.session.SleuthController      : 
  After Async Method Call
2017-01-11 21:30:40.622  
  INFO [Baeldung Sleuth Tutorial,c187f81915377fff,8a9f3f097dca6a9e,false] 10072 
  --- [lTaskExecutor-1] c.baeldung.spring.session.SleuthService  : 
  Start Async Method
2017-01-11 21:30:41.622  
  INFO [Baeldung Sleuth Tutorial,c187f81915377fff,8a9f3f097dca6a9e,false] 10072 
  --- [lTaskExecutor-1] c.baeldung.spring.session.SleuthService  : 
  End Async Method
```

我们可以看到，与我们的 runnable 示例非常相似，`Sleuth`将`traceId`传播到异步方法中，并添加了一个惟一的 spanId。

现在让我们来看一个使用 spring 支持调度任务的例子。

### 3.6。`@Scheduled`支持

最后，让我们看看`Sleuth` 如何与`@Scheduled` 方法一起工作。为此，让我们更新我们的`ThreadConfig`类来启用调度:

```java
@Configuration
@EnableAsync
@EnableScheduling
public class ThreadConfig extends AsyncConfigurerSupport
  implements SchedulingConfigurer {

    //...

    @Override
    public void configureTasks(ScheduledTaskRegistrar scheduledTaskRegistrar) {
        scheduledTaskRegistrar.setScheduler(schedulingExecutor());
    }

    @Bean(destroyMethod = "shutdown")
    public Executor schedulingExecutor() {
        return Executors.newScheduledThreadPool(1);
    }
}
```

注意，我们已经实现了`SchedulingConfigurer`接口并覆盖了它的 configureTasks 方法。我们还把`@EnableScheduling` 加到了我们班的最前面。

接下来，让我们为计划任务添加一个服务:

```java
@Service
public class SchedulingService {

    private Logger logger = LoggerFactory.getLogger(this.getClass());

    @Autowired
    private SleuthService sleuthService;

    @Scheduled(fixedDelay = 30000)
    public void scheduledWork() throws InterruptedException {
        logger.info("Start some work from the scheduled task");
        sleuthService.asyncMethod();
        logger.info("End work from scheduled task");
    }
}
```

在本课中，我们创建了一个固定延迟为 30 秒的单个计划任务。

现在让我们重新启动我们的应用程序，等待我们的任务被执行。观察控制台的如下输出:

```java
2017-01-11 21:30:58.866  
  INFO [Baeldung Sleuth Tutorial,3605f5deaea28df2,3605f5deaea28df2,false] 10072 
  --- [pool-1-thread-1] c.b.spring.session.SchedulingService     : 
  Start some work from the scheduled task
2017-01-11 21:30:58.866  
  INFO [Baeldung Sleuth Tutorial,3605f5deaea28df2,3605f5deaea28df2,false] 10072 
  --- [pool-1-thread-1] c.b.spring.session.SchedulingService     : 
  End work from scheduled task
```

我们可以看到,`Sleuth`已经为我们的任务创建了新的跟踪和跨度 id。默认情况下，任务的每个实例都有自己的跟踪和跨度。

## 4。结论

总之，我们已经看到了如何在单个 web 应用程序中的各种情况下使用`Spring Sleuth`。我们可以使用这种技术轻松地关联来自单个请求的日志，即使该请求跨越多个线程。

现在我们可以看到`Spring Cloud Sleuth`如何帮助我们在调试多线程环境时保持理智。通过识别`traceId`中的每个操作和`spanId`中的每个步骤，我们可以真正开始在我们的日志中分解我们对复杂工作的分析。

即使我们不去云，`Spring Sleuth`在几乎任何项目中都可能是一个关键的依赖项；它可以无缝集成，并且**是一个巨大的增值**。

从这里开始，您可能想研究一下`Sleuth`的其他特性。它可以支持使用`RestTemplate`的分布式系统中的跟踪，跨越`RabbitMQ`和`Redis`使用的消息传递协议，并通过像 Zuul 这样的网关。

和往常一样，你可以在 Github 上找到源代码[。](https://web.archive.org/web/20220812054235/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-sleuth)