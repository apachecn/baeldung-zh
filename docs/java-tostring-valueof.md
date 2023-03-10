# Java 中的 Integer.toString()与 String.valueOf()

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-tostring-valueof>

## 1.概观

我们知道，从`int` 到`String `的转换是 Java 中非常常见的操作。

在这个简短的教程中，我们将介绍两个非常流行的方法，`Integer` 类的 [`toString()`和`String`](https://web.archive.org/web/20221008100127/https://docs.oracle.com/en/java/javase/18/docs/api/java.base/java/lang/Integer.html#toString()) 类的 [`valueOf()`，这有助于我们进行这种转换。此外，我们还将查看使用这两种方法的一些要点和示例，以便更好地理解它。](https://web.archive.org/web/20221008100127/https://docs.oracle.com/en/java/javase/18/docs/api/java.base/java/lang/String.html#valueOf(int))

## 2.`Integer.toString()`法

这个方法**接受一个原始数据类型的整数`int `作为参数，并返回一个表示指定整数的`String`对象。**

让我们来看看它的签名:

```java
public static String toString(int i)
```

现在，我们将看到几个示例，其中我们将有符号/无符号整数作为参数传递给它，以了解整数到字符串的转换是如何发生的:

```java
@Test
public void whenValidIntIsPassed_thenShouldConvertToString() {
    assertEquals("11", Integer.toString(11)); 
    assertEquals("11", Integer.toString(+11)); 
    assertEquals("-11", Integer.toString(-11));
}
```

## 3.`String.valueOf() `法

这个方法也接受一个原始数据类型的整数作为参数，并返回一个对象。有趣的是，**返回的字符串表示与`Integer.toString(int i)`方法返回的字符串表示完全相同。是因为在内部，它用的是`Integer.toString()`的方法。**

让我们看看它的内部实现，如在`java.lang.String`类中给出的:

```java
/**
 * Returns the string representation of the {@code int} argument.
 * <p>
 * The representation is exactly the one returned by the
 * {@code Integer.toString} method of one argument.
 *
 * @param   i   an {@code int}.
 * @return  a string representation of the {@code int} argument.
 * @see     java.lang.Integer#toString(int, int)
 */
public static String valueOf(int i) {
    return Integer.toString(i);
}
```

为了更好地理解它，我们将看几个例子，其中我们将有符号/无符号整数作为参数传递给它，以理解整数到字符串的转换是如何发生的:

```java
@Test
public void whenValidIntIsPassed_thenShouldConvertToValidString() {
    assertEquals("11", String.valueOf(11)); 
    assertEquals("11", String.valueOf(+11));
    assertEquals("-11", String.valueOf(-11));
}
```

## 4.整数之间的差异。`toString()`和弦。`valueOf()`

综上所述，这两种方法没有实际的区别，但是我们应该了解以下几点以避免混淆。

当我们使用`String.valueOf()`方法时，堆栈跟踪中有一个额外的调用，因为它在内部使用相同的`Integer.toString()`方法。

当将一个`null`对象传递给`valueOf() method because,` **时，当将一个原始的`int `传递给`valueOf()`方法时，可能会有些混乱，因为它看起来是一样的，但实际的方法调用却是不同的重载方法。**

如果给定的`Integer`对象是`null` ，那么 **`Integer.toString()`可能会抛出`NullPointerException`。 `String.valueOf()`不会抛出异常，因为它会转到`String.valueOf(Object obj)` 方法并返回`null`。**请注意，传递给`String.valueOf(int i)`的`primitive int`永远不可能是`null`，但是因为有另一个方法`String.valueOf(Object obj)`，我们可能会在这两个重载方法之间混淆。****

让我们用下面的例子来理解最后一点:

```java
@Test(expected = NullPointerException.class)
public void whenNullIntegerObjectIsPassed_thenShouldThrowException() {
    Integer i = null; 
    System.out.println(String.valueOf(i)); 
    System.out.println(i.toString());
}
```

请注意，原语`int`永远不能是`null`，我们正在检查它，以防它下面的方法抛出异常。

## 5.JVM 方法内联对`String.valueOf()`方法的影响

正如我们前面讨论的，`String.valueOf()`方法涉及到一个额外的调用。但是，JVM 甚至可以通过方法内联消除堆栈跟踪中额外调用。

然而，这完全取决于 JVM 是否选择内联该方法。要获得更详细的描述，请访问我们关于 JVM 中方法内联的文章。

## 6.结论

在这篇文章中，我们学习了整数。`toString()`和弦。`valueOf()`方法。我们还看了一些我们应该集中精力避免编程时混淆的地方。

和往常一样，本文的完整代码示例可以在 GitHub 的[中找到。](https://web.archive.org/web/20221008100127/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-conversions-2)