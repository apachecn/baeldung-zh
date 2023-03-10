# Java(字符串)或者。toString()？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-string-casting-vs-tostring>

## 1.介绍

在本文中，我们将简要解释**`String`铸造和执行`toString()`** **方法**的区别。我们将简要回顾这两种语法，并通过一个例子解释使用它们的目的。最后，我们将看看哪一个是更好的方法。

## 2.`String` 式铸造和 **`toString()`** 法铸造

让我们先快速回顾一下。使用`(String) `语法与 Java 中的[类型转换紧密相关。简而言之，使用这种语法的主要任务是**将一个源变量转换成** `**String**:`](/web/20221129000201/https://www.baeldung.com/java-type-casting)

```java
String str = (String) object; 
```

我们知道，Java 中的每个类都是`Object` 类的直接或间接扩展，它实现了[`toStr``ing()`方法](/web/20221129000201/https://www.baeldung.com/java-tostring)。我们用它来**得到任何`Object`** 的一个`String`表示:

```java
String str = object.toString();
```

现在我们已经做了一个简短的回顾，让我们通过一些例子来帮助理解何时使用每种方法。

## 3.`(String)`对`toString()`

假设我们有一个`Object`变量，我们想要获得一个`String`。我们应该使用哪种语法？

在继续之前，我们应该强调，下面的实用方法只是用来帮助解释我们的主题。实际上，我们不会像这样使用实用方法。

首先，让我们介绍一个简单的实用方法，将一个`Object`转换成一个`String`:

```java
public static String castToString(Object object) {
    if (object instanceof String) {
        return (String) object;
    }
    return null;
}
```

正如我们所看到的，在造型之前，我们必须检查我们的`object` 变量是一个`String`的实例。如果我们不这样做，它可能会失败并产生一个`ClassCastException`:

```java
@Test(expected = ClassCastException.class)
public void givenIntegerObject_whenCastToObjectAndString_thenCastClassException() {
    Integer input = 1234;

    Object obj = input;
    String str = (String) obj;
}
```

但是，这个操作是空安全的。在非实例化变量上使用它，即使它以前没有被应用于`String`变量，也会成功:

```java
@Test
public void givenNullInteger_whenCastToObjectAndString_thenSameAndNoException() {
    Integer input = null;

    Object obj = input;
    String str = (String) obj;

    assertEquals(obj, str);
    assertEquals(str, input);
    assertSame(input, str);
}
```

现在，是时候在请求的对象上实现另一个调用`toString()`的实用函数了:

```java
public static String getStringRepresentation(Object object) {
    if (object != null) {
        return object.toString();
    }
    return null;
}
```

在这种情况下，我们不需要知道对象的类型，它可以在没有类型转换的对象上成功执行。我们只需添加一个简单的`null`检查。如果我们不添加这个检查，当向方法传递一个非实例化变量时，我们可能会得到一个`NullPointerException`:

```java
@Test(expected = NullPointerException.class)
public void givenNullInteger_whenToString_thenNullPointerException() {
    Integer input = null;

    String str = input.toString();
}
```

此外，由于核心的`String`实现，对`String`变量执行`toString()`方法会返回相同的对象:

```java
@Test
public void givenString_whenToString_thenSame() {
    String str = "baeldung";

    assertEquals("baeldung", str.toString());
    assertSame(str, str.toString());
}
```

让我们回到我们的问题——我们应该在对象变量上使用哪种语法？正如我们在上面看到的，如果我们知道**我们的变量是一个`String`实例，我们应该使用类型转换**:

```java
@Test
public void givenString_whenCastToObject_thenCastToStringReturnsSame() {
    String input = "baeldung";

    Object obj = input;

    assertSame(input, StringCastUtils.castToString(obj));
}
```

这种方法通常更有效、更快，因为我们不需要执行额外的函数调用。但是，让我们记住，我们永远不应该把一个`String`当作一个`Object`来传递。这将暗示我们有代码气味。

当我们传递**任何其他对象类型时，我们需要显式调用`toString()`方法**。重要的是要记住它根据实现返回一个`String`值:

```java
@Test
public void givenIntegerNotNull_whenCastToObject_thenGetToStringReturnsString() {
    Integer input = 1234;

    Object obj = input;

    assertEquals("1234", StringCastUtils.getStringRepresentation(obj));
    assertNotSame("1234", StringCastUtils.getStringRepresentation(obj));
}
```

## 4.结论

在这个简短的教程中，我们比较了两种方法:`String`类型转换和使用`toString()`方法获取字符串表示。通过例子，我们解释了不同之处，并探索了何时使用(`String)`或`toString()`)。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20221129000201/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-operations-3)