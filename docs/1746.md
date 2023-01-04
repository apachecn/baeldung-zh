# Hystrix 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/introduction-to-hystrix>

## 1。概述

典型的分布式系统由许多协同工作的服务组成。

这些服务容易失败或延迟响应。如果一个服务失败，它可能会影响其他服务，从而影响性能，并可能使应用程序的其他部分不可访问，或者在最糟糕的情况下使整个应用程序停止运行。

**当然，有一些解决方案可以帮助应用程序具有弹性和容错能力 Hystrix 就是这样一个框架。**

Hystrix 框架库通过提供容错和延迟容限来帮助控制服务之间的交互。它通过隔离失败的服务和阻止失败的级联效应来提高系统的整体弹性。

在这一系列的文章中，我们将首先看看当一个服务或系统出现故障时，Hystrix 是如何进行救援的，以及在这些情况下 Hystrix 能做些什么。

## 2。简单的例子

Hystrix 提供容错和延迟的方式是隔离和包装对远程服务的调用。

在这个简单的例子中，我们将调用包装在`HystrixCommand:`的`run()`方法中

```
class CommandHelloWorld extends HystrixCommand<String> {

    private String name;

    CommandHelloWorld(String name) {
        super(HystrixCommandGroupKey.Factory.asKey("ExampleGroup"));
        this.name = name;
    }

    @Override
    protected String run() {
        return "Hello " + name + "!";
    }
}
```

我们如下执行调用:

```
@Test
public void givenInputBobAndDefaultSettings_whenCommandExecuted_thenReturnHelloBob(){
    assertThat(new CommandHelloWorld("Bob").execute(), equalTo("Hello Bob!"));
}
```

## 3。Maven 设置

要在 Maven 项目中使用 Hystrix，我们需要在项目`pom.xml`中拥有来自网飞的`hystrix-core`和`rxjava-core`依赖项:

```
<dependency>
    <groupId>com.netflix.hystrix</groupId>
    <artifactId>hystrix-core</artifactId>
    <version>1.5.4</version>
</dependency> 
```

最新版本总是可以在[这里](https://web.archive.org/web/20220708040045/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22hystrix-core%22)找到。

```
<dependency>
    <groupId>com.netflix.rxjava</groupId>
    <artifactId>rxjava-core</artifactId>
    <version>0.20.7</version>
</dependency>
```

这个库的最新版本总是可以在这里找到。

## 4。设置远程服务

让我们从模拟一个真实世界的例子开始。

**在下面**的例子中，类`RemoteServiceTestSimulator`代表远程服务器上的服务。它有一个方法，在给定的时间后用一条消息来响应。我们可以想象这种等待是对远程系统上耗时过程的模拟，导致对调用服务的响应延迟:

```
class RemoteServiceTestSimulator {

    private long wait;

    RemoteServiceTestSimulator(long wait) throws InterruptedException {
        this.wait = wait;
    }

    String execute() throws InterruptedException {
        Thread.sleep(wait);
        return "Success";
    }
}
```

**这是我们调用`RemoteServiceTestSimulator`的样本客户端**。

对服务的调用被隔离并包装在`HystrixCommand.`的`run()`方法中，正是这种包装提供了我们上面提到的弹性:

```
class RemoteServiceTestCommand extends HystrixCommand<String> {

    private RemoteServiceTestSimulator remoteService;

    RemoteServiceTestCommand(Setter config, RemoteServiceTestSimulator remoteService) {
        super(config);
        this.remoteService = remoteService;
    }

    @Override
    protected String run() throws Exception {
        return remoteService.execute();
    }
}
```

这个调用是通过在`RemoteServiceTestCommand`对象的实例上调用`execute()`方法来执行的。

以下测试演示了这是如何实现的:

```
@Test
public void givenSvcTimeoutOf100AndDefaultSettings_whenRemoteSvcExecuted_thenReturnSuccess()
  throws InterruptedException {

    HystrixCommand.Setter config = HystrixCommand
      .Setter
      .withGroupKey(HystrixCommandGroupKey.Factory.asKey("RemoteServiceGroup2"));

    assertThat(new RemoteServiceTestCommand(config, new RemoteServiceTestSimulator(100)).execute(),
      equalTo("Success"));
}
```

到目前为止，我们已经看到了如何在`HystrixCommand`对象中包装远程服务调用。在下一节中，让我们看看如何处理远程服务开始恶化的情况。

## 5。使用远程服务和防御性编程

### 5.1。超时防御编程

为调用远程服务设置超时是一般的编程惯例。

让我们先来看看如何在`HystrixCommand`上设置超时，以及它如何通过短路来提供帮助:

```
@Test
public void givenSvcTimeoutOf5000AndExecTimeoutOf10000_whenRemoteSvcExecuted_thenReturnSuccess()
  throws InterruptedException {

    HystrixCommand.Setter config = HystrixCommand
      .Setter
      .withGroupKey(HystrixCommandGroupKey.Factory.asKey("RemoteServiceGroupTest4"));

    HystrixCommandProperties.Setter commandProperties = HystrixCommandProperties.Setter();
    commandProperties.withExecutionTimeoutInMilliseconds(10_000);
    config.andCommandPropertiesDefaults(commandProperties);

    assertThat(new RemoteServiceTestCommand(config, new RemoteServiceTestSimulator(500)).execute(),
      equalTo("Success"));
}
```

在上面的测试中，我们通过将超时设置为 500 毫秒来延迟服务的响应。我们还将`HystrixCommand`上的执行超时设置为 10，000 毫秒，从而为远程服务的响应留出足够的时间。

现在让我们看看当执行超时小于服务超时调用时会发生什么:

```
@Test(expected = HystrixRuntimeException.class)
public void givenSvcTimeoutOf15000AndExecTimeoutOf5000_whenRemoteSvcExecuted_thenExpectHre()
  throws InterruptedException {

    HystrixCommand.Setter config = HystrixCommand
      .Setter
      .withGroupKey(HystrixCommandGroupKey.Factory.asKey("RemoteServiceGroupTest5"));

    HystrixCommandProperties.Setter commandProperties = HystrixCommandProperties.Setter();
    commandProperties.withExecutionTimeoutInMilliseconds(5_000);
    config.andCommandPropertiesDefaults(commandProperties);

    new RemoteServiceTestCommand(config, new RemoteServiceTestSimulator(15_000)).execute();
}
```

请注意我们是如何降低门槛并将执行超时设置为 5000 毫秒的。

我们期望服务在 5，000 毫秒内响应，而我们将服务设置为 15，000 毫秒后响应。如果您在执行测试时注意到，测试将在 5，000 毫秒后退出，而不是等待 15，000 毫秒，并将抛出一个`HystrixRuntimeException.`

**这演示了 Hystrix 等待响应的时间不会超过配置的超时时间。这有助于使受 Hystrix 保护的系统更加灵敏。**

在下面几节中，我们将研究如何设置线程池大小来防止线程耗尽，并讨论它的好处。

### 5.2。线程池有限的防御性编程

为服务调用设置超时并不能解决与远程服务相关的所有问题。

当远程服务开始缓慢响应时，典型的应用程序将继续调用该远程服务。

应用程序不知道远程服务是否健康，并且每次请求进来时都会产生新的线程。这将导致已经很吃力的服务器上的线程被使用。

我们不希望发生这种情况，因为我们需要这些线程用于其他远程调用或运行在我们服务器上的进程，我们还希望避免 CPU 利用率激增。

让我们看看如何在`HystrixCommand`中设置线程池的大小:

```
@Test
public void givenSvcTimeoutOf500AndExecTimeoutOf10000AndThreadPool_whenRemoteSvcExecuted
  _thenReturnSuccess() throws InterruptedException {

    HystrixCommand.Setter config = HystrixCommand
      .Setter
      .withGroupKey(HystrixCommandGroupKey.Factory.asKey("RemoteServiceGroupThreadPool"));

    HystrixCommandProperties.Setter commandProperties = HystrixCommandProperties.Setter();
    commandProperties.withExecutionTimeoutInMilliseconds(10_000);
    config.andCommandPropertiesDefaults(commandProperties);
    config.andThreadPoolPropertiesDefaults(HystrixThreadPoolProperties.Setter()
      .withMaxQueueSize(10)
      .withCoreSize(3)
      .withQueueSizeRejectionThreshold(10));

    assertThat(new RemoteServiceTestCommand(config, new RemoteServiceTestSimulator(500)).execute(),
      equalTo("Success"));
}
```

在上面的测试中，我们设置了最大队列大小、核心队列大小和队列拒绝大小。当最大线程数达到 10 且任务队列的大小达到 10 时,`Hystrix`将开始拒绝请求。

核心大小是线程池中始终保持活动状态的线程数量。

### 5.3。具有短路断路器模式的防御性编程

然而，我们仍然可以对远程服务调用进行改进。

让我们考虑远程服务开始失败的情况。

我们不想不断向 it 部门发出请求，浪费资源。理想情况下，我们希望在一段时间内停止发出请求，以便在恢复请求之前给服务时间进行恢复。这就是所谓的`Short Circuit Breaker`模式。

让我们看看 Hystrix 是如何实现这种模式的:

```
@Test
public void givenCircuitBreakerSetup_whenRemoteSvcCmdExecuted_thenReturnSuccess()
  throws InterruptedException {

    HystrixCommand.Setter config = HystrixCommand
      .Setter
      .withGroupKey(HystrixCommandGroupKey.Factory.asKey("RemoteServiceGroupCircuitBreaker"));

    HystrixCommandProperties.Setter properties = HystrixCommandProperties.Setter();
    properties.withExecutionTimeoutInMilliseconds(1000);
    properties.withCircuitBreakerSleepWindowInMilliseconds(4000);
    properties.withExecutionIsolationStrategy
     (HystrixCommandProperties.ExecutionIsolationStrategy.THREAD);
    properties.withCircuitBreakerEnabled(true);
    properties.withCircuitBreakerRequestVolumeThreshold(1);

    config.andCommandPropertiesDefaults(properties);
    config.andThreadPoolPropertiesDefaults(HystrixThreadPoolProperties.Setter()
      .withMaxQueueSize(1)
      .withCoreSize(1)
      .withQueueSizeRejectionThreshold(1));

    assertThat(this.invokeRemoteService(config, 10_000), equalTo(null));
    assertThat(this.invokeRemoteService(config, 10_000), equalTo(null));
    assertThat(this.invokeRemoteService(config, 10_000), equalTo(null));

    Thread.sleep(5000);

    assertThat(new RemoteServiceTestCommand(config, new RemoteServiceTestSimulator(500)).execute(),
      equalTo("Success"));

    assertThat(new RemoteServiceTestCommand(config, new RemoteServiceTestSimulator(500)).execute(),
      equalTo("Success"));

    assertThat(new RemoteServiceTestCommand(config, new RemoteServiceTestSimulator(500)).execute(),
      equalTo("Success"));
}
```

```
public String invokeRemoteService(HystrixCommand.Setter config, int timeout)
  throws InterruptedException {

    String response = null;

    try {
        response = new RemoteServiceTestCommand(config,
          new RemoteServiceTestSimulator(timeout)).execute();
    } catch (HystrixRuntimeException ex) {
        System.out.println("ex = " + ex);
    }

    return response;
}
```

在上面的测试中，我们设置了不同的断路器属性。最重要的是:

*   `CircuitBreakerSleepWindow` 设置为 4，000 毫秒。这将配置断路器窗口并定义时间间隔，在此之后将恢复对远程服务的请求
*   `CircuitBreakerRequestVolumeThreshold`设置为 1，定义在考虑故障率之前所需的最小请求数

有了上面的设置，我们的`HystrixCommand`将在两次请求失败后跳闸打开。第三个请求甚至不会命中远程服务，即使我们已经将服务延迟设置为 500 毫秒，`Hystrix` 也会短路，我们的方法将返回`null`作为响应。

随后，我们将添加一个`Thread.sleep(5000)`，以跨越我们设置的休眠窗口的限制。这将导致`Hystrix`闭合电路，后续请求将成功通过。

## 6。 **结论**

总之，Hystrix 旨在:

1.  针对通常通过网络访问的服务的故障和延迟提供保护和控制
2.  停止由于某些服务关闭而导致的故障级联
3.  快速失败并快速恢复
4.  尽可能优雅地降级
5.  指挥中心对故障的实时监控和报警

在下一篇文章中，我们将看到如何将 Hystrix 的优点与 Spring 框架结合起来。

完整的项目代码和所有示例可以在 github 项目中找到。