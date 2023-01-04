# 用 Java 计算字符串中的单词

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-word-counting>

## 1.概观

在本教程中，我们将回顾使用 Java 对给定字符串中的单词进行计数的**种不同方法。**

## 2.使用`StringTokenizer`

**在 Java 中计算字符串中的单词数**的一个简单方法是使用`StringTokenizer` 类:

```java
assertEquals(3, new StringTokenizer("three blind mice").countTokens());
assertEquals(4, new StringTokenizer("see\thow\tthey\trun").countTokens());
```

注意，`StringTokenizer` 自动为我们处理**的空白，比如制表符和回车。**

但是，在某些地方可能会出错，比如连字符:

```java
assertEquals(7, new StringTokenizer("the farmer's wife--she was from Albuquerque").countTokens());
```

在这种情况下，我们希望“妻子”和“她”是不同的词，但是因为它们之间没有空格，所以缺省值让我们失望。

幸运的是，`StringTokenizer `与另一个建造者一起运送。**我们可以在构造函数中传递一个分隔符**来完成上面的工作:

```java
assertEquals(7, new StringTokenizer("the farmer's wife--she was from Albuquerque", " -").countTokens());
```

这在试图计算来自**的字符串中的单词时很方便，比如 CSV 文件:**

```java
assertEquals(10, new StringTokenizer("did,you,ever,see,such,a,sight,in,your,life", ",").countTokens());
```

所以，`StringTokenizer`很简单，它让我们大部分时间都在那里。

让我们看看正则表达式能给我们带来什么。

## 3.正则表达式

为了让我们为这个任务提出一个有意义的正则表达式，我们需要定义我们所认为的单词:**一个单词以一个字母开始，以一个空格字符或一个标点符号**结束。

考虑到这一点，给定一个字符串，我们要做的是在我们遇到空格和标点符号的每一点上拆分该字符串，然后计算得到的单词数。

```java
assertEquals(7, countWordsUsingRegex("the farmer's wife--she was from Albuquerque"));
```

让我们稍微夸张一下，看看 regex 的威力:

```java
assertEquals(9, countWordsUsingRegex("no&one;#should%ever-write-like,this;but:well"));
```

仅仅通过向`StringTokenizer` 传递一个分隔符来解决这个问题是不实际的，因为我们必须定义一个很长的分隔符来尝试列出所有可能的标点符号。

事实证明我们真的不需要做太多，**传递 regex** `[\pP\s&&[^']]+` **到** `split` **方法的** `String` **类就可以搞定**:

```java
public static int countWordsUsingRegex(String arg) {
    if (arg == null) {
        return 0;
    }
    final String[] words = arg.split("[\pP\s&&[^']]+");
    return words.length;
}
```

**正则表达式** `[\pP\s&&[^']]+` **查找任意长度的标点符号或空格，并忽略撇号标点符号。**

要了解更多关于正则表达式的信息，请参考 Baeldung 上的[正则表达式。](/web/20221129215915/https://www.baeldung.com/regular-expressions-java)

## 4.循环和`String ` API

另一种方法是用一个标志来记录遇到的单词。

当遇到新单词时，我们将标志设置为`WORD`并增加单词计数，然后当遇到非单词(标点符号或空格字符)时，返回到`SEPARATOR`。

这种方法给出了与正则表达式相同的结果:

```java
assertEquals(9, countWordsManually("no&one;#should%ever-write-like,this but   well")); 
```

我们必须小心标点符号不是真正的单词分隔符的特殊情况，例如:

```java
assertEquals(6, countWordsManually("the farmer's wife--she was from Albuquerque"));
```

这里我们想要的是把“farmer's”算作一个单词，虽然撇号“'”是一个标点符号。

在正则表达式版本中，我们可以使用正则表达式灵活地定义什么不符合字符的条件。但是现在我们正在编写自己的实现，**我们必须在一个单独的方法**中定义这个排除:

```java
private static boolean isAllowedInWord(char charAt) {
    return charAt == '\'' || Character.isLetter(charAt);
} 
```

所以我们在这里所做的是允许单词中的所有字符和合法的标点符号，在这里是撇号。

我们现在可以在实现中使用这种方法:

```java
public static int countWordsManually(String arg) {
    if (arg == null) {
        return 0;
    }
    int flag = SEPARATOR;
    int count = 0;
    int stringLength = arg.length();
    int characterCounter = 0;

    while (characterCounter < stringLength) {
        if (isAllowedInWord(arg.charAt(characterCounter)) && flag == SEPARATOR) {
            flag = WORD;
            count++;
        } else if (!isAllowedInWord(arg.charAt(characterCounter))) {
            flag = SEPARATOR;
        }
        characterCounter++;
    }
    return count;
}
```

第一个条件是当遇到一个单词时标记该单词，并递增计数器。第二个条件检查字符是否不是字母，并将标志设置为`SEPARATOR`。

## 5.结论

在本教程中，我们已经了解了使用几种方法计算单词的方法。我们可以根据特定的用例来选择。

像往常一样，本教程的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221129215915/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-algorithms-2)