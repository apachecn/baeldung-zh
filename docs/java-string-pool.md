# Java 字符串池指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-string-pool>

## 1。概述

对象是 Java 语言中使用最多的类。

在这篇简短的文章中，我们将探索 Java 字符串池—**—JVM**存储`Strings`的特殊内存区域。

## 2。字符串实习

由于 Java 中`Strings`的不变性，JVM 可以通过**在池**中只存储每个文字`String`的一个副本来优化分配给它们的内存量。这个过程被称为`interning`。

当我们创建一个*字符串*变量并给它赋值时，JVM 在池中搜索一个具有相同值的`String`。

如果找到了，Java 编译器将简单地返回对其内存地址的引用，而不分配额外的内存。

如果没有找到，它将被添加到池中(interned ),其引用将被返回。

让我们编写一个小测试来验证这一点:

```java
String constantString1 = "Baeldung";
String constantString2 = "Baeldung";

assertThat(constantString1)
  .isSameAs(constantString2);
```

## 3。`Strings`使用构造函数分配

当我们通过`new`操作符创建一个`String`时，Java 编译器将创建一个新的对象，并将它存储在为 JVM 保留的堆空间中。

每个像这样创建的`String`都将指向一个不同的内存区域，并有自己的地址。

让我们看看这与前一个案例有何不同:

```java
String constantString = "Baeldung";
String newString = new String("Baeldung");

assertThat(constantString).isNotSameAs(newString);
```

## 4。`String`文字 vs `String Object`

**当我们使用`new()`操作符创建一个`String`对象时，它总是在堆内存中创建一个新对象。另一方面，如果我们使用`String`字面语法创建一个对象，例如“Baeldung”，它可能从字符串池返回一个现有的对象，如果它已经存在的话。**否则，它会创建一个新的字符串对象并放入字符串池中以备将来重用。

在高层次上，两者都是`String`对象，但是主要的区别在于`new()`操作符总是创建一个新的`String`对象。此外，当我们使用 literal 创建一个`String`时——它是 interned。

当我们比较使用`String`文字和`new`操作符创建的两个`String`对象时，这将变得更加清楚:

```java
String first = "Baeldung"; 
String second = "Baeldung"; 
System.out.println(first == second); // True
```

在这个例子中，`String`对象将具有相同的引用。

接下来，让我们使用`new`创建两个不同的对象，并检查它们是否有不同的引用:

```java
String third = new String("Baeldung");
String fourth = new String("Baeldung"); 
System.out.println(third == fourth); // False
```

类似地，当我们使用==操作符比较一个`String`文本和一个使用`new()`操作符创建的`String`对象时，它将返回`false:`

```java
String fifth = "Baeldung";
String sixth = new String("Baeldung");
System.out.println(fifth == sixth); // False
```

一般来说，**我们应该尽可能使用`String`的字面符号**。它更容易阅读，并且给编译器一个机会来优化我们的代码。

## 5。手工实习

我们可以在 Java 字符串池中手动填充一个`String`,方法是在我们想要填充的对象上调用`intern()`方法。

手动强制`String`将把它的引用存储在池中，JVM 将在需要时返回这个引用。

让我们为此创建一个测试用例:

```java
String constantString = "interned Baeldung";
String newString = new String("interned Baeldung");

assertThat(constantString).isNotSameAs(newString);

String internedString = newString.intern();

assertThat(constantString)
  .isSameAs(internedString);
```

## 6。垃圾收集

在 Java 7 之前，JVM **将 Java 字符串池放在`PermGen`空间中，它的大小是固定的——在运行时不能扩展，也没有资格进行垃圾收集**。

在`PermGen` (而不是`Heap`)中实习`Strings`的风险在于，如果我们实习了太多的`Strings`，我们可能会从 JVM 得到一个`OutOfMemory`错误。

从 Java 7 开始，Java 字符串池被**存储在`Heap`空间，由 JVM `.` 进行垃圾收集**这种方法的优点是**降低了`OutOfMemory`错误**的风险，因为未引用的`Strings`将从池中移除，从而释放内存。

## 7。性能和优化

在 Java 6 中，我们可以执行的唯一优化是在程序调用期间使用`MaxPermSize` JVM 选项增加`PermGen`空间:

```java
-XX:MaxPermSize=1G
```

在 Java 7 中，我们有更详细的选项来检查和扩展/减小池的大小。让我们看一下查看池大小的两个选项:

```java
-XX:+PrintFlagsFinal
```

```java
-XX:+PrintStringTableStatistics
```

如果我们想根据存储桶增加池的大小，我们可以使用`StringTableSize` JVM 选项:

```java
-XX:StringTableSize=4901
```

在 Java 7u40 之前，默认的池大小是 1009 个存储桶，但是在最新的 Java 版本中，这个值会有一些变化。准确地说，从 Java 7u40 到 Java 11 的默认池大小是 60013，现在增加到 65536。

**注意，增加池大小会消耗更多内存，但有利于减少将`Strings`插入表中所需的时间。**

## 8。关于 Java 9 的一点说明

在 Java 8 之前，`Strings`在内部被表示为字符数组——`char[]`，用`UTF-16`编码，因此每个字符使用两个字节的内存。

Java 9 提供了一种新的表示，称为`Compact Strings.` ，这种新格式将根据存储的内容在`char[]`和`byte[]`之间选择合适的编码。

由于新的`String`表示将仅在必要时使用`UTF-16`编码，因此`heap` 内存的数量将显著减少，这进而导致`JVM.`上的`Garbage Collector`开销减少

## 9。结论

在本指南中，我们展示了 JVM 和 Java 编译器如何通过 Java 字符串池为`String`对象优化内存分配。

本文中使用的所有代码示例都可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220927081517/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-strings)