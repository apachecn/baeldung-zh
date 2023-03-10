# 实现定制的 Spring AOP 注释

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-aop-annotation>

## 1。简介

在本文中，我们将使用 Spring 中的 AOP 支持实现一个定制的 AOP 注释。

首先，我们将给出 AOP 的高级概述，解释它是什么以及它的优点。接下来，我们将一步一步地实现我们的注释，逐渐建立对 AOP 概念更深入的理解。

其结果将是对 AOP 的更好理解，以及将来创建自定义 Spring 注释的能力。

## 2。什么是 AOP 注释？

简而言之，AOP 代表面向方面编程。本质上，**它是一种在不修改代码**的情况下向现有代码添加行为的方法。

关于 AOP 的详细介绍，有关于 AOP [切入点](/web/20220902075953/https://www.baeldung.com/spring-aop-pointcut-tutorial)和[建议](/web/20220902075953/https://www.baeldung.com/spring-aop-advice-tutorial)的文章。本文假设我们已经有了基本的知识。

我们将在本文中实现的 AOP 类型是注释驱动的。如果我们使用过弹簧 [`@Transactional`](https://web.archive.org/web/20220902075953/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/annotation/Transactional.html) 的注释，我们可能已经很熟悉了:

```java
@Transactional
public void orderGoods(Order order) {
   // A series of database calls to be performed in a transaction
}
```

这里的关键是无创性。通过使用注释元数据，我们的核心业务逻辑不会被我们的事务代码污染。这使得推理、重构和孤立测试变得更加容易。

有时候，开发 Spring 应用程序的人会把这看作是 `‘` Spring Magic ,,而不去考虑它是如何工作的。事实上，正在发生的事情并不特别复杂。然而，一旦我们完成了本文中的步骤，我们将能够创建我们自己的定制注释，以便理解和利用 AOP。

## 3。Maven 依赖关系

首先，让我们添加我们的 [Maven 依赖项](https://web.archive.org/web/20220902075953/https://search.maven.org/classic/#search%7Cga%7C1%7C%20(g%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-parent%22)%20OR%20%20(g%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter%22)%20OR%20%20(g%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-aop%22))。

在本例中，我们将使用 Spring Boot，因为它的配置方法传统让我们可以尽快启动并运行:

```java
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.2.RELEASE</version>
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-aop</artifactId>
    </dependency>
</dependencies>
```

注意，我们已经包含了 AOP starter，它引入了我们开始实现方面所需的库。

## 4。创建我们的自定义注释

我们将要创建的注释将用于记录一个方法执行所花费的时间。让我们创建我们的注释:

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface LogExecutionTime {

} 
```

尽管这是一个相对简单的实现，但值得注意的是这两个元注释的用途。

`[@Target](https://web.archive.org/web/20220902075953/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/annotation/Target.html)` 注释告诉我们我们的注释将适用于哪里。这里我们使用了`ElementType.Method,` ,这意味着它只对方法有效。如果我们试图在其他地方使用注释，那么我们的代码将无法编译。这种行为是有意义的，因为我们的注释将用于记录方法执行时间。

而 [`@Retention`](https://web.archive.org/web/20220902075953/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/annotation/Retention.html) 只是声明注解在运行时是否对 JVM 可用。默认情况下不是这样，所以 Spring AOP 看不到注释。这就是它被重新配置的原因。

## 5。创建我们的方面

现在我们有了注释，让我们创建我们的方面。这只是封装我们横切关注点的模块，我们的例子是方法执行时间日志。它只是一个类，注释为`[@Aspect](https://web.archive.org/web/20220902075953/https://www.eclipse.org/aspectj/doc/released/aspectj5rt-api/org/aspectj/lang/annotation/Aspect.html?is-external=true):`

```java
@Aspect
@Component
public class ExampleAspect {

}
```

我们还包含了`[@Component](https://web.archive.org/web/20220902075953/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/stereotype/Component.html)` 注释，因为我们的类也需要是一个 Spring bean 才能被检测到。本质上，这是我们将实现我们希望自定义注释注入的逻辑的类。

## 6。创建我们的切入点和建议

现在，让我们创建切入点和建议。这将是一个存在于我们方面的带注释的方法:

```java
@Around("@annotation(LogExecutionTime)")
public Object logExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable {
    return joinPoint.proceed();
}
```

从技术上来说，这还没有改变任何事情的行为，但是仍然有很多事情需要分析。

首先，我们用`[@Around](https://web.archive.org/web/20220902075953/https://www.eclipse.org/aspectj/doc/released/aspectj5rt-api/org/aspectj/lang/annotation/Around.html?is-external=true).` 注释了我们的方法，这是我们的建议，围绕建议意味着我们在方法执行前后都添加了额外的代码。还有其他类型的建议，比如`before` 和`after`，但它们不在本文讨论范围之内。

接下来，我们的`@Around` 注释有一个点切参数。我们的切入点只是说，‘将这个建议应用于任何用`@LogExecutionTime`注释的方法。’还有许多其他类型的切入点，但是如果超出范围，它们将再次被忽略。

方法本身就是我们的建议。只有一个参数，在我们的例子中是`[ProceedingJoinPoint](https://web.archive.org/web/20220902075953/https://www.eclipse.org/aspectj/doc/next/runtime-api/org/aspectj/lang/ProceedingJoinPoint.html).` ，这将是一个执行方法，已经用`@LogExecutionTime.` 进行了注释

最后，当我们的带注释的方法最终被调用时，将会发生的是我们的建议将首先被调用。然后由我们的建议来决定下一步做什么。在我们的例子中，我们的建议是除了调用`proceed(),` 之外什么也不做，这只是调用原始的带注释的方法。

## 7 .**。记录我们的执行时间**

现在我们已经有了框架，我们需要做的就是给我们的建议添加一些额外的逻辑。除了调用原始方法之外，这将记录执行时间。让我们将这个额外的行为添加到我们的建议中:

```java
@Around("@annotation(LogExecutionTime)")
public Object logExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable {
    long start = System.currentTimeMillis();

    Object proceed = joinPoint.proceed();

    long executionTime = System.currentTimeMillis() - start;

    System.out.println(joinPoint.getSignature() + " executed in " + executionTime + "ms");
    return proceed;
}
```

同样，我们没有做任何特别复杂的事情。我们刚刚记录了当前时间，执行了该方法，然后在控制台上打印了所用的时间。我们还记录了方法签名，它是为了使用`joinpoint` 实例而提供的。如果我们愿意的话，我们还可以访问其他信息，比如方法参数。

现在，让我们尝试用`@LogExecutionTime,` 注释一个方法，然后执行它，看看会发生什么。请注意，这必须是一个 Spring Bean 才能正常工作:

```java
@LogExecutionTime
public void serve() throws InterruptedException {
    Thread.sleep(2000);
}
```

执行后，我们应该看到控制台记录了以下内容:

```java
void org.baeldung.Service.serve() executed in 2030ms
```

## 8。结论

在本文中，我们利用 Spring Boot AOP 创建了我们的自定义注释，我们可以将它应用于 Spring beans，以便在运行时为它们注入额外的行为。

我们应用程序的源代码可以在 GitHub 的[上找到；这是一个 Maven 项目，应该能够运行。](https://web.archive.org/web/20220902075953/https://github.com/eugenp/tutorials/tree/master/spring-aop)