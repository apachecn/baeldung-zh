# Apache Commons 文本介绍

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-apache-commons-text>

## 1。概述

简而言之，Apache Commons 文本库包含许多有用的实用方法来使用`Strings`，超出了核心 Java 提供的范围。

在这个快速介绍中，我们将看到什么是 Apache Commons 文本，它的用途是什么，以及一些使用该库的实际例子。

## 2。Maven 依赖关系

让我们从向我们的`pom.xml`添加以下 Maven 依赖项开始:

```
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-text</artifactId>
    <version>1.1</version>
</dependency>
```

您可以在 [Maven Central Repository](https://web.archive.org/web/20221018034552/https://mvnrepository.com/artifact/org.apache.commons/commons-text) 找到该库的最新版本。

## 3。概述

根包`org.apache.commons.text`被分成不同的子包:

*   `**org.apache.commons.text.diff**`–区别于`Strings`
*   `**org.apache.commons.text.similarity**`—`Strings`之间的相似性和距离
*   `**org.apache.commons.text.translate**`–翻译文本

让我们更详细地看看每个包的用途。

## 3。处理文本

`org.apache.commons.text`包包含了多个使用`Strings.`的工具

例如，`WordUtils`的 API 能够大写`String,`中每个单词的第一个字母，交换`String,`的大小写，并检查`String`是否包含给定数组中的所有单词。

让我们看看如何将`String:`中每个单词的第一个字母大写

```
@Test
public void whenCapitalized_thenCorrect() {
    String toBeCapitalized = "to be capitalized!";
    String result = WordUtils.capitalize(toBeCapitalized);

    assertEquals("To Be Capitalized!", result);
}
```

下面是我们如何检查一个字符串是否包含数组中的所有单词:

```
@Test
public void whenContainsWords_thenCorrect() {
    boolean containsWords = WordUtils
      .containsAllWords("String to search", "to", "search");

    assertTrue(containsWords);
}
```

`StrSubstitutor`提供了一种从模板构建`Strings`的便捷方式:

```
@Test
public void whenSubstituted_thenCorrect() {
    Map<String, String> substitutes = new HashMap<>();
    substitutes.put("name", "John");
    substitutes.put("college", "University of Stanford");
    String templateString = "My name is ${name} and I am a student at the ${college}.";
    StrSubstitutor sub = new StrSubstitutor(substitutes);
    String result = sub.replace(templateString);

    assertEquals("My name is John and I am a student at the University of Stanford.", result);
}
```

`StrBuilder`是 `Java.lang.StringBuilder`的替代。它提供了一些`StringBuilder`没有提供的新功能。

例如，我们可以替换另一个`String`中`String`的所有出现，或者清除一个`String`，而不用给它的引用分配一个新的对象。

这里有一个快速替换部分`String:`的例子

```
@Test
public void whenReplaced_thenCorrect() {
    StrBuilder strBuilder = new StrBuilder("example StrBuilder!");
    strBuilder.replaceAll("example", "new");

    assertEquals(new StrBuilder("new StrBuilder!"), strBuilder);
}
```

要清除一个`String,`，我们可以简单地通过调用构建器上的`clear()`方法来完成:

```
strBuilder.clear();
```

## 4。计算`Strings`和之间的差值

包`org.apache.commons.text.diff`实现了 Myers 算法来计算两个`Strings.`之间的差异

**两个`Strings`之间的差异由一系列修改定义，这些修改可以将一个`String`转换成另一个`String`。**

有三种类型的命令可用于将一个`String`转换成另一个——`InsertCommand`、 `KeepCommand`和 `DeleteCommand`。

一个`EditScript` 对象持有将一个`String`转换成另一个应该运行的脚本。让我们计算一下为了将一个`String`转换成另一个【】应该进行的单字符修改的数量:

```
@Test
public void whenEditScript_thenCorrect() {
    StringsComparator cmp = new StringsComparator("ABCFGH", "BCDEFG");
    EditScript<Character> script = cmp.getScript();
    int mod = script.getModifications();

    assertEquals(4, mod);
}
```

## 5。`Strings`相似与距离

`org.apache.commons.text.similarity`包包含用于查找`Strings.`之间的相似性和距离的算法

例如，`LongestCommonSubsequence`可用于查找两个`Strings`中的共同字符数:

```
@Test
public void whenCompare_thenCorrect() {
    LongestCommonSubsequence lcs = new LongestCommonSubsequence();
    int countLcs = lcs.apply("New York", "New Hampshire");

    assertEquals(5, countLcs);
}
```

类似地，`LongestCommonSubsequenceDistance`可以用来找出两个`Strings`中不同字符的个数:

```
@Test
public void whenCalculateDistance_thenCorrect() {
    LongestCommonSubsequenceDistance lcsd = new LongestCommonSubsequenceDistance();
    int countLcsd = lcsd.apply("New York", "New Hampshire");

    assertEquals(11, countLcsd);
}
```

## 6。文本翻译

最初创建`org.apache.text.translate`包是为了允许我们定制`StringEscapeUtils`提供的规则。

这个包有一组类，负责将文本翻译成一些不同的字符编码模型，比如 Unicode 和数字字符引用。我们也可以为翻译创建我们自己定制的例程。

让我们看看如何将一个`String`转换成它的等价 Unicode 文本:

```
@Test
public void whenTranslate_thenCorrect() {
    UnicodeEscaper ue = UnicodeEscaper.above(0);
    String result = ue.translate("ABCD");

    assertEquals("\\u0041\\u0042\\u0043\\u0044", result);
}
```

这里，我们将开始翻译的字符的索引传递给`above()`方法。

使我们能够定义我们自己的查找表，其中每个字符可以有一个相应的值，我们可以将任何文本翻译成其相应的等价物。

## 7。结论

在这个快速教程中，我们看到了 Apache Commons 文本的概述以及它的一些通用特性。

代码样本可以在 GitHub 上找到。