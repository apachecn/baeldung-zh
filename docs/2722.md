# Java 什么时候抛出 UndeclaredThrowableException？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-undeclaredthrowableexception>

## 1.概观

在本教程中，我们将看到是什么导致 Java 抛出了一个`UndeclaredThrowableException `异常的实例。

首先，我们将从一些理论开始。然后，我们将通过两个真实的例子来更好地理解这个异常的本质。

## 2.`UndeclaredThrowableException`

理论上讲，当我们试图抛出一个未声明的[检查异常](/web/20220628060946/https://www.baeldung.com/java-checked-unchecked-exceptions)时，Java 会抛出一个`UndeclaredThrowableException`的实例。**也就是说，我们没有在`throws `子句中声明被检查的异常，但是我们在方法体中抛出了那个异常。**

有人可能会说这是不可能的，因为 Java 编译器用一个编译错误阻止了这一点。例如，如果我们试图编译:

```
public void undeclared() {
    throw new IOException();
}
```

Java 编译器失败，并显示以下消息:

```
java: unreported exception java.io.IOException; must be caught or declared to be thrown
```

尽管抛出未声明的检查异常可能不会在编译时发生，但在运行时仍有可能发生。例如，让我们考虑一个运行时代理拦截一个不抛出任何异常的方法:

```
public void save(Object data) {
    // omitted
}
```

**如果代理本身抛出一个检查过的异常，从调用者的角度来看，`save `方法抛出那个检查过的异常。**调用者可能不知道任何关于这个代理的事情，并且会将这个异常归咎于`save `。

**在这种情况下，Java 会将实际检查的异常封装在一个`UndeclaredThrowableException `中，并抛出`UndeclaredThrowableException `来代替。**值得一提的是，`UndeclaredThrowableException `本身就是一个未检查的异常。

既然我们已经对这个理论有了足够的了解，让我们来看几个现实世界的例子。

## 3.Java 动态代理

作为我们的第一个例子，让我们为`java.util.List `接口创建一个[运行时代理](/web/20220628060946/https://www.baeldung.com/java-dynamic-proxies)，并拦截它的方法调用。首先，我们应该实现`[InvocationHandler](https://web.archive.org/web/20220628060946/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/reflect/InvocationHandler.html) `接口，并将额外的逻辑放在那里:

```
public class ExceptionalInvocationHandler implements InvocationHandler {

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        if ("size".equals(method.getName())) {
            throw new SomeCheckedException("Always fails");
        }

        throw new RuntimeException();
    }
}

public class SomeCheckedException extends Exception {

    public SomeCheckedException(String message) {
        super(message);
    }
}
```

如果被代理的方法是`size. `，这个代理抛出一个检查过的异常，否则，它将抛出一个未检查的异常。

让我们看看 Java 是如何处理这两种情况的。首先，我们将调用`List.size() `方法:

```
ClassLoader classLoader = getClass().getClassLoader();
InvocationHandler invocationHandler = new ExceptionalInvocationHandler();
List<String> proxy = (List<String>) Proxy.newProxyInstance(classLoader, 
  new Class[] { List.class }, invocationHandler);

assertThatThrownBy(proxy::size)
  .isInstanceOf(UndeclaredThrowableException.class)
  .hasCauseInstanceOf(SomeCheckedException.class);
```

如上所示，我们为`List `接口创建了一个代理，并在其上调用了`size `方法。**代理反过来拦截调用并抛出一个检查过的异常。然后，Java 将这个检查过的异常封装在`UndeclaredThrowableException.`** 的一个实例中，这是因为我们以某种方式抛出了一个检查过的异常，而没有在方法声明中声明它。

如果我们在`List `接口上调用任何其他方法:

```
assertThatThrownBy(proxy::isEmpty).isInstanceOf(RuntimeException.class);
```

由于代理抛出了一个未检查的异常，Java 让异常按原样传播。

## 4.春天的样子

当我们在一个 [Spring 方面](/web/20220628060946/https://www.baeldung.com/spring-aop)中抛出一个被检查的异常，而被通知的方法没有声明它们时，也会发生同样的事情。让我们从注释开始:

```
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface ThrowUndeclared {}
```

现在我们要通知所有用这个注释标注的方法:

```
@Aspect
@Component
public class UndeclaredAspect {

    @Around("@annotation(undeclared)")
    public Object advise(ProceedingJoinPoint pjp, ThrowUndeclared undeclared) throws Throwable {
        throw new SomeCheckedException("AOP Checked Exception");
    }
}
```

基本上，这个建议会让所有带注释的方法抛出一个检查过的异常，即使它们没有声明这样的异常。现在，让我们创建一个服务:

```
@Service
public class UndeclaredService {

    @ThrowUndeclared
    public void doSomething() {}
}
```

如果我们调用带注释的方法，Java 将抛出一个`UndeclaredThrowableException `异常的实例:

```
@RunWith(SpringRunner.class)
@SpringBootTest(classes = UndeclaredApplication.class)
public class UndeclaredThrowableExceptionIntegrationTest {

    @Autowired private UndeclaredService service;

    @Test
    public void givenAnAspect_whenCallingAdvisedMethod_thenShouldWrapTheException() {
        assertThatThrownBy(service::doSomething)
          .isInstanceOf(UndeclaredThrowableException.class)
          .hasCauseInstanceOf(SomeCheckedException.class);
    }
}
```

如上所示，Java 将实际的异常封装为原因，并抛出`UndeclaredThrowableException `异常。

## 5.结论

在本教程中，我们看到了是什么导致 Java 抛出了一个`UndeclaredThrowableException `异常的实例。

像往常一样，所有的例子都可以在 GitHub 上找到[。](https://web.archive.org/web/20220628060946/https://github.com/eugenp/tutorials/tree/master/spring-aop-2)