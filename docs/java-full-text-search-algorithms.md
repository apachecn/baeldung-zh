# 用 Java 实现大文本的字符串搜索算法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-full-text-search-algorithms>

## 1。简介

在本文中，我们将展示几种在大文本中搜索模式的算法。我们将用提供的代码和简单的数学背景来描述每个算法。

请注意，所提供的算法并不是在更复杂的应用程序中进行全文搜索的最佳方式。为了正确地进行全文搜索，我们可以使用 [Solr](/web/20221101164501/https://www.baeldung.com/full-text-search-with-solr) 或 [ElasticSearch](/web/20221101164501/https://www.baeldung.com/elasticsearch-full-text-search-rest-api) 。

## 2。算法

我们将从一个简单的文本搜索算法开始，这是最直观的算法，有助于发现与该任务相关的其他高级问题。

### 2.1。助手方法

在我们开始之前，让我们定义一下 Rabin Karp 算法中使用的计算素数的简单方法:

```java
public static long getBiggerPrime(int m) {
    BigInteger prime = BigInteger.probablePrime(getNumberOfBits(m) + 1, new Random());
    return prime.longValue();
}
private static int getNumberOfBits(int number) {
    return Integer.SIZE - Integer.numberOfLeadingZeros(number);
} 
```

### 2.2。简单文本搜索

这个算法的名字比其他任何解释都更好地描述了它。这是最自然的解决方案:

```java
public static int simpleTextSearch(char[] pattern, char[] text) {
    int patternSize = pattern.length;
    int textSize = text.length;

    int i = 0;

    while ((i + patternSize) <= textSize) {
        int j = 0;
        while (text[i + j] == pattern[j]) {
            j += 1;
            if (j >= patternSize)
                return i;
        }
        i += 1;
    }
    return -1;
}
```

这个算法的思想很简单:遍历文本，如果模式的第一个字母匹配，检查模式的所有字母是否匹配文本。

如果`m`是模式中的字母数，`n`是文本中的字母数，则该算法的时间复杂度为`O(m(n-m + 1))`。

最坏情况发生在`String`发生多次局部事件的情况下:

```java
Text: baeldunbaeldunbaeldunbaeldun
Pattern: baeldung
```

### 2.3。拉宾卡普算法

如上所述，当模式很长并且有大量重复的模式元素时，简单的文本搜索算法是非常低效的。

[Rabin Karp 算法](/web/20221101164501/https://www.baeldung.com/cs/rabin-karp-algorithm)的思想是使用哈希在文本中寻找模式。在算法开始时，我们需要计算一个模式的散列，这个散列将在算法中使用。这个过程叫做指纹计算，我们可以在这里找到详细的解释[。](https://web.archive.org/web/20221101164501/https://en.wikipedia.org/wiki/Rabin%E2%80%93Karp_algorithm)

预处理步骤的重要之处在于，它的时间复杂度是`O(m)`，遍历文本将花费`O(n)`，这给出了整个算法的时间复杂度`O(m+n)`。

算法的代码:

```java
public static int RabinKarpMethod(char[] pattern, char[] text) {
    int patternSize = pattern.length;
    int textSize = text.length;      

    long prime = getBiggerPrime(patternSize);

    long r = 1;
    for (int i = 0; i < patternSize - 1; i++) {
        r *= 2;
        r = r % prime;
    }

    long[] t = new long[textSize];
    t[0] = 0;

    long pfinger = 0;

    for (int j = 0; j < patternSize; j++) {
        t[0] = (2 * t[0] + text[j]) % prime;
        pfinger = (2 * pfinger + pattern[j]) % prime;
    }

    int i = 0;
    boolean passed = false;

    int diff = textSize - patternSize;
    for (i = 0; i <= diff; i++) {
        if (t[i] == pfinger) {
            passed = true;
            for (int k = 0; k < patternSize; k++) {
                if (text[i + k] != pattern[k]) {
                    passed = false;
                    break;
                }
            }

            if (passed) {
                return i;
            }
        }

        if (i < diff) {
            long value = 2 * (t[i] - r * text[i]) + text[i + patternSize];
            t[i + 1] = ((value % prime) + prime) % prime;
        }
    }
    return -1;

}
```

在最坏的情况下，该算法的时间复杂度为`O(m(n-m+1))`。然而，平均而言，该算法具有 `O(n+m)`的时间复杂度。

此外，这种算法还有蒙特卡罗版本，速度更快，但可能会导致错误匹配(假阳性)。

### 2.4。克努特-莫里斯-普拉特算法

在简单的文本搜索算法中，我们看到如果文本中有许多部分与模式匹配，算法会变得很慢。

[Knuth-Morris-Pratt 算法](/web/20221101164501/https://www.baeldung.com/cs/knuth-morris-pratt)的思想是计算移位表，该表为我们提供了应该在哪里搜索候选模式的信息。

KMP 算法的 Java 实现；

```java
public static int KnuthMorrisPrattSearch(char[] pattern, char[] text) {
    int patternSize = pattern.length;
    int textSize = text.length;

    int i = 0, j = 0;

    int[] shift = KnuthMorrisPrattShift(pattern);

    while ((i + patternSize) <= textSize) {
        while (text[i + j] == pattern[j]) {
            j += 1;
            if (j >= patternSize)
                return i;
        }

        if (j > 0) {
            i += shift[j - 1];
            j = Math.max(j - shift[j - 1], 0);
        } else {
            i++;
            j = 0;
        }
    }
    return -1;
}
```

下面是我们计算移位表的方法:

```java
public static int[] KnuthMorrisPrattShift(char[] pattern) {
    int patternSize = pattern.length;

    int[] shift = new int[patternSize];
    shift[0] = 1;

    int i = 1, j = 0;

    while ((i + j) < patternSize) {
        if (pattern[i + j] == pattern[j]) {
            shift[i + j] = i;
            j++;
        } else {
            if (j == 0)
                shift[i] = i + 1;

            if (j > 0) {
                i = i + shift[j - 1];
                j = Math.max(j - shift[j - 1], 0);
            } else {
                i = i + 1;
                j = 0;
            }
        }
    }
    return shift;
}
```

这个算法的时间复杂度也是`O(m+n)`。

### 2.5。简单的博耶-摩尔算法

博耶和摩尔两位科学家提出了另一个想法。为什么不从右向左而不是从左向右比较模式和文本，同时保持移动方向不变:

```java
public static int BoyerMooreHorspoolSimpleSearch(char[] pattern, char[] text) {
    int patternSize = pattern.length;
    int textSize = text.length;

    int i = 0, j = 0;

    while ((i + patternSize) <= textSize) {
        j = patternSize - 1;
        while (text[i + j] == pattern[j]) {
            j--;
            if (j < 0)
                return i;
        }
        i++;
    }
    return -1;
}
```

正如所料，这将在`O(m * n)`时间内运行。但该算法引入了发生和匹配启发式算法，大大提高了算法的速度。我们可以在这里找到更多。

### 2.6。博耶-摩尔-霍斯普尔算法

Boyer-Moore 算法的启发式实现有许多变体，最简单的一种是 Horspool 变体。

这个版本的算法被称为 Boyer-Moore-Horspool，这个变体解决了负移位的问题(我们可以在 Boyer-Moore 算法的描述中读到关于负移位的问题)。

像 Boyer-Moore 算法一样，最坏情况下的时间复杂度是`O(m * n)`，而平均复杂度是 O(n)。空间使用不取决于模式的大小，而只取决于字母表的大小，即 256，因为这是英语字母表中 ASCII 字符的最大值:

```java
public static int BoyerMooreHorspoolSearch(char[] pattern, char[] text) {

    int shift[] = new int[256];

    for (int k = 0; k < 256; k++) {
        shift[k] = pattern.length;
    }

    for (int k = 0; k < pattern.length - 1; k++){
        shift[pattern[k]] = pattern.length - 1 - k;
    }

    int i = 0, j = 0;

    while ((i + pattern.length) <= text.length) {
        j = pattern.length - 1;

        while (text[i + j] == pattern[j]) {
            j -= 1;
            if (j < 0)
                return i;
        }

        i = i + shift[text[i + pattern.length - 1]];
    }
    return -1;
}
```

## 4。结论

在本文中，我们提出了几种文本搜索算法。由于一些算法需要更强的数学背景，我们试图用简单的方式来表达每个算法的主要思想。

和往常一样，源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221101164501/https://github.com/eugenp/tutorials/tree/master/algorithms-modules/algorithms-searching)