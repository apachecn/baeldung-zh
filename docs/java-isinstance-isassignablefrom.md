# class . is instance vs class . isassignablefrom 和 instanceof

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-isinstance-isassignablefrom>

## 1.介绍

在这个快速教程中，我们将看看[`instanceof`](/web/20220626075718/https://www.baeldung.com/java-instanceof)`Class.isInstance`和`Class.isAssignableFrom`之间的区别。我们将学习如何使用每种方法以及它们之间的区别。

## 2.设置

让我们在探索`instanceof`、`Class.isInstance`和`Class.isAssignableFrom`功能时设置一个接口和几个类。

首先，让我们定义一个接口:

```java
public interface Shape {
}
```

接下来，让我们定义一个实现`Shape`的类:

```java
public class Triangle implements Shape {
}
```

现在，我们将创建一个扩展`Triangle`的类:

```java
public class IsoscelesTriangle extends Triangle {
}
```

## 3.`instanceof`

`instanceof`关键字是一个二元运算符，我们可以用它来验证某个对象是否是给定类型的实例。因此，运算的结果不是`true`就是`false`。此外，`instanceof`关键字是检查一个对象是否是另一个类型的子类型的最常见和最直接的方法。

让我们使用带有`instanceof`操作符的类:

```java
Shape shape = new Triangle();
Triangle triangle = new Triangle();
IsoscelesTriangle isoscelesTriangle = new IsoscelesTriangle();
Shape nonspecificShape = null;

assertTrue(shape instanceof Shape);
assertTrue(triangle instanceof Shape);
assertTrue(isoscelesTriangle instanceof Shape);
assertFalse(nonspecificShape instanceof Shape);

assertTrue(shape instanceof Triangle);
assertTrue(triangle instanceof Triangle);
assertTrue(isoscelesTriangle instanceof Triangle);
assertFalse(nonspecificShape instanceof Triangle);

assertFalse(shape instanceof IsoscelesTriangle);
assertFalse(triangle instanceof IsoscelesTriangle);
assertTrue(isoscelesTriangle instanceof IsoscelesTriangle);
assertFalse(nonspecificShape instanceof IsoscelesTriangle);
```

通过上面的代码片段，我们可以看到右边的类型**比左边的对象**更通用。更具体地说，`instanceof`操作符将把`null`值处理成`false`。

## 4.`Class.isInstance`

**`Class`类上的`isInstance`方法相当于`instanceof`操作符。**Java 1.1 中引入了`isInstance`方法，因为它可以动态使用。通常，如果参数不是`null`，该方法将返回`true`，并且可以成功地转换为引用类型，而无需引发`ClassCastException`。

让我们看看如何将`isInstance`方法用于我们定义的接口和类:

```java
Shape shape = new Triangle();
Triangle triangle = new Triangle();
IsoscelesTriangle isoscelesTriangle = new IsoscelesTriangle();
Triangle isoscelesTriangle2 = new IsoscelesTriangle();
Shape nonspecificShape = null;

assertTrue(Shape.class.isInstance(shape));
assertTrue(Shape.class.isInstance(triangle));
assertTrue(Shape.class.isInstance(isoscelesTriangle));
assertTrue(Shape.class.isInstance(isoscelesTriangle2));
assertFalse(Shape.class.isInstance(nonspecificShape));

assertTrue(Triangle.class.isInstance(shape));
assertTrue(Triangle.class.isInstance(triangle));
assertTrue(Triangle.class.isInstance(isoscelesTriangle));
assertTrue(Triangle.class.isInstance(isoscelesTriangle2));

assertFalse(IsoscelesTriangle.class.isInstance(shape));
assertFalse(IsoscelesTriangle.class.isInstance(triangle));
assertTrue(IsoscelesTriangle.class.isInstance(isoscelesTriangle));
assertTrue(IsoscelesTriangle.class.isInstance(isoscelesTriangle2));
```

我们可以看到，**右手边可以等同于或者比左手边**更具体。特别是，向`isInstance`方法提供`null`会返回`false`。

## 5.`Class.isAssignableFrom`

如果语句左侧的`Class`与所提供的`Class`参数相同或者是其超类或超接口，则`Class.isAssignableFrom`方法将返回`true`。

让我们用`isAssignableFrom`方法来使用我们的类:

```java
Shape shape = new Triangle();
Triangle triangle = new Triangle();
IsoscelesTriangle isoscelesTriangle = new IsoscelesTriangle();
Triangle isoscelesTriangle2 = new IsoscelesTriangle();

assertFalse(shape.getClass().isAssignableFrom(Shape.class));
assertTrue(shape.getClass().isAssignableFrom(shape.getClass()));
assertTrue(shape.getClass().isAssignableFrom(triangle.getClass()));
assertTrue(shape.getClass().isAssignableFrom(isoscelesTriangle.getClass()));
assertTrue(shape.getClass().isAssignableFrom(isoscelesTriangle2.getClass()));

assertFalse(triangle.getClass().isAssignableFrom(Shape.class));
assertTrue(triangle.getClass().isAssignableFrom(shape.getClass()));
assertTrue(triangle.getClass().isAssignableFrom(triangle.getClass()));
assertTrue(triangle.getClass().isAssignableFrom(isoscelesTriangle.getClass()));
assertTrue(triangle.getClass().isAssignableFrom(isoscelesTriangle2.getClass()));

assertFalse(isoscelesTriangle.getClass().isAssignableFrom(Shape.class));
assertFalse(isoscelesTriangle.getClass().isAssignableFrom(shape.getClass()));
assertFalse(isoscelesTriangle.getClass().isAssignableFrom(triangle.getClass()));
assertTrue(isoscelesTriangle.getClass().isAssignableFrom(isoscelesTriangle.getClass()));
assertTrue(isoscelesTriangle.getClass().isAssignableFrom(isoscelesTriangle2.getClass()));

assertFalse(isoscelesTriangle2.getClass().isAssignableFrom(Shape.class));
assertFalse(isoscelesTriangle2.getClass().isAssignableFrom(shape.getClass()));
assertFalse(isoscelesTriangle2.getClass().isAssignableFrom(triangle.getClass()));
assertTrue(isoscelesTriangle2.getClass().isAssignableFrom(isoscelesTriangle.getClass()));
assertTrue(isoscelesTriangle2.getClass().isAssignableFrom(isoscelesTriangle2.getClass()));
```

与`isInstance`的例子一样，我们可以清楚地看到，右手边必须与左手边相同或者比左手边更具体。我们还可以看到，我们永远无法分配我们的`Shape`接口。

## 6.差异

既然我们已经列出了几个详细的例子，让我们来看看它们的不同之处。

### 6.1.语义差异

从表面上看，`instanceof`是一个 Java 语言关键字。相比之下，`isInstance`和`isAssignableFrom`都是来自`Class`类型的本地方法。

语义上，我们使用它们来验证两个编程元素之间的不同关系:

*   两个对象:我们可以测试这两个对象是否相同或相等。
*   一个对象和一个类型:我们可以检查对象是否是类型的实例。显然，`instanceof`关键字和`isInstance`方法都属于这一类。
*   两种类型:我们可以检查一种类型是否与另一种类型兼容，比如`isAssignableFrom`方法。

### 6.2.使用极限情况差异

首先，它们的区别在于一个`null`值:

```java
assertFalse(null instanceof Shape);
assertFalse(Shape.class.isInstance(null));
assertFalse(Shape.class.isAssignableFrom(null)); // NullPointerException
```

从上面的代码片段来看，`instanceof`和`isInstance`都返回`false`；然而，`isAssignableFrom`法抛出了`NullPointerException`。

其次，它们与原始类型不同:

```java
assertFalse(10 instanceof int); // illegal
assertFalse(int.class.isInstance(10));
assertTrue(Integer.class.isInstance(10));
assertTrue(int.class.isAssignableFrom(int.class));
assertFalse(float.class.isAssignableFrom(int.class));
```

正如我们所见，`instanceof`关键字不支持基本类型。如果我们使用带有`int`值的`isInstance`方法，Java 编译器会将`int`值转换成一个`Integer`对象。因此，`isInstance`方法支持原始类型，但总是返回`false`。当我们使用`isAssignableFrom`方法时，结果取决于确切的类型值。

第三，它们与类实例变量不同:

```java
Shape shape = new Triangle();
Triangle triangle = new Triangle();
Class<?> clazz = shape.getClass();

assertFalse(triangle instanceof clazz); // illegal
assertTrue(clazz.isInstance(triangle));
assertTrue(clazz.isAssignableFrom(triangle.getClass()));
```

从上面的代码片段中，我们意识到`isInstance`和`isAssignableFrom`方法都支持类实例变量，但是`instanceof`关键字不支持。

### 6.3.字节码差异

在编译后的类文件中，它们使用不同的操作码:

*   `instanceof`关键字对应于`instanceof`操作码
*   `isInstance`和`isAssignableFrom`方法都将使用`invokevirtual`操作码

在 [JVM 指令集](https://web.archive.org/web/20220626075718/https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html#jvms-6.5.instanceof)中，`instanceof`操作码的值为 193，它有一个两字节的操作数:

```java
instanceof
indexbyte1
indexbyte2
```

然后，JVM 会将`(indexbyte1 << 8) | indexbyte2`计算成一个`index`。这个`index`指向当前类的运行时常量池。在索引处，运行时常数池包含对一个 [`CONSTANT_Class_info`](https://web.archive.org/web/20220626075718/https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.4.1) 常数的符号引用。并且，这个常量正是`instanceof`关键字右边需要的值。

此外，它还解释了为什么`instanceof`关键字不能使用类实例变量。这是因为`instanceof`操作码需要运行时常量池中的常量类型，我们不能用类实例变量替换常量类型。

并且，`instanceof`关键字的左侧对象信息存储在哪里？它在操作数堆栈的顶部。因此，`instanceof`关键字需要操作数堆栈上的一个对象和运行时常量池中的一个常量类型。

在 [JVM 指令集](https://web.archive.org/web/20220626075718/https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html#jvms-6.5.invokevirtual)中，`invokevirtual`操作码的值为 182，它还有一个双字节操作数:

```java
invokevirtual
indexbyte1
indexbyte2
```

类似地，JVM 会将`(indexbyte1 << 8) | indexbyte2`计算成一个`index`。在`index`，运行时常量池保存对一个 [`CONSTANT_Methodref_info`](https://web.archive.org/web/20220626075718/https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.4.2) 常量的符号引用。该常量包含目标方法信息，如类名、方法名和方法描述符。

`isInstance`方法在操作数栈上需要两个元素:第一个元素是类型；第二个元素是对象。然而，`isAssignableFrom`方法在操作数堆栈上需要两个类型元素。

### 6.4.总结

综上所述，让我们用一个表格来说明它们的区别:

| 财产 | `instanceof` | `Class.isInstance` | `Class.isAssignableFrom` |
| --- | --- | --- | --- |
| 种类 | 关键字 | 本地方法 | 本地方法 |
| 操作数 | 一个对象和一个类型 | 一个类型和一个对象 | 一种类型和另一种类型 |
| 零处理 | `false` | `false` | `NullPointerException` |
| 原始类型 | 不支持 | 支持，但总是`false` | 是 |
| 类实例变量 | 不 | 是 | 是 |
| 字节码 | `instanceof` | `invokevirtual` | `invokevirtual` |
| 最适合何时 | 对象给定，类型在编译时已知 | 给定了对象，但目标类型在编译类型时未知 | 没有给定对象，只有类型是已知的，并且只在运行时 |
| 用例 | 日常使用，适合大多数情况 | 复杂和不典型的情况，例如使用反射 API 实现一个库或实用程序 |

## 7.结论

在本教程中，我们查看了`instanceof`、`Class.isInstance`和`Class.isAssignableFrom`方法，并探究了它们的用法和区别。

一如既往，本教程的示例代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220626075718/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-3)