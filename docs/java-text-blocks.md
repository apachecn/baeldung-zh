# Java 文本块

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-text-blocks>

## 1.介绍

在之前的教程中，我们看到了如何在任何 Java 版本中使用[多行字符串](/web/20220916193820/https://www.baeldung.com/java-multiline-string)。

在本教程中，我们将详细了解**如何使用 Java 15 文本块特性**最有效地声明多行字符串。

## 2.使用

从 Java 15 开始，文本块成为一个标准特性。对于 Java 13 和 14，我们需要将其作为[预览特性](https://web.archive.org/web/20220916193820/https://openjdk.java.net/jeps/355)来启用。

**文本块以`“””`(三个双引号)开头，后面跟着可选的空格和换行符。**最简单的例子看起来是这样的:

```java
String example = """
     Example text""";
```

注意，文本块的结果类型仍然是`String` `.`文本块只是为我们提供了另一种在源代码中编写`String`文字的方式。

在文本块中，我们可以自由地使用换行符和引号，而不需要转义换行符。它允许我们以更优雅和可读的方式包含 HTML、JSON、SQL 或任何我们需要的文字片段。

**在结果`String,`中，不包括(base)缩进和第一个换行符。**我们将在下一节看看缩进的处理。

## 3.刻痕

幸运的是，当使用文本块时，我们仍然可以适当地缩进代码。为此，缩进的一部分被视为源代码，而缩进的另一部分被视为文本块的一部分。为了做到这一点，编译器检查所有非空行的最小缩进。接下来，编译器将整个文本块左移。

考虑一个包含 HTML 的文本块:

```java
public String getBlockOfHtml() {
    return """
            <html>

                <body>
                    <span>example text</span>
                </body>
            </html>""";
}
```

在这种情况下，最小缩进是 12 个空格。因此，`<html>`左侧和所有后续行的所有 12 个空格都被删除。让我们来测试一下:

```java
@Test
void givenAnOldStyleMultilineString_whenComparing_thenEqualsTextBlock() {
    String expected = "<html>\n"
      + "\n" 
      + "    <body>\n"
      + "        <span>example text</span>\n"
      + "    </body>\n"
      + "</html>";
    assertThat(subject.getBlockOfHtml()).isEqualTo(expected);
}

@Test
void givenAnOldStyleString_whenComparing_thenEqualsTextBlock() {
    String expected = "<html>\n\n    <body>\n        <span>example text</span>\n    </body>\n</html>";
    assertThat(subject.getBlockOfHtml())
       .isEqualTo(expected);
}
```

**当我们需要显式缩进**时，我们可以对非空行(或最后一行)使用较少的缩进:

```java
public String getNonStandardIndent() {
    return """
                Indent
            """;
}

@Test
void givenAnIndentedString_thenMatchesIndentedOldStyle() {
    assertThat(subject.getNonStandardIndent())
            .isEqualTo("    Indent\n");
}
```

此外，我们还可以在文本块中使用转义，我们将在下一节中看到。

## 4.逃避

### 4.1.转义双引号

在文本块中，双引号不必转义。我们甚至可以通过转义其中一个来在文本块中再次使用三个双引号:

```java
public String getTextWithEscapes() {
    return """
            "fun" with
            whitespace
            and other escapes \"""
            """;
}
```

这是必须对双引号进行转义的唯一情况。在其他情况下，这被认为是一种不好的做法。

### 4.2.转义线终止符

一般来说，文本块中的换行符不必转义。

**但是，请注意，即使源文件有 Windows 行尾(`\r\n`)，文本块也只能以换行符(`\n` )** 结束。如果我们需要回车符(`\r`)出现，我们必须显式地将它们添加到文本块中:

```java
public String getTextWithCarriageReturns() {
return """
separated with\r
carriage returns""";
}

@Test
void givenATextWithCarriageReturns_thenItContainsBoth() {
assertThat(subject.getTextWithCarriageReturns())
.isEqualTo("separated with\r\ncarriage returns");
}
```

有时，我们可能在源代码中有很长的文本行，我们希望以可读的方式格式化。Java 14 预览版增加了一个允许我们这样做的特性。我们可以对换行符进行转义，这样它就会被忽略。

```java
public String getIgnoredNewLines() {
    return """
            This is a long test which looks to \
            have a newline but actually does not""";
}
```

实际上，这个`String`文字正好等于一个正常的非中断`String`:

```java
@Test
void givenAStringWithEscapedNewLines_thenTheResultHasNoNewLines() {
    String expected = "This is a long test which looks to have a newline but actually does not";
    assertThat(subject.getIgnoredNewLines())
            .isEqualTo(expected);
}
```

### 4.3.转义空格

**编译器忽略文本块中的所有尾随空格**。但是，从 Java 14 预览版开始，我们可以使用新的转义序列`\s`对空格进行转义。编译器还会保留这个转义空格前面的所有空格。

让我们仔细看看逃逸空间的影响:

```java
public String getEscapedSpaces() {
    return """
            line 1·······
            line 2·······\s
            """;
}

@Test
void givenAStringWithEscapesSpaces_thenTheResultHasLinesEndingWithSpaces() {
    String expected = "line 1\nline 2        \n";
    assertThat(subject.getEscapedSpaces())
            .isEqualTo(expected);
} 
```

**注**:上例中的空格被替换为“”符号，以使其可见。

编译器将删除第一行的空格。但是，第二行以一个转义空格结束，因此所有空格都被保留。

## 5.格式化

为了帮助变量替换，增加了一个新方法，允许直接在字符串上调用 [`String.format`方法](/web/20220916193820/https://www.baeldung.com/string/format):

```java
public String getFormattedText(String parameter) {
    return """
            Some parameter: %s
            """.formatted(parameter);
}
```

## 6.结论

在这个简短的教程中，我们研究了 Java 文本块特性。它可能不是游戏规则的改变者，但它帮助我们写出更好、更可读的代码，这通常是一件好事。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20220916193820/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-14)