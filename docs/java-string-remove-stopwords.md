# 在 Java 中从字符串中删除停用词

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-string-remove-stopwords>

## 1。概述

在本教程中，我们将讨论在 Java 中从`String`中移除停用词的不同方法。当我们想要从文本中删除不想要的或不允许的单词时，这是一个非常有用的操作，例如用户在网站上添加的评论或评论。

我们将使用一个简单的循环、`Collection.removeAll()`和正则表达式。

最后，我们将使用 [Java 微基准测试工具](/web/20221208143830/https://www.baeldung.com/java-microbenchmark-harness)来比较它们的性能。

## 2。加载停用词

首先，我们将从文本文件中加载停用词。

这里我们有文件`english_stopwords.txt`，它包含了我们认为是停用词的单词列表，比如`I`、`he`、`she`和`the`。

我们将使用 `Files.readAllLines()`将停用词加载到`String` 的`List`中:

```java
@BeforeClass
public static void loadStopwords() throws IOException {
    stopwords = Files.readAllLines(Paths.get("english_stopwords.txt"));
}
```

## 3。手动移除停用字词

对于我们的第一个解决方案，**我们将通过迭代每个单词并检查它是否是停用词来手动移除停用词**:

```java
@Test
public void whenRemoveStopwordsManually_thenSuccess() {
    String original = "The quick brown fox jumps over the lazy dog"; 
    String target = "quick brown fox jumps lazy dog";
    String[] allWords = original.toLowerCase().split(" ");

    StringBuilder builder = new StringBuilder();
    for(String word : allWords) {
        if(!stopwords.contains(word)) {
            builder.append(word);
            builder.append(' ');
        }
    }

    String result = builder.toString().trim();
    assertEquals(result, target);
}
```

## 4。使用`Collection.removeAll()`

接下来，我们可以使用`Collection.removeAll()`一次删除所有停用词，而不是迭代`String`、**中的每个词:**

```java
@Test
public void whenRemoveStopwordsUsingRemoveAll_thenSuccess() {
    ArrayList<String> allWords = 
      Stream.of(original.toLowerCase().split(" "))
            .collect(Collectors.toCollection(ArrayList<String>::new));
    allWords.removeAll(stopwords);

    String result = allWords.stream().collect(Collectors.joining(" "));
    assertEquals(result, target);
}
```

在这个例子中，在将我们的`String`分割成一个单词数组之后，我们将把它转换成一个`ArrayList`以便能够应用`removeAll()`方法。

## 5。使用正则表达式

最后，**我们可以从我们的`stopwords`列表**中创建一个正则表达式，然后用它替换我们的`String`中的停用词:

```java
@Test
public void whenRemoveStopwordsUsingRegex_thenSuccess() {
    String stopwordsRegex = stopwords.stream()
      .collect(Collectors.joining("|", "\\b(", ")\\b\\s?"));

    String result = original.toLowerCase().replaceAll(stopwordsRegex, "");
    assertEquals(result, target);
}
```

结果`stopwordsRegex`的格式为“\\b(he|she|the|…)\\b\\s？”。在这个正则表达式中，“\b”指的是一个词的边界，例如，为了避免替换“heat”中的“he”，而“\s？”指零个或一个空格，在替换停用词后删除多余的空格。

## 6。性能比较

现在，让我们看看哪种方法的性能最好。

首先，**让我们建立我们的基准**。我们将使用一个相当大的文本文件作为名为`shakespeare-hamlet.txt`的`String` 的源文件:

```java
@Setup
public void setup() throws IOException {
    data = new String(Files.readAllBytes(Paths.get("shakespeare-hamlet.txt")));
    data = data.toLowerCase();
    stopwords = Files.readAllLines(Paths.get("english_stopwords.txt"));
    stopwordsRegex = stopwords.stream().collect(Collectors.joining("|", "\\b(", ")\\b\\s?"));
}
```

然后我们将有我们的基准方法，从`removeManually()`开始:

```java
@Benchmark
public String removeManually() {
    String[] allWords = data.split(" ");
    StringBuilder builder = new StringBuilder();
    for(String word : allWords) {
        if(!stopwords.contains(word)) {
            builder.append(word);
            builder.append(' ');
        }
    }
    return builder.toString().trim();
}
```

接下来，我们有了`removeAll()`基准测试:

```java
@Benchmark
public String removeAll() {
    ArrayList<String> allWords = 
      Stream.of(data.split(" "))
            .collect(Collectors.toCollection(ArrayList<String>::new));
    allWords.removeAll(stopwords);
    return allWords.stream().collect(Collectors.joining(" "));
}
```

最后，我们将为`replaceRegex()`添加基准:

```java
@Benchmark
public String replaceRegex() {
    return data.replaceAll(stopwordsRegex, "");
}
```

这是我们基准测试的结果:

```java
Benchmark                           Mode  Cnt   Score    Error  Units
removeAll                           avgt   60   7.782 ±  0.076  ms/op
removeManually                      avgt   60   8.186 ±  0.348  ms/op
replaceRegex                        avgt   60  42.035 ±  1.098  ms/op
```

似乎使用`Collection.removeAll()`的**执行时间最快，而使用正则表达式的**执行时间最慢。

## 7。结论

在这篇简短的文章中，我们学习了在 Java 中从一个`String`中移除停用词的不同方法。我们还对它们进行了基准测试，看哪种方法的性能最好。

GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20221208143830/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-algorithms)