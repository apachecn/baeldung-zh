# 什么是【ljava . lang . object；？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-tostring-array>

## 1.概观

在本教程中，我们将学习`[Ljava.lang.Object`是什么意思，以及如何访问对象的正确值。

## 2.Java 对象类

在 Java 中，如果我们想直接从一个对象打印一个值，我们可以尝试的第一件事就是调用它的`toString`方法:

```
Object[] arrayOfObjects = { "John", 2, true };
assertTrue(arrayOfObjects.toString().startsWith("[Ljava.lang.Object;"));
```

如果我们运行测试，它将是成功的，但是通常，它不是一个非常有用的结果。

我们要做的是打印数组中的值。取而代之，我们有`[Ljava.lang.Object.`类的名字，如`Object.class` :

```
getClass().getName() + '@' + Integer.toHexString(hashCode())
```

当我们直接从对象中获取类名时，我们从 JVM 中获取内部名称及其类型，这就是为什么我们有像`[`和`L`这样的额外字符，它们分别代表数组和类名类型。

## 3.打印有意义的值

为了能够正确地打印结果，我们可以使用 [`java.util`](https://web.archive.org/web/20221208143956/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/package-summary.html) 包中的一些类。

### 3.1.`Arrays`

例如，我们可以使用 [`Arrays`](/web/20221208143956/https://www.baeldung.com/java-util-arrays) 类中的两个方法来处理转换。

对于一维数组，我们可以使用`toString`方法:

```
Object[] arrayOfObjects = { "John", 2, true };
assertEquals(Arrays.toString(arrayOfObjects), "[John, 2, true]");
```

对于更深的数组，我们有`deepToString`方法:

```
Object[] innerArray = { "We", "Are", "Inside" };
Object[] arrayOfObjects = { "John", 2, innerArray };
assertEquals(Arrays.deepToString(arrayOfObjects), "[John, 2, [We, Are, Inside]]");
```

### 3.2.流动

**JDK 8 中一个重要的新特性是[引入了 Java 流](/web/20221208143956/https://www.baeldung.com/java-8-streams-introduction)** ，它包含了处理元素序列的类:

```
Object[] arrayOfObjects = { "John", 2, true };
List<String> listOfString = Stream.of(arrayOfObjects)
  .map(Object::toString)
  .collect(Collectors.toList());
assertEquals(listOfString.toString(), "[John, 2, true]");
```

首先，我们使用助手方法`of.` 创建了一个流，使用`map,`将数组中的所有对象转换为一个字符串，然后使用`collect`将它插入到一个列表中以打印值。

## 4.结论

在本教程中，我们已经看到了如何从数组中打印有意义的信息并避免默认的`[Ljava.lang.Object;.`

我们总能在 GitHub 上找到这篇文章[的源代码。](https://web.archive.org/web/20221208143956/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-arrays-guides)