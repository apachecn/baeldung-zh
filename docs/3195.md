# 在 Java 中将 Camel 大小写和 Title 大小写转换为单词

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-camel-case-title-case-to-words>

## 1.概观

字符串通常包含单词和其他分隔符的混合。有时，这些字符串可以通过改变大小写来分隔单词，而不需要空格。比如 **camel 的大小写是第一个**后面的每个单词都大写，title 的大小写(或者 Pascal 的大小写)是每个单词都大写。

我们可能希望将这些字符串解析回单词，以便对它们进行处理。

在这个简短的教程中，我们将看看如何使用[正则表达式](/web/20220526055359/https://www.baeldung.com/regular-expressions-java)找到混合大小写字符串中的单词，以及如何将它们转换成句子或标题。

## 2.解析大写字符串的用例

处理 camel 大小写字符串的一个常见用例可能是文档中的字段名。假设一个文档有一个字段`“firstName” –`，我们可能希望在屏幕上显示为“名字”或“名字”`.`

类似地，如果我们要通过反射扫描应用程序中的类型或函数，以便使用它们的名称生成报告，我们通常会找到我们希望转换的 camel case 或 title case 标识符。

在解析这些表达式时，我们需要解决的一个额外问题是**单字母单词会导致连续的大写字母**。

为了清楚起见:

*   `thisIsAnExampleOfCamelCase`
*   `ThisIsTitleCase`
*   `thisHasASingleLetterWord`

现在我们知道了需要解析的标识符的种类，让我们使用正则表达式来查找单词。

## 3.使用正则表达式查找单词

### 3.1.定义正则表达式来查找单词

让我们定义一个[正则表达式](/web/20220526055359/https://www.baeldung.com/tag/regex/)来定位仅由小写字母、单个大写字母后跟小写字母或单个大写字母组成的单词:

```
Pattern WORD_FINDER = Pattern.compile("(([A-Z]?[a-z]+)|([A-Z]))");
```

这个表达式为正则表达式引擎提供了两个选项。第一个使用`“[A-Z]?”`表示“一个可选的第一个大写字母”，然后使用`“[a-z]+”`表示“一个或多个小写字母”。在那之后，还有提供`or`逻辑的`“|” `字符，接着是表达`“[A-Z]”`，意思是“单个大写字母”。

现在我们有了正则表达式，让我们解析我们的字符串。

### 3.2.在字符串中查找单词

我们将定义一个方法来使用这个正则表达式:

```
public List<String> findWordsInMixedCase(String text) {
    Matcher matcher = WORD_FINDER.matcher(text);
    List<String> words = new ArrayList<>();
    while (matcher.find()) {
        words.add(matcher.group(0));
    }
    return words;
}
```

这使用由正则表达式的`Pattern`创建的`Matcher`来帮助我们[找到单词](/web/20220526055359/https://www.baeldung.com/java-matcher-find-vs-matches)。**我们在匹配器仍有匹配时对其进行迭代**，将它们添加到我们的列表中。

这应该可以提取任何符合我们的单词定义的内容。我们来测试一下。

### 3.3.测试单词查找器

我们的单词查找器应该能够找到由任何非单词字符分隔的单词，以及由大小写变化分隔的单词。让我们从一个简单的例子开始:

```
assertThat(findWordsInMixedCase("some words"))
  .containsExactly("some", "words");
```

这个测试通过了，表明我们的算法正在工作。接下来，我们将尝试骆驼案例:

```
assertThat(findWordsInMixedCase("thisIsCamelCaseText"))
  .containsExactly("this", "Is", "Camel", "Case", "Text");
```

在这里，我们看到单词是从 camel case 字符串中提取出来的，它们的大小写没有变化。比如`“Is”`原文以大写字母开头，提取时大写。

我们也可以用标题案例来尝试:

```
assertThat(findWordsInMixedCase("ThisIsTitleCaseText"))
  .containsExactly("This", "Is", "Title", "Case", "Text");
```

另外，我们可以检查单个字母的单词是否按照我们的意图提取:

```
assertThat(findWordsInMixedCase("thisHasASingleLetterWord"))
  .containsExactly("this", "Has", "A", "Single", "Letter", "Word");
```

到目前为止，我们已经构建了一个单词提取器，但是这些单词的大写方式对于输出来说可能并不理想。

## 4.将单词列表转换为人类可读的格式

提取出一串单词后，我们大概想用 [`toUpperCase`或者`toLowerCase`](/web/20220526055359/https://www.baeldung.com/java-string-convert-case) 这样的方法来规范化它们。然后我们可以使用 [`String.join`](/web/20220526055359/https://www.baeldung.com/java-strings-concatenation#3stringjoin-java-8) 用分隔符将它们连接成一个单独的字符串。让我们看看用这些实现真实世界用例的几种方法。

### 4.1.转换成句子

**句子以大写字母开始，以句号**–`“.”`结束。我们需要能够创造一个以大写字母开头的单词:

```
private String capitalizeFirst(String word) {
    return word.substring(0, 1).toUpperCase()
      + word.substring(1).toLowerCase();
}
```

然后我们可以循环遍历单词，将第一个单词大写，将其他单词小写:

```
public String sentenceCase(List<String> words) {
    List<String> capitalized = new ArrayList<>();
    for (int i = 0; i < words.size(); i++) {
        String currentWord = words.get(i);
        if (i == 0) {
            capitalized.add(capitalizeFirst(currentWord));
        } else {
            capitalized.add(currentWord.toLowerCase());
        }
    }
    return String.join(" ", capitalized) + ".";
}
```

这里的逻辑是，第一个单词的第一个字符大写，其余的都是小写。我们用一个空格作为分隔符将它们连接起来，并在末尾添加一个句点。

让我们来测试一下:

```
assertThat(sentenceCase(Arrays.asList("these", "Words", "Form", "A", "Sentence")))
  .isEqualTo("These words form a sentence.");
```

### 4.2.转换为标题大小写

[题例](/web/20220526055359/https://www.baeldung.com/java-string-title-case)比一句话的规则稍微复杂一点。**每个单词都必须有一个大写字母，除非它是一个特殊的停用词**，通常不大写。但是，整个标题必须以大写字母开头。

我们可以通过定义停用词来实现这一点:

```
Set<String> STOP_WORDS = Stream.of("a", "an", "the", "and", 
  "but", "for", "at", "by", "to", "or")
  .collect(Collectors.toSet());
```

在这之后，我们可以修改循环中的`if`语句，使任何不是停用词的单词大写，第一个也是:

```
if (i == 0 || 
  !STOP_WORDS.contains(currentWord.toLowerCase())) {
    capitalized.add(capitalizeFirst(currentWord));
 }
```

组合单词的算法是相同的，尽管我们没有在最后加上句号。

让我们来测试一下:

```
assertThat(capitalizeMyTitle(Arrays.asList("title", "words", "capitalize")))
  .isEqualTo("Title Words Capitalize");

assertThat(capitalizeMyTitle(Arrays.asList("a", "stop", "word", "first")))
  .isEqualTo("A Stop Word First");
```

## 5.结论

在这篇短文中，我们看了如何使用正则表达式在`String`中找到单词。我们看到了如何定义这个正则表达式，使用大写作为单词边界来查找不同的单词。

我们还研究了一些简单的算法，用于获取单词列表并将它们转换成句子或标题的正确大写。

和往常一样，示例代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220526055359/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-regex-2)