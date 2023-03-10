# 使用 indexOf 查找字符串中某个单词的所有匹配项

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-indexof-find-string-occurrences>

## 1.概观

在较大的文本字符串中搜索字符模式或单词的工作在不同的领域中完成。例如，在生物信息学中，我们可能需要找到染色体中的 DNA 片段。

在媒体中，编辑在浩如烟海的文本中找到一个特定的短语。数据监控通过查找数据中嵌入的可疑词语来检测诈骗或垃圾邮件。

在任何情况下，搜索是如此众所周知和令人生畏的苦差事，以至于它被普遍称为**“大海捞针的问题”**。在本教程中，我们将演示一个简单的算法，它使用 Java `String`类的`[indexOf(String str, int fromIndex)](https://web.archive.org/web/20220630011440/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html#indexOf(java.lang.String,int)) `方法来查找一个单词在一个字符串中的所有出现。

## 2.简单算法

我们的算法不是简单地计算一个单词在一个大文本中的出现次数，而是找到并识别出一个特定单词在文本中存在的每个位置。我们解决问题的方法简单明了，因此:

1.  搜索**将在文本**的单词中找到该单词。因此，如果我们在搜索“能够”这个词，我们会在“舒适”和“平板电脑”中找到它。
2.  搜索**将不区分大小写**。
3.  算法**基于简单的字符串搜索方法**。这意味着，由于我们对单词和文本字符串中的字符的性质并不了解，我们将使用蛮力来检查文本的每个位置，以找到搜索词的一个实例。

### 2.1.履行

现在我们已经定义了搜索的参数，让我们编写一个简单的解决方案:

```java
public class WordIndexer {

    public List<Integer> findWord(String textString, String word) {
        List<Integer> indexes = new ArrayList<Integer>();
        String lowerCaseTextString = textString.toLowerCase();
        String lowerCaseWord = word.toLowerCase();

        int index = 0;
        while(index != -1){
            index = lowerCaseTextString.indexOf(lowerCaseWord, index);
            if (index != -1) {
                indexes.add(index);
                index++;
            }
        }
        return indexes;
    }
}
```

### 2.2。测试解决方案

为了测试我们的算法，我们将使用莎士比亚的《哈姆雷特》中的一段著名的话，并搜索出现了五次的单词“or ”:

```java
@Test
public void givenWord_whenSearching_thenFindAllIndexedLocations() {
    String theString;
    WordIndexer wordIndexer = new WordIndexer();

    theString = "To be, or not to be: that is the question: "
      + "Whether 'tis nobler in the mind to suffer "
      + "The slings and arrows of outrageous fortune, "
      + "Or to take arms against a sea of troubles, "
      + "And by opposing end them? To die: to sleep; "
      + "No more; and by a sleep to say we end "
      + "The heart-ache and the thousand natural shocks "
      + "That flesh is heir to, 'tis a consummation "
      + "Devoutly to be wish'd. To die, to sleep; "
      + "To sleep: perchance to dream: ay, there's the rub: "
      + "For in that sleep of death what dreams may come,";

    List<Integer> expectedResult = Arrays.asList(7, 122, 130, 221, 438);
    List<Integer> actualResult = wordIndexer.findWord(theString, "or");
    assertEquals(expectedResult, actualResult);
}
```

当我们运行测试时，我们得到了预期的结果。**搜索“或”会产生以不同方式嵌入文本字符串的五个实例:**

```java
index of 7, in "or"
index of 122, in "fortune"
index of 130, in "Or
index of 221, in "more"
index of 438, in "For"
```

在数学术语中，该算法有一个大 O 符号`O(m*(n-m))`，其中`m`是单词的长度，`n`是文本字符串的长度。这种方法可能适用于几千个字符的干草堆文本字符串，但如果有几十亿个字符，速度就会慢得令人无法忍受。

## 3.改进算法

上面这个简单的例子演示了一种在文本字符串中搜索给定单词的简单、粗暴的方法。因此，它将适用于任何搜索词和任何文本。

如果我们事先知道搜索词不包含重复的字符模式，例如“aaa”，那么我们可以编写一个稍微更高效的算法。

在这种情况下，我们可以安全地避免进行备份来重新检查文本字符串中的每个位置，以作为潜在的开始位置。在我们调用了`indexOf( )`方法之后，我们将简单地滑动到最近一次出现的位置。这个简单的调整产生了最好的场景`O(n)`。

让我们来看看早期的`findWord( )`方法的增强版本。

```java
public List<Integer> findWordUpgrade(String textString, String word) {
    List<Integer> indexes = new ArrayList<Integer>();
    StringBuilder output = new StringBuilder();
    String lowerCaseTextString = textString.toLowerCase();
    String lowerCaseWord = word.toLowerCase();
    int wordLength = 0;

    int index = 0;
    while(index != -1){
        index = lowerCaseTextString.indexOf(lowerCaseWord, index + wordLength);  // Slight improvement
        if (index != -1) {
            indexes.add(index);
        }
        wordLength = word.length();
    }
    return indexes;
}
```

## 4.结论

在本教程中，我们介绍了一种不区分大小写的搜索算法，用于在较大的文本字符串中查找单词的所有变体。但是不要让这掩盖了一个事实，即 Java `String` class' `indexOf()`方法天生是区分大小写的，例如可以区分“bob”和“Bob”。

总之，`indexOf()`是一种查找隐藏在文本字符串中的字符序列的便捷方法，无需对子串操作进行任何编码。

像往常一样，这个例子的完整代码库在 GitHub 上[。](https://web.archive.org/web/20220630011440/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-algorithms)