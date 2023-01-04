# Java 12 中的字符串 API 更新

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java12-string-api>

## 1。简介

Java 12 为 [`String`类](/web/20220813064811/https://www.baeldung.com/java-string)添加了一些有用的 API。在本教程中，我们将通过示例探索这些新的 API。

## 2。`indent()`

`indent()`方法根据传递给它的参数调整字符串每行的缩进。

当在字符串上调用`indent()`时，会采取以下动作:

1.  字符串在概念上使用`lines()`分成几行。[T1 是 Java 11](/web/20220813064811/https://www.baeldung.com/java-11-string-api) 中引入的字符串 API。
2.  然后根据传递给它的`int`参数`n`对每一行进行调整，然后以换行符“\n”作为后缀。
    1.  如果`n` >为 0，那么`n`个空格被插入到每一行的开头。
    2.  如果`n` <为 0，那么**到** `n`的空白字符都从每行的开头删除。如果给定行没有包含足够的空白，那么所有前导空白字符都将被删除。
    3.  如果`n` == 0，则该行保持不变。但是，行终止符仍然是规范化的。
3.  然后，结果行被连接并返回。

例如:

```java
@Test
public void whenPositiveArgument_thenReturnIndentedString() {
    String multilineStr = "This is\na multiline\nstring.";
    String outputStr = "   This is\n   a multiline\n   string.\n";

    String postIndent = multilineStr.indent(3);

    assertThat(postIndent, equalTo(outputStr));
}
```

我们也可以传递一个负的`int`来减少字符串的缩进。例如:

```java
@Test
public void whenNegativeArgument_thenReturnReducedIndentedString() {
    String multilineStr = "   This is\n   a multiline\n   string.";
    String outputStr = " This is\n a multiline\n string.\n";

    String postIndent = multilineStr.indent(-2);

    assertThat(postIndent, equalTo(outputStr));
}
```

## 3。`transform()`

我们可以使用`transform()` 方法对`this`字符串应用一个函数。该函数应该期待一个单独的`String`参数并产生一个结果:

```java
@Test
public void whenTransformUsingLamda_thenReturnTransformedString() {
    String result = "hello".transform(input -> input + " world!");

    assertThat(result, equalTo("hello world!"));
}
```

输出不必一定是字符串。例如:

```java
@Test
public void whenTransformUsingParseInt_thenReturnInt() {
    int result = "42".transform(Integer::parseInt);

    assertThat(result, equalTo(42));
}
```

## 4。结论

在本文中，我们探索了 Java 12 中新的`String`API。像往常一样，代码片段可以在 GitHub 上找到。