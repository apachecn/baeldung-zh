# 在 Java 中检查一个字符串是否包含多个关键字

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/string-contains-multiple-words>

## 1.**简介**

在这个快速教程中，**我们将了解如何检测字符串**中的多个单词。

## 2。我们的例子

让我们假设我们有字符串:

```java
String inputString = "hello there, Baeldung";
```

我们的任务是发现`inputString `是否包含`“hello”`和`“Baeldung”`单词。

所以，让我们把关键字放入一个数组:

```java
String[] words = {"hello", "Baeldung"};
```

此外，单词的顺序并不重要，匹配应该区分大小写。

## 3。使用`String.contains()`

作为开始，**我们将展示如何使用 [`String.contains()`](/web/20220627091544/https://www.baeldung.com/string/contains) 方法来实现我们的目标**。

让我们遍历关键字数组，检查每个条目在`inputString:`中的出现情况

```java
public static boolean containsWords(String inputString, String[] items) {
    boolean found = true;
    for (String item : items) {
        if (!inputString.contains(item)) {
            found = false;
            break;
        }
    }
    return found;
}
```

如果`inputString`包含给定的`item`，则`contains()`方法将返回`true`。当我们的字符串中没有任何关键字时，我们可以停止前进并立即返回一个`false`。

尽管我们需要编写更多的代码，但这种解决方案对于简单的用例来说是很快的。

## 4。使用`String.indexOf()`

类似于使用`String.contains()`方法的解决方案，**我们可以通过使用 [`String.indexOf()`](/web/20220627091544/https://www.baeldung.com/string/index-of) 方法**来检查关键字的索引。为此，我们需要一个接受`inputString`和关键字列表的方法:

```java
public static boolean containsWordsIndexOf(String inputString, String[] words) {
    boolean found = true;
    for (String word : words) {
        if (inputString.indexOf(word) == -1) {
            found = false;
            break;
        }
    }
    return found;
}
```

`indexOf()`方法返回`inputString`中单词的索引。当文本中没有这个词时，索引将是-1。

## 5。使用正则表达式

现在，让我们用一个[正则表达式](/web/20220627091544/https://www.baeldung.com/regular-expressions-java)来匹配我们的单词。为此，我们将使用`Pattern`类。

首先，让我们定义字符串表达式。因为我们需要匹配两个关键字，所以我们将使用两个前瞻来构建正则表达式规则:

```java
Pattern pattern = Pattern.compile("(?=.*hello)(?=.*Baeldung)");
```

对于一般情况:

```java
StringBuilder regexp = new StringBuilder();
for (String word : words) {
    regexp.append("(?=.*").append(word).append(")");
}
```

之后，我们将使用`matcher()`方法来`find()`事件:

```java
public static boolean containsWordsPatternMatch(String inputString, String[] words) {

    StringBuilder regexp = new StringBuilder();
    for (String word : words) {
        regexp.append("(?=.*").append(word).append(")");
    }

    Pattern pattern = Pattern.compile(regexp.toString());

    return pattern.matcher(inputString).find();
}
```

但是，正则表达式是有性能代价的。如果我们要查找多个单词，这个解决方案的性能可能不是最佳的。

## 6。使用 Java 8 和`List`和

最后，我们可以使用 Java 8 的[流 API](/web/20220627091544/https://www.baeldung.com/java-8-streams-introduction) 。但是首先，让我们对初始数据做一些小的转换:

```java
List<String> inputString = Arrays.asList(inputString.split(" "));
List<String> words = Arrays.asList(words);
```

现在，是时候使用流 API 了:

```java
public static boolean containsWordsJava8(String inputString, String[] words) {
    List<String> inputStringList = Arrays.asList(inputString.split(" "));
    List<String> wordsList = Arrays.asList(words);

    return wordsList.stream().allMatch(inputStringList::contains);
}
```

如果输入字符串包含我们所有的关键字，上面的操作管道将返回`true` 。

或者，**我们可以简单地使用[集合框架](/web/20220627091544/https://www.baeldung.com/java-collections)** 的`containsAll()`方法来获得想要的结果:

```java
public static boolean containsWordsArray(String inputString, String[] words) {
    List<String> inputStringList = Arrays.asList(inputString.split(" "));
    List<String> wordsList = Arrays.asList(words);

    return inputStringList.containsAll(wordsList);
}
```

然而，这种方法只对整个单词有效。因此，只有在文本中用空格分隔的情况下，它才会找到我们的关键字。

## 7。使用`Aho-Corasick`算法

简单来说， **[`Aho-Corasick`算法](https://web.archive.org/web/20220627091544/https://en.wikipedia.org/wiki/Aho%E2%80%93Corasick_algorithm)是针对多关键词**的文本搜索。无论我们搜索多少个关键词或者文本有多长，它都有时间复杂度。

让我们在我们的`pom.xml`中包含 [Aho-Corasick 算法依赖](https://web.archive.org/web/20220627091544/https://search.maven.org/search?q=g:org.ahocorasick%20a:ahocorasick):

```java
<dependency>
    <groupId>org.ahocorasick</groupId>
    <artifactId>ahocorasick</artifactId>
    <version>0.4.0</version>
</dependency>
```

首先，让我们用关键字的*单词*数组构建 trie 管道。为此，我们将使用 [Trie](/web/20220627091544/https://www.baeldung.com/trie-java) 数据结构:

```java
Trie trie = Trie.builder().onlyWholeWords().addKeywords(words).build();
```

之后，让我们用`inputString`文本调用解析器方法，我们希望在其中找到关键字并将结果保存在`emits`集合中:

```java
Collection<Emit> emits = trie.parseText(inputString);
```

最后，如果我们打印我们的结果:

```java
emits.forEach(System.out::println);
```

对于每个关键字，我们将看到该关键字在文本中的开始位置、结束位置和关键字本身:

```java
0:4=hello
13:20=Baeldung
```

最后，让我们看看完整的实现:

```java
public static boolean containsWordsAhoCorasick(String inputString, String[] words) {
    Trie trie = Trie.builder().onlyWholeWords().addKeywords(words).build();

    Collection<Emit> emits = trie.parseText(inputString);
    emits.forEach(System.out::println);

    boolean found = true;
    for(String word : words) {
        boolean contains = Arrays.toString(emits.toArray()).contains(word);
        if (!contains) {
            found = false;
            break;
        }
    }

    return found;
}
```

在本例中，我们只查找整个单词。因此，如果我们不仅想匹配`inputString`，还想匹配`“helloBaeldung”`，我们应该简单地从`Trie`构建器管道中删除`onlyWholeWords()`属性。

此外，请记住，我们还从`emits`集合中删除了重复的元素，因为同一个关键字可能有多个匹配。

## 8。结论

在本文中，我们学习了如何在一个字符串中查找多个关键字。此外，**我们展示了使用核心 JDK 以及使用`the Aho-Corasick`库的例子。**

和往常一样，这篇文章的完整代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220627091544/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-algorithms)