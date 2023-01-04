# 在 Java 中连接空字符串

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-concat-null-string>

## 1.概观

Java 为连接`String` s `. `提供了各种方法和类，然而，如果我们不注意`null`对象，结果`String` 可能包含一些不需要的值。

在本教程中，我们将会看到一些在连接`String`时避免`null` `String`对象的方法

## 2.问题陈述

假设我们想要连接一个`String`数组的元素，其中任何元素都可能是`null`。

我们可以使用+运算符简单地做到这一点:

```java
String[] values = { "Java ", null, "", "is ", "great!" };
String result = "";

for (String value : values) {
    result = result + value;
}
```

这将把所有元素连接成结果`String`，如下所示:

```java
Java nullis great!
```

但是，我们可能不希望在输出中显示或附加这样的“空”值。

类似地，如果我们的应用程序运行在 Java 8 或更高版本上，我们使用 [`String.join()`](https://web.archive.org/web/20221208143814/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html#join(java.lang.CharSequence,java.lang.CharSequence...)) 静态方法得到相同的输出:

```java
String result = String.join("", values);
```

当使用`String.join()` 方法时，我们也不能避免空元素被连接起来。

让我们看看一些避免这些`null`元素被连接起来的方法，并得到我们期望的结果:“Java 太棒了！”。

## 3.使用+运算符

加法(+)运算符被重载以在 Java 中连接`String` s。当使用+运算符连接时，我们可以检查`String `是否是`null`，并用空的(" "`String:`)替换`null` `String`

```java
for (String value : values) {
    result = result + (value == null ? "" : value);
}

assertEquals("Java is great!", result);
```

或者，我们可以将检查`null` `String`的代码提取到一个接受`String`对象并返回非`null` `String`对象的帮助器方法中:

```java
for (String value : values) {
    result = result + getNonNullString(value);
}
```

在这里，`getNonNullString() `方法是我们的辅助方法。它只是检查输入`String`对象的`null`引用。如果输入对象是`null`，则返回一个空值(" "`String`)，否则返回相同的`String`:

```java
return value == null ? "" : value;
```

然而，我们知道，`String`对象在 Java 中是不可变的。这意味着，每次我们使用+运算符连接`String`对象时，都会在内存中创建一个新的`String`。因此，使用+运算符进行连接的代价是[昂贵的](/web/20221208143814/https://www.baeldung.com/java-string-performance)。

此外，我们可以使用这种创建助手方法的方法来检查各种其他连接支持操作中的`null` `String`对象。让我们来看看其中的一些。

## 4.使用`String.concat() M`方法

当我们想要连接`String`对象时， [`Str` `ing.conca` `t` `()`](https://web.archive.org/web/20221208143814/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html#concat(java.lang.String)) 方法是一个不错的选择。

这里，我们可以使用我们的`getNonNullString()`方法来检查一个`null`对象并返回一个空的`String`:

```java
for (String value : values) {
    result = result.concat(getNonNullString(value));
}
```

由`getNonNullString()`方法返回的空的`String` 被连接到结果中，从而忽略了`null`对象。

## 5.使用`StringBuilder` 类

`[StringBuilder](https://web.archive.org/web/20221208143814/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/StringBuilder.html) `提供了一堆有用又方便的`String`搭建方法。其中之一就是 [`append()`](https://web.archive.org/web/20221208143814/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/StringBuilder.html#append(java.lang.String)) 法。

在这里，我们也可以使用同样的`getNonNullString()`方法来避开`null`对象，同时使用`append()`方法:

```java
for (String value : values) {
    result = result.append(getNonNullString(value));
}
```

## 6.使用`StringJoiner` 类(Java 8+)

[`StringJoiner`](/web/20221208143814/https://www.baeldung.com/java-string-joiner) 类提供了`String.join()` 的所有功能，以及一个以给定前缀开始，以给定后缀`.` 结束的选项。我们可以使用它的`add()`方法来连接`String`

和以前一样，我们可以使用我们的帮助器方法`getNonNullString() `来避免`null` `String`值被连接起来:

```java
StringJoiner result = new StringJoiner("");

for (String value : values) {
    result = result.add(getNonNullString(value));
}
```

**`String.join()`和`StringJoiner`的一个区别是，与`String.join()`不同，我们必须遍历集合(一个`rray, List,`等等)。)来联结所有的元素。**

## 7.使用`Streams.filter` (Java 8+)

[`Stream`](/web/20221208143814/https://www.baeldung.com/java-8-streams-introduction) API 提供了大量的顺序和并行聚合操作。一个这样的中间流操作是`filter `，它接受一个`[Predicate](/web/20221208143814/https://www.baeldung.com/java-8-functional-interfaces#Predicates) `作为输入，并基于给定的`Predicate.`将`Stream`转换成另一个`Stream`

因此，我们可以定义一个`Predicate `，它将检查一个`String`的`null`值，并将这个`Predicate `传递给`filter()` 方法。因此，`filter`将从原始的`Stream.`中过滤出那些`null`值

最后，我们可以使用`Collectors.joining()`连接所有非`null` `String`值，最后将结果`Stream`收集到一个`String` 变量中:

```java
result = Stream.of(values).filter(value -> null != value).collect(Collectors.joining("")); 
```

## 8.结论

在本文中，我们举例说明了避免`null` `String`对象串联的各种方法。总会有不止一种正确的方法来满足我们的需求。因此，我们必须确定哪种方法最适合给定的地方。

我们必须记住，串联`String `本身可能是一个昂贵的操作，尤其是在循环中。因此，考虑 Java `String` API 的性能方面总是明智的。

和往常一样，这些例子的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221208143814/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-operations-4)