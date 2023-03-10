# 是什么原因导致 Java . lang . reflect . invocationtargetexception？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-lang-reflect-invocationtargetexception>

## 1.概观

在使用 [Java 反射 API](/web/20221206020504/https://www.baeldung.com/java-reflection) 的时候，经常会遇到`java.lang.reflect.InvocationTargetException`。

在本教程中，我们将通过一个简单的例子`. `来看看它以及如何处理它

## 2.**起因`InvocationTargetException`**

它主要发生在我们使用反射层并试图调用一个抛出底层异常的方法或构造函数时。

**反射层用`InvocationTargetException`包装方法抛出的实际异常。**

我们试着用一个例子来理解一下。

我们将编写一个带有故意抛出异常的方法的类:

```java
public class InvocationTargetExample {
    public int divideByZeroExample() {
        return 1 / 0;
    }
}
```

让我们在一个简单的 JUnit 5 测试中使用反射调用上面的方法:

```java
InvocationTargetExample targetExample = new InvocationTargetExample(); 
Method method =
  InvocationTargetExample.class.getMethod("divideByZeroExample");

Exception exception =
  assertThrows(InvocationTargetException.class, () -> method.invoke(targetExample));
```

在上面的代码中，我们断言了在调用方法时抛出的`InvocationTargetException`。这里需要注意的重要一点是，实际的异常——在本例中为`ArithmeticException`——被包装在一个`InvocationTargetException`中。

现在，为什么反射不首先抛出实际的异常？

原因是它可以让我们了解`Exception`的发生是因为通过反射层调用方法失败，还是发生在方法本身内部。

## 3.如何处理`InvocationTargetException`？

这里实际的底层异常是`InvocationTargetException`的原因，所以我们可以**使用 `Throwable.getCause()`** 来获得关于它的更多信息。

让我们看看如何使用 `getCause()`在上面使用的同一个例子中获得实际的异常:

```java
assertEquals(ArithmeticException.class, exception.getCause().getClass());
```

我们在抛出的同一个`exception`对象上使用了`getCause()`方法。而且我们已经断言`ArithmeticException.class`是异常的原因。

因此，一旦我们得到了底层异常，我们可以重新抛出它，将它包装在一些自定义异常中，或者根据我们的要求简单地记录异常。

## 4.结论

在这篇短文中，我们看到了反射层是如何包装任何底层异常的。

我们还看到了如何确定`InvocationTargetException`的根本原因，以及如何通过一个简单的例子来处理这种情况。

像往常一样，本文中使用的代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221206020504/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-reflection)