# Java 多行字符串

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-multiline-string>

## 1。概述

在本教程中，我们将学习如何在 Java 中声明多行字符串。

既然 Java 15 已经推出，我们可以使用新的原生特性，称为文本块。

如果我们不能使用这个特性，我们还将回顾其他方法。

## 2.文本块

我们可以通过用**“”****(三个双引号)**声明字符串来使用`Text Blocks`:

```java
public String textBlocks() {
    return """
        Get busy living
        or
        get busy dying.
        --Stephen King""";
}
```

这是声明多行字符串最方便的方式。事实上，**我们不必处理行分隔符或缩进空间**，正如我们在[的专门文章](/web/20221129214038/https://www.baeldung.com/java-text-blocks)中提到的。

这个特性在 Java 15 中是可用的，但是如果我们[启用预览特性](/web/20221129214038/https://www.baeldung.com/java-preview-features)，那么在 Java 13 和 14 中也是可用的。

在接下来的章节中，我们将回顾如果我们使用以前版本的 Java 或者`Text Blocks`不适用的话，其他合适的方法。

## 3。获取行分隔符

每个操作系统都有自己定义和识别新行的方式。

在 Java 中，很容易得到操作系统的行分隔符:

```java
String newLine = System.getProperty("line.separator");
```

我们将在接下来的章节中使用这个`newLine`来创建多行字符串。

## 4。字符串串联

字符串串联是一种简单的本地方法，可用于创建多行字符串:

```java
public String stringConcatenation() {
    return "Get busy living"
            .concat(newLine)
            .concat("or")
            .concat(newLine)
            .concat("get busy dying.")
            .concat(newLine)
            .concat("--Stephen King");
}
```

使用+运算符是实现相同目的的另一种方法。

Java 编译器以同样的方式翻译`concat()`和+运算符:

```java
public String stringConcatenation() {
    return "Get busy living"
            + newLine
            + "or"
            + newLine
            + "get busy dying."
            + newLine
            + "--Stephen King";
}
```

## 5。字符串连接

Java 8 引入了 [`String#join`](https://web.archive.org/web/20221129214038/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html#join(java.lang.CharSequence,java.lang.CharSequence...)) ，它采用一个分隔符和一些字符串作为参数。

它返回一个最终字符串，其中所有输入字符串都用分隔符连接在一起:

```java
public String stringJoin() {
    return String.join(newLine,
                       "Get busy living",
                       "or",
                       "get busy dying.",
                       "--Stephen King");
}
```

## 6。字符串生成器

`StringBuilder`是构建`String` s. [`StringBuilder`的助手类](/web/20221129214038/https://www.baeldung.com/java-string-builder-string-buffer)是在 Java 1.5 中作为`StringBuffer`的替代而引入的。

这是在循环中构建巨大字符串的好选择:

```java
public String stringBuilder() {
    return new StringBuilder()
            .append("Get busy living")
            .append(newLine)
            .append("or")
            .append(newLine)
            .append("get busy dying.")
            .append(newLine)
            .append("--Stephen King")
            .toString();
}
```

## 7。字符串写入器

`StringWriter`是我们可以用来创建多行字符串的另一种方法。这里我们不需要`newLine`，因为我们使用了`PrintWriter`。

`println`功能自动添加新行:

```java
public String stringWriter() {
    StringWriter stringWriter = new StringWriter();
    PrintWriter printWriter = new PrintWriter(stringWriter);
    printWriter.println("Get busy living");
    printWriter.println("or");
    printWriter.println("get busy dying.");
    printWriter.println("--Stephen King");
    return stringWriter.toString();
}
```

## 8。番石榴木工

仅仅为了这样一个简单的任务而使用一个外部库没有多大意义。但是，如果项目已经将该库用于其他目的，我们可以利用它。

比如谷歌的番石榴库就很受欢迎。

[番石榴有一个`Joiner`类](/web/20221129214038/https://www.baeldung.com/guava-joiner-and-splitter-tutorial)，能够构建多行字符串:

```java
public String guavaJoiner() {
    return Joiner.on(newLine).join(ImmutableList.of("Get busy living",
        "or",
        "get busy dying.",
        "--Stephen King"));
}
```

## 9。从文件加载

Java 完全按照文件的原样读取文件。这意味着，如果我们在一个文本文件中有一个多行字符串，当我们读取该文件时，我们将得到相同的字符串。Java 中有很多方法[从文件中读取。](/web/20221129214038/https://www.baeldung.com/java-read-file)

将长字符串从代码中分离出来实际上是一个很好的做法:

```java
public String loadFromFile() throws IOException {
    return new String(Files.readAllBytes(Paths.get("src/main/resources/stephenking.txt")));
}
```

## 10。使用 IDE 功能

许多现代的 ide 支持多行复制/粘贴。Eclipse 和 IntelliJ IDEA 就是这种 ide 的例子。我们可以简单地复制我们的多行字符串，并将其粘贴在 ide 中的两个双引号内。

显然，这个方法不能在运行时创建字符串，但是这是一个快速简单的获得多行字符串的方法。

## 11。结论

在本文中，我们学习了几种在 Java 中构建多行字符串的方法。

好消息是 Java 15 通过`Text Blocks`提供了对多行字符串的原生支持。

所有其他方法都可以在 Java 15 或任何以前的版本中使用。

本文中所有方法的代码都可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221129214038/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-strings)