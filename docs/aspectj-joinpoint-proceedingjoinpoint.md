# AspectJ 中的 Joinpoint 与 ProceedingJoinPoint

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/aspectj-joinpoint-proceedingjoinpoint>

## 1.介绍

在这个简短的教程中，我们将了解在`[AspectJ](/web/20220523234138/https://www.baeldung.com/aspectj).`中`JoinPoint`和`ProceedingJoinPoint`接口的区别

我们将用简短的解释和代码示例来介绍它。

## 2.`JoinPoint`

`JoinPoint`是一个`AspectJ`接口，由**提供对给定连接点**处可用状态的反射访问，比如方法参数、返回值或抛出的异常。它还提供了关于方法本身的所有静态信息。

我们可以将它与`@Before`、`@After`、`@AfterThrowing`和`@AfterReturning`建议一起使用。这些切入点将分别在方法执行之前、执行之后、返回值之后、或者仅在抛出异常之后、或者仅在方法返回值之后启动。

为了更好的理解，我们来看一个基本的例子。首先，我们需要声明一个切入点。我们将定义来自`ArticleService`类的`getArticleList()`的每次执行:

```
@Pointcut("execution(* com.baeldung.ArticleService.getArticleList(..))")
public void articleListPointcut(){ }
```

接下来，我们可以定义建议。在我们的例子中，我们将使用`@Before`:

```
@Before("articleListPointcut()")
public void beforeAdvice(JoinPoint joinPoint) {
    log.info(
      "Method {} executed with {} arguments",
      joinPoint.getStaticPart().getSignature(),
      joinPoint.getArgs()
    );
}
```

在上面的例子中，我们使用`@Before`通知来记录方法执行及其参数。一个类似的用例是记录代码中出现的异常:

```
@AfterThrowing(
  pointcut = "articleListPointcut()",
  throwing = "e"
)
public void logExceptions(JoinPoint jp, Exception e) {
    log.error(e.getMessage(), e);
}
```

通过使用`@AfterThrowing`建议，我们确保只有在异常发生时才进行日志记录。

## 3.`ProceedingJoinPoint`

`ProceedingJoinPoint`是`JoinPoint`的扩展，它公开了额外的`proceed()`方法。调用时，代码执行跳转到下一个通知或目标方法。**它给了我们控制代码流**并决定是否继续进一步调用的权力。

它可能只是围绕整个方法调用的`@Around`建议:

```
@Around("articleListPointcut()")
public Object aroundAdvice(ProceedingJoinPoint pjp) {
    Object articles = cache.get(pjp.getArgs());
    if (articles == null) {
        articles = pjp.proceed(pjp.getArgs());
    }
    return articles;
}
```

在上面的例子中，我们举例说明了`@Around`建议最流行的用法之一。只有当缓存没有返回结果时，才会调用实际的方法。**这正是 [Spring Cache 注解](/web/20220523234138/https://www.baeldung.com/spring-cache-tutorial)的工作方式。**

如果出现任何异常，我们也可以使用`ProceedingJoinPoint`和`@Around`建议来重试操作:

```
@Around("articleListPointcut()")
public Object aroundAdvice(ProceedingJoinPoint pjp) {
    try {
        return pjp.proceed(pjp.getArgs());
    } catch (Throwable) {
        log.error(e.getMessage(), e);
        log.info("Retrying operation");
        return pjp.proceed(pjp.getArgs());
    }
}
```

例如，该解决方案可用于在网络中断的情况下重试 HTTP 调用。

## 4.结论

在这篇文章中，我们了解了`AspectJ`中`Joinpoint`和`ProceedingJoinPoint`的区别。和往常一样，所有的源代码都可以在 GitHub 的[上获得。](https://web.archive.org/web/20220523234138/https://github.com/eugenp/tutorials/tree/master/spring-aop)