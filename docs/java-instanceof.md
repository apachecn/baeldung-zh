# Java instanceof 运算符

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-instanceof>

## 1.介绍

在这个快速教程中，我们将学习 Java 中的`instanceof `操作符。

## 2.什么是`instanceof`运算符？

我们用来测试一个对象是否属于给定类型的二元运算符。运算的结果不是`true`就是`false`。它也被称为类型比较运算符，因为它将实例与类型进行比较。

在[施放](/web/20220910004940/https://www.baeldung.com/java-type-casting)未知物体之前，应该一直使用`instanceof`检查。这样做有助于在运行时避免`ClassCastException`。

`instanceof`运算符的基本语法是:

```
(object) instanceof (type)
```

现在让我们看一个`instanceof `操作符的基本例子。首先，我们将创建一个类`Round`:

```
public class Round {
    // implementation details
}
```

接下来，我们将创建一个扩展了`Round`的类`Ring`:

```
public class Ring extends Round {
    // implementation details
}
```

我们可以使用`instanceof`来检查`Ring`的实例是否属于`Round`类型:

```
@Test
public void givenWhenInstanceIsCorrect_thenReturnTrue() {
    Ring ring = new Ring();
    Assert.assertTrue(ring instanceof Round);
}
```

## 3.`instanceof`运算符是如何工作的？

**`instanceof`运算符的工作原理是“是-是”关系**。is-a 关系的概念基于类[继承](/web/20220910004940/https://www.baeldung.com/java-inheritance-composition)或接口实现。

为了演示这一点，我们将创建一个`Shape`接口:

```
public interface Shape {
    // implementation details
}
```

我们还将创建一个类`Circle,`，它实现了`Shape`接口并扩展了`Round`类:

```
public class Circle extends Round implements Shape {
    // implementation details
}
```

**如果对象是类型:**的实例，`instanceof`结果将是`true`

```
@Test
public void givenWhenObjectIsInstanceOfType_thenReturnTrue() {
    Circle circle = new Circle();
    Assert.assertTrue(circle instanceof Circle);
}
```

**如果对象是**类型的子类的实例，它也将是`true`

```
@Test
public void giveWhenInstanceIsOfSubtype_thenReturnTrue() {
    Circle circle = new Circle();
    Assert.assertTrue(circle instanceof Round);
}
```

**如果类型是接口，则返回`true`如果对象实现接口:**

```
@Test
public void givenWhenTypeIsInterface_thenReturnTrue() {
    Circle circle = new Circle();
    Assert.assertTrue(circle instanceof Shape);
}
```

**如果被比较的对象和被比较的类型之间没有关系，就不能使用 `instanceof`操作符。**

我们将创建一个新的类，`Triangle,` ，它实现了`Shape,` ，但是与`Circle`没有关系:

```
public class Triangle implements Shape {
    // implementation details
}
```

现在，如果我们使用`instanceof`来检查`Circle`是否是`Triangle`的实例:

```
@Test
public void givenWhenComparingClassInDiffHierarchy_thenCompilationError() {
    Circle circle = new Circle();
    Assert.assertFalse(circle instanceof Triangle);
}
```

我们将得到一个编译错误，因为`Circle`和`Triangle`类之间没有关系:

```
java.lang.Error: Unresolved compilation problem:
  Incompatible conditional operand types Circle and Triangle
```

## 4.将`instanceof`与`Object`类型一起使用

在 Java 中，每个类都隐式继承自`Object`类。因此，使用带有`Object`类型的`instanceof`操作符将始终计算为`true`:

```
@Test
public void givenWhenTypeIsOfObjectType_thenReturnTrue() {
    Thread thread = new Thread();
    Assert.assertTrue(thread instanceof Object);
}
```

## 5.当对象为`null`时，使用`instanceof`运算符

如果我们对任何一个`null`对象使用` instanceof`操作符，它将返回`false`。当使用`instanceof`操作符时，我们也不需要空检查。

```
@Test
public void givenWhenInstanceValueIsNull_thenReturnFalse() {
    Circle circle = null;
    Assert.assertFalse(circle instanceof Round);
}
```

## 6.`instanceof`和仿制药

**实例测试和转换依赖于在运行时检查类型信息。因此，我们不能使用`instanceof `连同[一起擦除通用类型](/web/20220910004940/https://www.baeldung.com/java-type-erasure)** 。

例如，如果我们试图编译下面的代码片段:

```
public static <T> void sort(List<T> collection) {
    if (collection instanceof List<String>) {
        // sort strings differently
    }

    // omitted
}
```

然后我们得到这个编译错误:

```
error: illegal generic type for instanceof
        if (collection instanceof List<String>) {
                                      ^
```

**从技术上讲，在 Java 中我们只允许将`instanceof` 和具体化的类型一起使用。**如果一个类型的类型信息在运行时存在，则该类型被具体化。

Java 中的具体化类型如下:

*   原始类型，如`int`
*   非泛型类和接口，如`String `或`Random`
*   所有类型都是无界通配符的泛型类型，如`Set<?>` 或 `Map<?, ?>`
*   原始类型，如`List or HashMap`
*   其他可具体化类型的数组，如`String[], List[], `或`Map<?, ?>[]`

因为泛型类型参数没有具体化，所以我们也不能使用它们:

```
public static <T> boolean isOfType(Object input) {
    return input instanceof T; // won't compile
}
```

然而，可以针对类似`List<?>`的东西进行测试:

```
if (collection instanceof List<?>) {
    // do something
}
```

## 7.结论

在这篇简短的文章中，我们学习了`instanceof`操作符以及如何使用它。完整的代码样本可以在 GitHub 的[上找到。](https://web.archive.org/web/20220910004940/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-operators)