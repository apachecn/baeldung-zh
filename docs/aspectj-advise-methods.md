# 用 AspectJ 通知注释类上的方法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/aspectj-advise-methods>

## 1.概观

在本教程中，当调用已配置类的方法时，我们将使用 [AspectJ](https://web.archive.org/web/20220523140025/https://www.eclipse.org/aspectj/) 来编写跟踪日志输出。通过使用 AOP 建议来编写跟踪日志输出，我们将逻辑封装到一个单独的编译单元中。

我们的例子扩展了在[AspectJ 简介](/web/20220523140025/https://www.baeldung.com/aspectj)中给出的信息。

## 2.跟踪日志注释

我们将使用一个注释来配置类，这样就可以跟踪它们的方法调用。使用注释为我们提供了一种简单的机制，可以将跟踪日志记录输出添加到新代码中，而不必直接添加日志记录语句。

让我们创建注释:

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface Trace {
}
```

## 3.创建我们的方面

**我们将创建一个方面来定义我们的`pointcut`，以匹配我们关心的连接点和包含要执行的逻辑的`around`通知。**

我们的方面看起来类似于这样:

```java
public aspect TracingAspect {
    private static final Log LOG = LogFactory.getLog(TracingAspect.class);

    pointcut traceAnnotatedClasses(): within(@Trace *) && execution(* *(..));

    Object around() : traceAnnotatedClasses() {
        String signature = thisJoinPoint.getSignature().toShortString();
        LOG.trace("Entering " + signature);
        try {
            return proceed();
        } finally {
            LOG.trace("Exiting " + signature);
        }
    }
}
```

在我们的方面，我们定义了一个名为`traceAnnotatedClasses`的`pointcut`来匹配用我们的`Trace`注释注释的方法`within`类的`execution`。通过定义和命名一个`pointcut,`，我们可以像对待类中的方法一样重用它。我们将使用这个名为`pointcut`的来配置我们的`around`建议。

**我们的`around`通知将代替我们的`pointcut`匹配的任何连接点执行，并将返回一个`Object`。通过拥有一个`Object`返回类型，我们可以考虑拥有任何返回类型的被建议方法，甚至是`void`。**

我们检索匹配连接点的签名，以创建签名的简短`String`表示，从而将上下文添加到我们的跟踪消息中。因此，我们的日志记录输出将具有类名和所执行的方法，这为我们提供了一些所需的上下文。

在跟踪输出调用之间，我们调用了一个名为`proceed`的方法。**该方法可用于`around`建议，以便继续执行匹配的连接点。**返回类型将是`Object`，因为我们在编译时无法知道返回类型。在将最终的跟踪输出发送到日志之后，我们将把这个值发送回调用者。

**我们将`proceed()`调用包装在`try` / `finally`块中，以确保退出消息被写入。**如果我们想跟踪抛出的异常，我们可以添加`after()`建议，在抛出异常时写一条日志消息:

```java
after() throwing (Exception e) : traceAnnotatedClasses() {
    LOG.trace("Exception thrown from " + thisJoinPoint.getSignature().toShortString(), e);
}
```

## 4.注释我们的代码

现在我们需要启用我们的跟踪。让我们创建一个简单的类，并用我们的自定义注释激活跟踪日志记录:

```java
@Trace
@Component
public class MyTracedService {

    public void performSomeLogic() {
        ...
    }

    public void performSomeAdditionalLogic() {
        ...
    }
}
```

有了`Trace`注释，我们类中的方法将与我们定义的`pointcut`相匹配。当这些方法执行时，跟踪消息将被写入日志。

**运行调用这些方法的代码后，我们的日志输出应该包括类似于**的内容

```java
22:37:58.867 [main] TRACE c.b.a.c.TracingAspect - Entering MyTracedService.performSomeAdditionalLogic()
22:37:58.868 [main] INFO  c.b.a.c.MyTracedService - Inside performSomeAdditionalLogic...
22:37:58.868 [main] TRACE c.b.a.c.TracingAspect - Exiting MyTracedService.performSomeAdditionalLogic()
22:37:58.869 [main] TRACE c.b.a.c.TracingAspect - Entering MyTracedService.performSomeLogic()
22:37:58.869 [main] INFO  c.b.a.c.MyTracedService - Inside performSomeLogic...
22:37:58.869 [main] TRACE c.b.a.c.TracingAspect - Exiting MyTracedService.performSomeLogic() 
```

## 5.结论

在本文中，我们使用 AspectJ 通过对类的一个注释来拦截类的所有方法。这样做允许我们快速地将跟踪日志记录功能添加到新代码中。

我们还将跟踪日志输出逻辑整合到一个编译单元中，以提高随着应用程序的发展修改跟踪日志输出的能力。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20220523140025/https://github.com/eugenp/tutorials/tree/master/spring-aop)