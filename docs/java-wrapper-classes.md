# Java 中的包装类

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-wrapper-classes>

## 1。概述

顾名思义，**包装类是封装了原始 Java 类型的对象。**

每个 Java 原语都有相应的包装器:

*   `boolean, byte, short, char, int, long, float, double `
*   `Boolean, Byte, Short, Character, Integer, Long, Float,` `Double`

这些都在`java.lang`包中定义，因此我们不需要手动导入它们。

## 2。包装类

"包装类的目的是什么？"。这是最常见的 Java 面试问题之一。

基本上，**泛型类只处理对象，不支持原语**。因此，如果我们想使用它们，我们必须将原始值转换成包装对象。

例如，Java 集合框架专门处理对象。很久以前(在 Java 5 之前，大约 15 年前)没有自动装箱，例如，我们不能简单地在一个`Integers.`集合上调用`add(5)`

那时，那些 ***原语*值需要手动转换成*对应的*包装类**并存储在集合中。

今天，有了自动装箱，我们可以很容易地做`ArrayList.add(101)`，但是在 Java 内部，在使用 `valueOf()` 方法`.`将原始值存储到`ArrayList` 之前，先将其转换为`Integer`

## 3。原语到包装类的转换

现在最大的问题是:我们如何将原始值转换成相应的包装类，例如从`int`到`Integer`或者从`char`到`Character?`

嗯，我们可以使用构造函数或静态工厂方法将原始值转换为包装类的对象。

然而，从 Java 9 开始，许多装箱原语的构造函数如`[Integer](https://web.archive.org/web/20220628083101/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Integer.html#%3Cinit%3E(int)) `或 [`Long`](https://web.archive.org/web/20220628083101/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Long.html#%3Cinit%3E(long)) 已经被弃用。

所以强烈建议只在新代码上使用工厂方法。

让我们看一个在 Java 中将`int`值转换成`Integer`对象的例子:

```
Integer object = new Integer(1);

Integer anotherObject = Integer.valueOf(1);
```

`valueOf()`方法返回一个代表指定的`int`值的实例。

它返回缓存的值，这使得它的效率。它总是缓存-128 到 127 之间的值，但也可以缓存这个范围之外的其他值。

类似地，我们也可以将`boolean`转换为`Boolean, byte`转换为`Byte, char`转换为`Character, long`转换为`Long, float`转换为`Float,`以及`double`转换为`Double.`，尽管如果我们必须将[转换为整数](https://web.archive.org/web/20220628083101/https://javarevisited.blogspot.com/2011/08/convert-string-to-integer-to-string.html)，那么我们需要使用`parseInt()`方法，因为`String`不是包装类。

另一方面，**要从一个包装对象转换到一个原语值，我们可以使用相应的方法如`intValue(), doubleValue()`** 等:

```
int val = object.intValue(); 
```

综合参考可以在这里找到[。](/web/20220628083101/https://www.baeldung.com/java-primitive-conversions)

## 4。自动装箱和拆箱

在上一节中，我们展示了如何手动将原始值转换为对象。

在 Java 5 之后，这种转换可以通过自动装箱和取消装箱来自动完成。

**“装箱”是指将原语值转换成相应的包装对象。**因为这可以自动发生，所以被称为自动装箱。

类似地，当一个包装对象被展开成一个原始值时，这被称为拆箱。

这实际上意味着我们可以将一个原语值传递给一个需要包装对象的方法，或者将一个原语赋给一个需要对象的变量:

```
List<Integer> list = new ArrayList<>();
list.add(1); // autoboxing

Integer val = 2; // autoboxing
```

在这个例子中，Java 会自动将原语`int`值转换成包装器。

在内部，它使用`valueOf()`方法来促进转换。例如，以下几行是等效的:

```
Integer value = 3;

Integer value = Integer.valueOf(3);
```

尽管这使得转换更容易，代码更可读，但是有些情况下**我们不应该使用自动装箱，比如在循环**中。

与自动装箱类似，当将对象传递给需要基元的方法或将其赋给基元变量时，取消装箱是自动完成的:

```
Integer object = new Integer(1); 
int val1 = getSquareValue(object); //unboxing
int val2 = object; //unboxing

public static int getSquareValue(int i) {
    return i*i;
}
```

基本上，如果我们编写一个接受原始值或包装对象的方法，我们仍然可以将这两个值传递给它们。 Java 会根据上下文传递正确的类型，如原语或包装。

## 5。结论

在这个快速教程中，我们讨论了 Java 中的包装类，以及自动装箱和取消装箱的机制。