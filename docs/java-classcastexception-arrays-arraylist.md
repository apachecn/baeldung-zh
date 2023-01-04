# ClassCastException:数组$ArrayList 不能强制转换为 ArrayList

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-classcastexception-arrays-arraylist>

## 1.介绍

`ClassCastException`是 Java 中的一个运行时异常，当我们试图**不正确地将一个类从一种类型强制转换为另一种类型时会出现这个异常。**抛出它是为了表明代码试图将一个对象转换成一个相关的类，但它不是这个类的实例。

要更深入地了解 Java 中的异常，请看这里的。

## 2.ClassCastException 详细信息

首先，我们来看一个简单的例子。考虑下面的代码片段:

```java
String[] strArray = new String[] { "John", "Snow" };
ArrayList<String> strList = (ArrayList<String>) Arrays.asList(strArray);
System.out.println("String list: " + strList);
```

上面的代码使`ClassCastException`将`Arrays.asList(strArray) `的返回值转换为`ArrayList.`

原因是尽管静态方法`Arrays.asList()`返回了一个`List,` **，但是直到运行时我们才知道到底返回了什么实现**。所以在编译时编译器也不知道，并允许强制转换`.`

当代码运行时，检查实际的实现，发现`Arrays.asList`()返回一个`Arrays$List`，从而导致一个`ClassCastException`。

## 3.解决

我们可以简单地将我们的`ArrayList`声明为`List`来避免这个异常:

```java
List<String> strList = Arrays.asList(strArray);
System.out.println("String list: " + strList);
```

然而，通过将我们的引用声明为`List`，我们可以将实现`List` 接口的任何类分配给**，包括方法调用返回的`Arrays$ArrayList`。**

## 4.摘要

在本文中，我们已经看到了对什么是`ClassCastException`以及我们必须采取什么措施来解决这个问题的解释。

完整的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220904175400/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-array-list)