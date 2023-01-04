# Java 中的 CharSequence 与 String

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-char-sequence-string>

## 1。简介

简单来说， `CharSequence`和`String`是 Java 中两个不同的基础概念。

在这篇简短的文章中，我们将看看这些类型之间的区别以及何时使用每种类型。

## 2。`CharSequence`

`CharSequence`是表示字符序列的接口。此接口不强制执行可变性。因此，可变类和不可变类都实现了这个接口。

当然，一个接口不能直接实例化；它需要一个实现来实例化一个变量:

```java
CharSequence charSequence = "baeldung";
```

这里，`charSequence`用一个`String.`实例化其他实现:

```java
CharSequence charSequence = new StringBuffer("baeldung");
CharSequence charSequence = new StringBuilder("baeldung");
```

## 3。`String`

`String`是 Java 中的字符序列。它是一个不可变的类，也是 Java 中最常用的类型之一。这个类实现了`CharSequence`、`Serializable`和`Comparable<String>`接口。

下面两个实例化用相同的内容创建`Strings`。然而，它们彼此并不平等:

```java
@Test
public void givenUsingString_whenInstantiatingString_thenWrong() {
    CharSequence firstString = "baeldung";
    String secondString = "baeldung";

    assertNotEquals(firstString, secondString);
}
```

## 4。`CharSequence`对`String`对

我们来比较一下`CharSequence`和`String`的区别和共性。它们都位于同一个名为`java.lang.`的包中，但前者是一个接口，后者是一个具体的类。而且，`String`类是不可变的。

在下面的示例中，每个 sum 操作都会创建另一个实例，增加存储的数据量，并返回最近创建的`String:`

```java
@Test
public void givenString_whenAppended_thenUnmodified() {
    String test = "a";
    int firstAddressOfTest = System.identityHashCode(test);
    test += "b";
    int secondAddressOfTest = System.identityHashCode(test);

    assertNotEquals(firstAddressOfTest, secondAddressOfTest);
}
```

另一方面，`StringBuilder`更新已经创建的`String`以保存新值:

```java
@Test
public void givenStringBuilder_whenAppended_thenModified() {
    StringBuilder test = new StringBuilder();
    test.append("a");
    int firstAddressOfTest = System.identityHashCode(test);
    test.append("b");
    int secondAddressOfTest = System.identityHashCode(test);        

    assertEquals(firstAddressOfTest, secondAddressOfTest);
}
```

另一个区别是该接口并不意味着内置的比较策略，而`String`类实现了`Comparable<String>`接口。

为了比较两个`CharSequence`,我们可以将它们转换为`String`,然后再进行比较:

```java
@Test
public void givenIdenticalCharSequences_whenCastToString_thenEqual() {
    CharSequence charSeq1 = "baeldung_1";
    CharSequence charSeq2 = "baeldung_2";

    assertTrue(charSeq1.toString().compareTo(charSeq2.toString()) > 0);
}
```

## 5。结论

我们通常在不确定字符序列使用什么的地方使用`String`。但是，在某些情况下，`StringBuilder`和`StringBuffer`可能更合适。

你可以在 JavaDocs 中找到更多关于 [`CharSequence`](https://web.archive.org/web/20220626120013/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/CharSequence.html) 和`[String](https://web.archive.org/web/20220626120013/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html).`的信息

和往常一样，所有这些例子和代码片段的实现都可以在 Github 上找到[。](https://web.archive.org/web/20220626120013/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-apis)