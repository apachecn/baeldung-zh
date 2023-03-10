# 春天如何做@Async

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-async>

## 1。概述

在本教程中，我们将探索 Spring 中的**异步执行支持和`@Async`注释。**

简单地说，用`@Async`注释 bean 的方法将使它**在一个单独的线程中执行。**换句话说，调用者不会等待被调用方法的完成。

Spring 中一个有趣的方面是框架 [**中的事件支持在必要时也支持异步处理**](/web/20221008082221/https://www.baeldung.com/spring-events) 。

## 延伸阅读:

## [春季事件](/web/20221008082221/https://www.baeldung.com/spring-events)

The Basics of Events in Spring - create a simple, custom Event, publish it and handle it in a listener.[Read more](/web/20221008082221/https://www.baeldung.com/spring-events) →

## [使用@Async 的 Spring 安全上下文传播](/web/20221008082221/https://www.baeldung.com/spring-security-async-principal-propagation)

A short example of propagating Spring Security context when using @Async annotation[Read more](/web/20221008082221/https://www.baeldung.com/spring-security-async-principal-propagation) →

## [Servlet 3 异步支持 Spring MVC 和 Spring Security](/web/20221008082221/https://www.baeldung.com/spring-mvc-async-security)

Quick intro to the Spring Security support for async requests in Spring MVC.[Read more](/web/20221008082221/https://www.baeldung.com/spring-mvc-async-security) →

## 2。启用异步支持

让我们从用 **Java 配置启用异步处理**的**开始。**

我们将通过向配置类添加`@EnableAsync`来实现这一点:

```java
@Configuration
@EnableAsync
public class SpringAsyncConfig { ... }
```

启用注释就足够了。但是也有一些简单的配置选项:

*   `**annotation** –` 默认情况下，`@EnableAsync`检测 Spring 的`@Async`注释和 EJB 3.1 `javax.ejb.Asynchronous`。我们也可以使用这个选项来检测其他用户定义的注释类型。
*   `**mode**` 表示应该使用的`advice`的类型——基于 JDK 代理或 AspectJ 编织。
*   `**proxyTargetClass**` 表示应该使用的`proxy` 的类型— CGLIB 或 JDK。该属性仅在 **`mode`** 设置为`AdviceMode.PROXY`时有效。
*   `**order**`设置应用`AsyncAnnotationBeanPostProcessor`的顺序。默认情况下，它最后运行，以便可以考虑所有现有的代理。

我们还可以通过使用`task`名称空间来启用带有 **XML 配置**的异步处理:

```java
<task:executor id="myexecutor" pool-size="5"  />
<task:annotation-driven executor="myexecutor"/>
```

## 3。`@Async`注解

首先，让我们复习一下规则。`@Async`有两个限制:

*   它必须只应用于`public`方法。
*   自调用——从同一个类中调用异步方法——不起作用。

原因很简单:**方法需要是`public`** ，这样它才能被代理。并且**自调用不工作**，因为它绕过代理直接调用底层方法。

### 3.1。具有 Void 返回类型的方法

这是将 void 返回类型的方法配置为异步运行的简单方法:

```java
@Async
public void asyncMethodWithVoidReturnType() {
    System.out.println("Execute method asynchronously. " 
      + Thread.currentThread().getName());
}
```

### 3.2。返回类型为的方法

我们还可以通过包装将来的实际返回，将`@Async`应用于具有返回类型的方法:

```java
@Async
public Future<String> asyncMethodWithReturnType() {
    System.out.println("Execute method asynchronously - " 
      + Thread.currentThread().getName());
    try {
        Thread.sleep(5000);
        return new AsyncResult<String>("hello world !!!!");
    } catch (InterruptedException e) {
        //
    }

    return null;
}
```

Spring 还提供了一个实现`Future`的`AsyncResult`类。我们可以用它来跟踪异步方法执行的结果。

现在让我们调用上面的方法，并使用`Future`对象检索异步流程的结果。

```java
public void testAsyncAnnotationForMethodsWithReturnType()
  throws InterruptedException, ExecutionException {
    System.out.println("Invoking an asynchronous method. " 
      + Thread.currentThread().getName());
    Future<String> future = asyncAnnotationExample.asyncMethodWithReturnType();

    while (true) {
        if (future.isDone()) {
            System.out.println("Result from asynchronous process - " + future.get());
            break;
        }
        System.out.println("Continue doing something else. ");
        Thread.sleep(1000);
    }
}
```

## 4。遗嘱执行人

默认情况下，Spring 使用一个`SimpleAsyncTaskExecutor`来异步运行这些方法。但是我们可以在两个层次上覆盖缺省值:应用程序层次或者单独的方法层次。

### 4.1。在方法级别覆盖执行器

我们需要在配置类中声明所需的执行器:

```java
@Configuration
@EnableAsync
public class SpringAsyncConfig {

    @Bean(name = "threadPoolTaskExecutor")
    public Executor threadPoolTaskExecutor() {
        return new ThreadPoolTaskExecutor();
    }
}
```

那么我们应该在`@Async`中提供执行者的名字作为属性:

```java
@Async("threadPoolTaskExecutor")
public void asyncMethodWithConfiguredExecutor() {
    System.out.println("Execute method with configured executor - "
      + Thread.currentThread().getName());
}
```

### 4.2。在应用程序级别覆盖执行器

配置类应该实现`AsyncConfigurer`接口。所以，它必须实现`getAsyncExecutor()`方法。这里，我们将返回整个应用程序的执行者。这现在成为运行用`@Async`注释的方法的默认执行器:

```java
@Configuration
@EnableAsync
public class SpringAsyncConfig implements AsyncConfigurer {

    @Override
    public Executor getAsyncExecutor() {
        return new ThreadPoolTaskExecutor();
    }

}
```

## 5。异常处理

当方法返回类型是`Future`时，异常处理很容易。`Future.get()`方法会抛出异常。

但是如果返回类型是`void`，**异常不会传播到调用线程。**所以，我们需要添加额外的配置来处理异常。

我们将通过实现`AsyncUncaughtExceptionHandler`接口来创建一个定制的异步异常处理程序。当有任何未捕获的异步异常时，调用`handleUncaughtException()` 方法:

```java
public class CustomAsyncExceptionHandler
  implements AsyncUncaughtExceptionHandler {

    @Override
    public void handleUncaughtException(
      Throwable throwable, Method method, Object... obj) {

        System.out.println("Exception message - " + throwable.getMessage());
        System.out.println("Method name - " + method.getName());
        for (Object param : obj) {
            System.out.println("Parameter value - " + param);
        }
    }

}
```

在上一节中，我们看了由配置类实现的`AsyncConfigurer`接口。作为其中的一部分，我们还需要覆盖`getAsyncUncaughtExceptionHandler()`方法来返回我们的自定义异步异常处理程序:

```java
@Override
public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
    return new CustomAsyncExceptionHandler();
}
```

## 6。结论

在本文中，我们看了用 Spring 运行异步代码的**。**

我们从最基本的配置和注释开始，让它工作。但是我们也研究了更高级的配置，比如提供我们自己的执行器或异常处理策略。

和往常一样，本文给出的完整代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221008082221/https://github.com/eugenp/tutorials/tree/master/spring-scheduling)