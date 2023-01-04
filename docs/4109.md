# Hystrix 与现有 Spring 应用程序的集成

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hystrix-integration-with-spring-aop>

## 1.概观

在上一篇文章中，我们介绍了 Hystrix 的基础知识，以及它如何帮助构建容错和弹性应用。

有许多现有的 Spring 应用程序调用外部系统，这些系统将受益于 Hystrix。不幸的是，可能无法重写这些应用程序来集成 Hystrix，但是在 [Spring AOP](https://web.archive.org/web/20220120025508/https://docs.spring.io/spring/docs/current/spring-framework-reference/html/aop.html) 的帮助下，集成 Hystrix 的非侵入式方法是可能的。

在本文中，我们将研究如何将 Hystrix 与现有的 Spring 应用程序集成。

## 2.Hystrix 成为弹簧应用

### 2.1。现有应用程序

让我们看看应用程序现有的客户端调用者，它调用我们在上一篇文章中创建的`RemoteServiceTestSimulator`:

```
@Component("springClient")
public class SpringExistingClient {

    @Value("${remoteservice.timeout}")
    private int remoteServiceDelay;

    public String invokeRemoteServiceWithOutHystrix() throws InterruptedException {
        return new RemoteServiceTestSimulator(remoteServiceDelay).execute();
    }
}
```

正如我们在上面的代码片段中看到的，`invokeRemoteServiceWithOutHystrix` 方法负责调用`RemoteServiceTestSimulator`远程服务。当然，真实世界的应用程序不会这么简单。

### 2.2。创建一个迂回建议

为了演示如何集成 Hystrix，我们将使用这个客户端作为示例。

为此，**我们将定义一个`Around`建议，它将在`invokeRemoteService` 被执行**时生效:

```
@Around("@annotation(com.baeldung.hystrix.HystrixCircuitBreaker)")
public Object circuitBreakerAround(ProceedingJoinPoint aJoinPoint) {
    return new RemoteServiceCommand(config, aJoinPoint).execute();
}
```

上面的通知被设计成一个`Around`通知，将在一个用`@HystrixCircuitBreaker`注释的切入点上执行。

现在让我们看看`HystrixCircuitBreaker`注释`:`的定义

```
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface HystrixCircuitBreaker {}
```

### 2.3。Hystrix 逻辑

现在我们来看看`RemoteServiceCommand`。它在示例代码中被实现为一个`static inner class`,以封装 Hystrix 调用逻辑:

```
private static class RemoteServiceCommand extends HystrixCommand<String> {

    private ProceedingJoinPoint joinPoint;

    RemoteServiceCommand(Setter config, ProceedingJoinPoint joinPoint) {
        super(config);
        this.joinPoint = joinPoint;
    }

    @Override
    protected String run() throws Exception {
        try {
            return (String) joinPoint.proceed();
        } catch (Throwable th) {
            throw new Exception(th);
        }
    }
}
```

`Aspect`组件的完整实现可以在[这里看到](https://web.archive.org/web/20220120025508/https://github.com/eugenp/tutorials/blob/master/hystrix/src/main/java/com/baeldung/hystrix/HystrixAspect.java)。

### 2.4。用`@HystrixCircuitBreaker`标注

一旦定义了方面，我们就可以用如下所示的`@HystrixCircuitBreaker` 来注释我们的客户端方法，并且每次调用被注释的方法都会触发 Hystrix:

```
@HystrixCircuitBreaker
public String invokeRemoteServiceWithHystrix() throws InterruptedException{
    return new RemoteServiceTestSimulator(remoteServiceDelay).execute();
}
```

以下集成测试将展示 Hystrix 路线和非 Hystrix 路线之间的差异。

### 2.5。测试集成

出于演示的目的，我们定义了两条方法执行路线，一条有 Hystrix，另一条没有。

```
public class SpringAndHystrixIntegrationTest {

    @Autowired
    private HystrixController hystrixController;

    @Test(expected = HystrixRuntimeException.class)
    public void givenTimeOutOf15000_whenClientCalledWithHystrix_thenExpectHystrixRuntimeException()
      throws InterruptedException {
        hystrixController.withHystrix();
    }

    @Test
    public void givenTimeOutOf15000_whenClientCalledWithOutHystrix_thenExpectSuccess()
      throws InterruptedException {
        assertThat(hystrixController.withOutHystrix(), equalTo("Success"));
    }
} 
```

当测试执行时，您可以看到没有 Hystrix 的方法调用将等待远程服务的整个执行时间，而 Hystrix 路由将短路并在定义的超时(在我们的例子中是 10 秒)后抛出`HystrixRuntimeException`。

## 3.结论

我们可以为我们希望用不同配置进行的每个远程服务调用创建一个方面。在下一篇文章中，我们将从项目的开始着眼于集成 Hystrix。

本文中的所有代码都可以在 [GitHub](https://web.archive.org/web/20220120025508/https://github.com/eugenp/tutorials/tree/master/hystrix) 资源库中找到。