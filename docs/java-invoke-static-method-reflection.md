# 使用 Java 反射 API 调用静态方法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-invoke-static-method-reflection>

## 1.概观

在这个快速教程中，我们将讨论如何通过使用[反射](/web/20221208143917/https://www.baeldung.com/java-reflection) API 来调用 Java 中的静态方法。

我们将讨论两种不同的场景:

*   静态方法是`public`。
*   静态方法是`private.`

## 2.一个示例类

为了便于演示和解释，我们先创建一个`GreetingAndBye`类作为示例:

```java
public class GreetingAndBye {

    public static String greeting(String name) {
        return String.format("Hey %s, nice to meet you!", name);
    }

    private static String goodBye(String name) {
        return String.format("Bye %s, see you next time.", name);
    }
} 
```

这个类看起来非常简单。它有两个`static`方法，一个`public`和一个`private`。

两种方法都接受一个`String`参数并返回一个`String`作为结果。

现在，让我们使用 Java 反射 API 调用这两个静态方法。在本教程中，我们将把代码称为单元测试方法。

## 3.调用`public` `static`方法

首先，让我们看看如何调用`public` `static`方法:

```java
@Test
void invokePublicMethod() throws NoSuchMethodException, InvocationTargetException, IllegalAccessException {
    Class<GreetingAndBye> clazz = GreetingAndBye.class;
    Method method = clazz.getMethod("greeting", String.class);

    Object result = method.invoke(null, "Eric");

    Assertions.assertEquals("Hey Eric, nice to meet you!", result);
} 
```

我们应该注意，当我们使用反射 API 时，我们需要处理所需的[检查异常](/web/20221208143917/https://www.baeldung.com/java-checked-unchecked-exceptions#checked)。

在上面的例子中，我们首先获得我们想要测试的类的实例，即`GreetingAndBye`。

有了类实例之后，我们可以通过调用`[getMethod](https://web.archive.org/web/20221208143917/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Class.html#getMethod(java.lang.String,java.lang.Class...))`方法来获得公共静态方法对象。

一旦我们持有了`method`对象，我们可以简单地通过调用`[invoke](https://web.archive.org/web/20221208143917/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/reflect/Method.html#invoke(java.lang.Object,java.lang.Object...))`方法来调用它。

有必要解释一下`invoke`方法的第一个论点。如果该方法是实例方法，则第一个参数是从中调用基础方法的对象。

然而，**当我们调用一个静态方法时，我们将`null`作为第一个参数**传递，因为静态方法不需要实例来被调用。

最后，如果我们运行测试，它会通过。

## 3.调用`private` `static`方法

调用`private` `static`方法与调用`public`方法非常相似。让我们先来看看代码:

```java
@Test
void invokePrivateMethod() throws NoSuchMethodException, InvocationTargetException, IllegalAccessException {
    Class<GreetingAndBye> clazz = GreetingAndBye.class;
    Method method = clazz.getDeclaredMethod("goodBye", String.class);
    method.setAccessible(true);

    Object result = method.invoke(null, "Eric");

    Assertions.assertEquals("Bye Eric, see you next time.", result);
} 
```

在上面的代码中我们可以看到，**当我们试图获取一个`private`方法的`Method`对象时，我们应该使用 [`getDeclaredMethod`](/web/20221208143917/https://www.baeldung.com/java-method-reflection#2-getdeclaredmethod) 而不是 [`getMethod`](/web/20221208143917/https://www.baeldung.com/java-method-reflection#1-getmethod)** 。

此外，**我们需要调用`[method.setAccessible(true)](https://web.archive.org/web/20221208143917/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/reflect/Method.html#setAccessible(boolean))`来调用`private`方法**。这将要求 JVM 取消对此方法的访问控制检查。

因此，它允许我们调用私有方法。否则，将引发一个`IllegalAccessException`异常。

如果我们执行它，测试就会通过。

## 4.结论

在这篇短文中，我们讨论了如何使用 Java 反射 API 调用静态方法。

和往常一样，完整的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221208143917/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-reflection-2)