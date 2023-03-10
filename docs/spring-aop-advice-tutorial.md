# 春季建议类型介绍

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-aop-advice-tutorial>

## 1。概述

在本文中，我们将讨论可以在 Spring 中创建的不同类型的 AOP 建议。

**Advice** 是一个方面在特定连接点采取的动作。**不同类型的建议包括“围绕”、“之前”和“之后”的建议。**方面的主要目的是支持横切关注点，比如日志记录、概要分析、缓存和事务管理。

如果你想更深入地了解切入点表达式，可以查看前面对这些的介绍。

## 2。启用建议

有了 Spring，你可以使用 AspectJ 注释声明通知，但是你必须首先**将`@EnableAspectJAutoProxy`注释应用到你的配置类**，这将支持处理用 AspectJ 的`@Aspect`注释标记的组件。

```java
@Configuration
@EnableAspectJAutoProxy
public class AopConfiguration {
    ...
}
```

### 2.1.Spring Boot

在 Spring Boot 项目中，我们不必显式地使用`@EnableAspectJAutoProxy`。如果`Aspect`或`Advice`在类路径中，有一个专用的 [`AopAutoConfiguration`](https://web.archive.org/web/20221026180855/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/autoconfigure/aop/AopAutoConfiguration.html) 来启用 Spring 的 AOP 支持。

## 3。建议前

顾名思义，这个通知是在连接点之前执行的。除非抛出异常，否则它不会阻止它建议的方法继续执行。

考虑下面这个方面，它只是在调用方法之前记录方法名:

```java
@Component
@Aspect
public class LoggingAspect {

    private Logger logger = Logger.getLogger(LoggingAspect.class.getName());

    @Pointcut("@target(org.springframework.stereotype.Repository)")
    public void repositoryMethods() {};

    @Before("repositoryMethods()")
    public void logMethodCall(JoinPoint jp) {
        String methodName = jp.getSignature().getName();
        logger.info("Before " + methodName);
    }
}
```

`logMethodCall`通知将在由`repositoryMethods`切入点定义的任何存储库方法之前执行。

## 4。建议后

**无论是否抛出异常，使用`@After` 注释声明的 After advice 都会在匹配方法执行后执行。**

在某些方面，它类似于一个`finally`块。如果需要在正常执行后才触发通知，应该使用由`@AfterReturning` 注释声明的`returning advice`。如果您希望您的通知只在目标方法抛出异常时被触发，那么您应该使用通过使用`@AfterThrowing` 注释声明的`throwing advice,` 。

假设我们希望在创建一个新的`Foo`实例时通知一些应用程序组件。我们可以发布来自`FooDao`的事件，但是这将违反单一责任原则。

相反，我们可以通过定义以下方面来实现这一点:

```java
@Component
@Aspect
public class PublishingAspect {

    private ApplicationEventPublisher eventPublisher;

    @Autowired
    public void setEventPublisher(ApplicationEventPublisher eventPublisher) {
        this.eventPublisher = eventPublisher;
    }

    @Pointcut("@target(org.springframework.stereotype.Repository)")
    public void repositoryMethods() {}

    @Pointcut("execution(* *..create*(Long,..))")
    public void firstLongParamMethods() {}

    @Pointcut("repositoryMethods() && firstLongParamMethods()")
    public void entityCreationMethods() {}

    @AfterReturning(value = "entityCreationMethods()", returning = "entity")
    public void logMethodCall(JoinPoint jp, Object entity) throws Throwable {
        eventPublisher.publishEvent(new FooCreationEvent(entity));
    }
}
```

首先注意，通过使用*@ after*`eturning`注释，我们可以访问目标方法的返回值。其次，通过声明类型为`JoinPoint,`的参数，我们可以访问目标方法调用的参数。

接下来，我们创建一个监听器，它将简单地记录[事件](/web/20221026180855/https://www.baeldung.com/spring-events):

```java
@Component
public class FooCreationEventListener implements ApplicationListener<FooCreationEvent> {

    private Logger logger = Logger.getLogger(getClass().getName());

    @Override
    public void onApplicationEvent(FooCreationEvent event) {
        logger.info("Created foo instance: " + event.getSource().toString());
    }
}
```

## 5。周边建议

**围绕建议**围绕一个连接点，比如一个方法调用。

这是最有力的建议。 **Around advice 可以在方法调用前后执行自定义行为。**它还负责选择是继续执行连接点，还是通过提供自己的返回值或抛出异常来简化建议方法的执行。

为了演示它的使用，假设我们想要测量方法执行时间。让我们为此创建一个方面:

```java
@Aspect
@Component
public class PerformanceAspect {

    private Logger logger = Logger.getLogger(getClass().getName());

    @Pointcut("within(@org.springframework.stereotype.Repository *)")
    public void repositoryClassMethods() {};

    @Around("repositoryClassMethods()")
    public Object measureMethodExecutionTime(ProceedingJoinPoint pjp) throws Throwable {
        long start = System.nanoTime();
        Object retval = pjp.proceed();
        long end = System.nanoTime();
        String methodName = pjp.getSignature().getName();
        logger.info("Execution of " + methodName + " took " + 
          TimeUnit.NANOSECONDS.toMillis(end - start) + " ms");
        return retval;
    }
}
```

当由`repositoryClassMethods`切入点匹配的任何连接点被执行时，这个通知被触发。

这个通知接受一个类型为 **`ProceedingJointPoint`的参数。参数让我们有机会在目标方法调用之前采取行动。我**在这种情况下，我们只需节省方法启动时间。

其次，通知返回类型是`Object`，因为目标方法可以返回任何类型的结果。如果目标方法是`void,`，`null`将被返回。在目标方法调用之后，我们可以测量计时，记录计时，并将方法的结果值返回给调用者。

## 6。概述

在本文中，我们学习了 Spring 中不同类型的通知以及它们的声明和实现。我们使用基于模式的方法和 AspectJ 注释来定义方面。我们还提供了几种可能的建议应用程序。

所有这些例子和代码片段的实现都可以在[我的 GitHub 项目](https://web.archive.org/web/20221026180855/https://github.com/eugenp/tutorials/tree/master/spring-aop)中找到。