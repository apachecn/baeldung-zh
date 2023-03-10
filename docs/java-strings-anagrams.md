# 检查两个字符串是否是 Java 中的变位词

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-strings-anagrams>

## 1.概观

根据[维基百科](https://web.archive.org/web/20220831065716/https://en.wikipedia.org/wiki/Anagram)的说法，变位词是通过重新排列不同单词或短语的字母而形成的单词或短语。

我们可以在字符串处理中概括这一点，说**一个字符串的变位词是另一个字符串，其中每个字符的数量完全相同，顺序任意**。

在本教程中，我们将研究如何检测整个字符串变位词，其中每个字符的数量必须相等，包括非字母字符，如空格和数字。例如，`“!low-salt!”` 和 `“owls-lat!!”` 将被视为变位词，因为它们包含完全相同的字符。

## 2.解决办法

让我们比较几个可以判断两个字符串是否是字谜的解决方案。每个解决方案将在开始时检查两个字符串是否具有相同的字符数。这是提前退出的快速方法，因为不同长度的**输入不能是字谜**。

对于每个可能的解决方案，让我们看看作为开发人员的实现复杂性。我们还将使用[大 O 符号](/web/20220831065716/https://www.baeldung.com/java-algorithm-complexity)来查看 CPU 的时间复杂度。

## 3.通过排序检查

我们可以通过排序每个字符串的字符来重新排列它们的字符，这将产生两个规范化的字符数组。

如果两个字符串是变位词，它们的规范化形式应该是相同的。

在 Java 中，我们可以先将这两个字符串转换成`char[]`数组。然后我们可以对这两个数组进行排序，并检查它们是否相等:

```java
boolean isAnagramSort(String string1, String string2) {
    if (string1.length() != string2.length()) {
        return false;
    }
    char[] a1 = string1.toCharArray();
    char[] a2 = string2.toCharArray();
    Arrays.sort(a1);
    Arrays.sort(a2);
    return Arrays.equals(a1, a2);
} 
```

这个解决方案易于理解和实现。然而，这个算法的总运行时间是`O(n log n)`，因为对一个由 *n* 个字符组成的数组进行排序需要花费`O(n log n)`时间。

为了使算法运行，它必须使用一点额外的内存，将两个输入字符串作为字符数组进行复制。

## 4.计数检查

另一种策略是统计每个字符在输入中出现的次数。如果这些直方图在输入之间相等，则字符串是变位词。

为了节省一点内存，我们只构建一个直方图。我们将增加第一个字符串中每个字符的计数，并减少第二个字符串中每个字符的计数。如果这两个字符串是变位词，那么结果将是一切平衡为 0。

直方图需要一个固定大小的计数表，其大小由字符集大小定义。例如，如果我们只使用一个字节来存储每个字符，那么我们可以使用 256 的计数数组来计算每个字符的出现次数:

```java
private static int CHARACTER_RANGE= 256;

public boolean isAnagramCounting(String string1, String string2) {
    if (string1.length() != string2.length()) {
        return false;
    }
    int count[] = new int[CHARACTER_RANGE];
    for (int i = 0; i < string1.length(); i++) {
        count[string1.charAt(i)]++;
        count[string2.charAt(i)]--;
    }
    for (int i = 0; i < CHARACTER_RANGE; i++) {
        if (count[i] != 0) {
            return false;
        }
    }
    return true;
}
```

这种解决方案以`O(n)`的时间复杂度更快。然而，它需要额外的空间用于计数阵列。256 个整数，对于 ASCII 来说还不算太糟。

然而，如果我们需要增加`CHARACTER_RANGE`来支持多字节字符集，比如 UTF 8，这将变得非常消耗内存。因此，只有当可能的字符数在很小的范围内时，它才真正实用。

从开发的角度来看，这个解决方案包含了更多需要维护的代码，并且较少使用 Java 库函数。

## 5.用`MultiSet`检查

我们可以通过使用`[MultiSet](/web/20220831065716/https://www.baeldung.com/guava-multiset)`来简化计数和比较过程。`MultiSet`是一个支持具有重复元素的顺序独立等式的集合。例如，多重集{a，a，b}和{a，b，a}相等。

要使用`Multiset`，我们首先需要将[番石榴](https://web.archive.org/web/20220831065716/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.google.guava%22%20a%3A%22guava%22)依赖项添加到我们的项目`pom.xml`文件中:

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency> 
```

我们将把每个输入字符串转换成一个字符的`MultiSet`。然后我们将检查它们是否相等:

```java
boolean isAnagramMultiset(String string1, String string2) {
    if (string1.length() != string2.length()) {
        return false;
    }
    Multiset<Character> multiset1 = HashMultiset.create();
    Multiset<Character> multiset2 = HashMultiset.create();
    for (int i = 0; i < string1.length(); i++) {
        multiset1.add(string1.charAt(i));
        multiset2.add(string2.charAt(i));
    }
    return multiset1.equals(multiset2);
} 
```

该算法在`O(n)`时间内解决问题，无需声明一个大的计数数组。

类似于之前的计数解决方案。然而，我们没有使用固定大小的表来计数，而是利用`MutlitSet`类来模拟一个可变大小的表，对每个字符进行计数。

这个解决方案的代码比我们的计数解决方案更多地利用了高级库功能。

## 6.基于字母的变位词

到目前为止，这些例子并没有严格遵循变位词的语言学定义。这是因为他们认为标点符号是变位词的一部分，而且是区分大小写的。

让我们调整算法来实现基于字母的变位词。我们只考虑不区分大小写的字母的重排，不考虑空格、标点等其他字符。例如，`“A decimal point”` 和 `“I’m a dot in place.”` 是彼此的变位词。

要解决这个问题，我们可以先对两个输入字符串进行预处理，过滤掉不需要的字符，将字母转换成小写字母。然后我们可以使用上面的解决方案之一(比如说,`MultiSet `解决方案)来检查处理过的字符串上的变位词:

```java
String preprocess(String source) {
    return source.replaceAll("[^a-zA-Z]", "").toLowerCase();
}

boolean isLetterBasedAnagramMultiset(String string1, String string2) {
    return isAnagramMultiset(preprocess(string1), preprocess(string2));
}
```

这种方法可以作为解决所有变位问题的通用方法。例如，如果我们还想包括数字，我们只需要调整预处理过滤器。

## 7.结论

在本文中，我们研究了三种算法，用于逐个字符地检查给定字符串是否是另一个字符串的变位词。对于每个解决方案，我们讨论了速度、可读性和所需内存大小之间的权衡。

我们还研究了如何调整算法来检查更传统的语言学意义上的字谜。我们通过将输入预处理成小写字母来实现这一点。

和往常一样，这篇文章的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220831065716/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-algorithms-3)