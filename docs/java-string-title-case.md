# 将字符串转换为标题大小写

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-string-title-case>

## 1。简介

在这个简短的教程中，我们将展示如何在 Java 中将`String`转换成标题大小写格式。

我们将展示实现自定义方法的不同方式，还将展示如何使用第三方库来实现。

## 2。核心 Java 解决方案

### 2.1。遍历`String`字符

将`String`转换成标题大小写的一种方法是遍历`String`的所有字符。

为此，当我们找到一个单词分隔符时，我们将下一个字符大写。之后，我们将剩余的字符改为小写，直到我们到达下一个单词分隔符。

让我们使用空格作为单词分隔符，并实现此解决方案:

```java
public static String convertToTitleCaseIteratingChars(String text) {
    if (text == null || text.isEmpty()) {
        return text;
    }

    StringBuilder converted = new StringBuilder();

    boolean convertNext = true;
    for (char ch : text.toCharArray()) {
        if (Character.isSpaceChar(ch)) {
            convertNext = true;
        } else if (convertNext) {
            ch = Character.toTitleCase(ch);
            convertNext = false;
        } else {
            ch = Character.toLowerCase(ch);
        }
        converted.append(ch);
    }

    return converted.toString();
}
```

正如我们所看到的，我们使用方法`Character.toTitleCase`来进行转换，因为**在 Unicode 中检查标题大小写是否等同于`Character`。**

如果我们使用这些输入来测试该方法:

```java
tHis IS a tiTLe
tHis, IS a   tiTLe
```

我们得到以下预期输出:

```java
This Is A Title
This, Is A   Title
```

### 2.2。拆分成单词

另一种方法是将`String`拆分成单词，将每个单词转换成标题大小写，最后使用相同的单词分隔符将所有单词再次连接起来。

让我们看看它的代码，再次使用空格作为单词分隔符，以及有用的`Stream` API:

```java
private static final String WORD_SEPARATOR = " ";

public static String convertToTitleCaseSplitting(String text) {
    if (text == null || text.isEmpty()) {
        return text;
    }

    return Arrays
      .stream(text.split(WORD_SEPARATOR))
      .map(word -> word.isEmpty()
        ? word
        : Character.toTitleCase(word.charAt(0)) + word
          .substring(1)
          .toLowerCase())
      .collect(Collectors.joining(WORD_SEPARATOR));
}
```

使用与之前相同的输入，我们得到完全相同的输出:

```java
This Is A Title
This, Is A   Title
```

## 3。使用 Apache Commons

如果我们不想实现自己的定制方法，我们可以使用 Apache Commons 库。在这篇文章中解释了这个库的设置。

这为**提供了`WordUtils`类，该类具有`capitalizeFully()`方法**，该方法正是我们想要实现的:

```java
public static String convertToTileCaseWordUtilsFull(String text) {
    return WordUtils.capitalizeFully(text);
}
```

正如我们所看到的，这非常容易使用，如果我们使用与之前相同的输入进行测试，我们会得到相同的结果:

```java
This Is A Title
This, Is A   Title
```

此外，`WordUtils`类为**提供了另一个`capitalize() `方法**，它的工作方式与`capitalizeFully(),` 类似，只是`**only **` **改变了每个单词**的第一个字符。这意味着它不会将剩余的字符转换成小写。

让我们看看如何使用它:

```java
public static String convertToTileCaseWordUtils(String text) {
    return WordUtils.capitalize(text);
}
```

现在，如果我们使用与之前相同的输入进行测试，我们会得到这些不同的输出:

```java
THis IS A TiTLe
THis, IS A   TiTLe
```

## 4。使用 ICU4J

我们可以使用的另一个库是 ICU4J，它提供了 Unicode 和全球化支持。

要使用它，我们需要将这个依赖项添加到我们的项目中:

```java
<dependency>
    <groupId>com.ibm.icu</groupId>
    <artifactId>icu4j</artifactId>
    <version>61.1</version>
</dependency>
```

最新版本可在[这里](https://web.archive.org/web/20220628122340/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22icu4j%22%20AND%20g%3A%22com.ibm.icu%22)找到。

这个库的工作方式与`WordUtils`非常相似，但是我们可以指定一个`BreakIterator `来告诉这个方法我们想要如何拆分`String`，以及我们想要将哪些单词转换成标题大小写:

```java
public static String convertToTitleCaseIcu4j(String text) {
    if (text == null || text.isEmpty()) {
        return text;
    }

    return UCharacter.toTitleCase(text, BreakIterator.getTitleInstance());
}
```

正如我们所见，他们有一个特定的`BreakIterator`来处理标题。**如果我们不指定任何`BreakIterator`，它会使用 Unicode** 的默认值，在这种情况下会产生相同的结果。

另外，请注意，这个方法让我们指定要转换的`String` 的`Locale `，以便进行特定于地区的转换。

## 5。结论

在这篇简短的文章中，我们展示了如何在 Java 中将`String`转换成标题大小写格式。我们首先使用了我们的自定义实现，之后，我们展示了如何使用外部库来实现。

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20220628122340/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-conversions)