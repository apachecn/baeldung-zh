# 在 Java 中检索类名

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-class-name>

## 1.概观

在本教程中，我们将学习从`Class` API 上的方法中检索类名的四种方法:`[getSimpleName()](https://web.archive.org/web/20220901133708/https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/lang/Class.html#getSimpleName()), [getName()](https://web.archive.org/web/20220901133708/https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/lang/Class.html#getName()), [getTypeName()](https://web.archive.org/web/20220901133708/https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/lang/Class.html#getTypeName())`和`[getCanonicalName()](https://web.archive.org/web/20220901133708/https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/lang/Class.html#getCanonicalName()).`

这些方法可能会令人混淆，因为它们的名称相似，并且它们的 Javadocs 有些模糊。当涉及到基本类型、对象类型、内部或匿名类以及数组时，它们也有一些细微差别。

## 2.检索简单名称

让我们从`getSimpleName()`方法开始。

在 Java 中，有两种名字:`simple`和`qualified`。**一个简单的名称由一个唯一的标识符组成，而一个合格的名称是由点分隔的一系列简单的名称。**

顾名思义，`getSimpleName()`返回底层类的简单名称，即**它在源代码**中被赋予的名称。

让我们想象下面的类:

```java
package com.baeldung.className;
public class RetrieveClassName {}
```

它的简单名字是`RetrieveClassName`:

```java
assertEquals("RetrieveClassName", RetrieveClassName.class.getSimpleName());
```

我们也可以得到基本类型和数组的简单名称。对于基本类型，这只是它们的名字，比如`int, boolean`或 `float`。

对于数组，该方法将返回数组类型的简单名称**，后跟数组每个维度的一对左右括号([])** :

```java
RetrieveClassName[] names = new RetrieveClassName[];
assertEquals("RetrieveClassName[]", names.getClass().getSimpleName());
```

因此，对于一个二维的`String`数组，在它的类上调用`getSimpleName()`将返回`String[][]`。

最后，还有匿名类的特殊情况。**在匿名类上调用`getSimpleName()`将返回一个空字符串。**

## 3.检索其他姓名

现在是时候看看我们如何获得类名、类型名或规范名了。与`getSimpleName()`不同，这些名字旨在给出更多关于这个类的信息。

`getCanonicalName()`方法总是返回在 [Java 语言规范](https://web.archive.org/web/20220901133708/https://docs.oracle.com/javase/specs/jls/se11/html/jls-6.html#jls-6.7)中定义的规范名称**。**

至于其他方法，根据用例的不同，输出会有一些不同。我们将看到这对于不同的原语和对象类型意味着什么。

### 3.1.原始类型

让我们从基本类型开始，因为它们很简单。**对于原始类型，所有三个方法`getName(), getTypeName()` 和`getCanonicalName()`将返回与`getSimpleName()`** 相同的结果:

```java
assertEquals("int", int.class.getName());
assertEquals("int", int.class.getTypeName());
assertEquals("int", int.class.getCanonicalName());
```

### 3.2.对象类型

我们现在来看看这些方法是如何处理对象类型的。它们的行为通常是相同的:**它们都返回类的规范名称**。

在大多数情况下，这是一个包含所有类包简单名称和类简单名称的限定名称:

```java
assertEquals("com.baeldung.className.RetrieveClassName", RetrieveClassName.class.getName());
assertEquals("com.baeldung.className.RetrieveClassName", RetrieveClassName.class.getTypeName());
assertEquals("com.baeldung.className.RetrieveClassName", RetrieveClassName.class.getCanonicalName());
```

### 3.3.内部类

我们在上一节中看到的是这些方法调用的一般行为，但也有一些例外。

内部类就是其中之一。对于内部类来说，`getName() `和`getTypeName()`方法的行为不同于`getCanonicalName()`方法。

**`getCanonicalName()`仍然返回类**的规范名，即封闭类的规范名加上用点分隔的内部类的简单名。

另一方面，`getName() `和`getTypeName()`方法返回几乎相同，但是**使用美元作为封闭类规范名和内部类简单名**之间的分隔符。

让我们想象一下我们的`RetrieveClassName`的一个内部类`InnerClass`:

```java
public class RetrieveClassName {
    public class InnerClass {}
}
```

然后每个调用以稍微不同的方式表示内部类:

```java
assertEquals("com.baeldung.RetrieveClassName.InnerClass", 
  RetrieveClassName.InnerClass.class.getCanonicalName());
assertEquals("com.baeldung.RetrieveClassName$InnerClass", 
  RetrieveClassName.InnerClass.class.getName());
assertEquals("com.baeldung.RetrieveClassName$InnerClass", 
  RetrieveClassName.InnerClass.class.getTypeName());
```

### 3.4.匿名类

匿名类是另一个例外。

正如我们已经看到的，它们没有简单的名字，但是**它们也没有规范的名字**。因此，`getCanonicalName()`不返回任何东西。**与`getSimpleName()`相反，`getCanonicalName()`在匿名类上被调用时将返回`null`** 而不是空字符串。

至于`getName()`和`getTypeName()`，它们将返回**调用类规范名称，后跟一个美元和一个数字，表示该匿名类在调用类**中创建的所有匿名类中的位置。

让我们用一个例子来说明这一点。我们将在这里创建两个匿名类，第一个调用`getName()`，第二个调用`getTypeName() `，在`com.baeldung.Main`中声明它们:

```java
assertEquals("com.baeldung.Main$1", new RetrieveClassName() {}.getClass().getName());
assertEquals("com.baeldung.Main$2", new RetrieveClassName() {}.getClass().getTypeName());
```

我们应该注意到，第二个调用返回一个末尾增加了数字的名称，因为它应用于第二个匿名类。

### 3.5.数组

最后，让我们看看以上三种方法是如何处理数组的。

为了表明我们正在处理数组，每个方法都会更新它的标准结果。**`getTypeName()`和`getCanonicalName()`方法会将括号对附加到它们的结果中。**

让我们看看下面的例子，我们在二维`InnerClass`数组上调用`getTypeName() `和`getCanonicalName()`:

```java
assertEquals("com.baeldung.RetrieveClassName$InnerClass[][]", 
  RetrieveClassName.InnerClass[][].class.getTypeName());
assertEquals("com.baeldung.RetrieveClassName.InnerClass[][]", 
  RetrieveClassName.InnerClass[][].class.getCanonicalName());
```

请注意第一个调用是如何使用美元而不是点来将内部类部分与名称的其余部分分开的。

现在让我们看看`getName()` 方法是如何工作的。当在一个原始类型数组上被调用时，它将返回**一个左括号和[一个代表原始类型](https://web.archive.org/web/20220901133708/https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/lang/Class.html#getName())** 的字母。让我们看看下面的例子，在一个二维原始整数数组上调用这个方法:

```java
assertEquals("[[I", int[][].class.getName());
```

另一方面，当在一个对象数组上被调用时，它会**给它的标准结果添加一个左括号和 L 字母，并以一个分号**结束。让我们在一组`RetrieveClassName`上试试:

```java
assertEquals("[Lcom.baeldung.className.RetrieveClassName;", RetrieveClassName[].class.getName());
```

## 4.结论

在本文中，我们研究了在 Java 中访问类名的四种方法。这些方法是:`getSimpleName(), getName(), getTypeName()`和`getCanonicalName()`。

我们了解到第一个只是返回一个类的源代码名，而其他的提供了更多的信息，比如包名和这个类是内部类还是匿名类的指示。

这篇文章的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220901133708/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang)