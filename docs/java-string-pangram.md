# 检查一个字符串是否是 Java 中的盘符

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-string-pangram>

## 1.概观

在本教程中，我们将学习使用一个简单的 Java 程序来检查一个给定的字符串是否有效。一个 **[盘符](https://web.archive.org/web/20221208143832/https://en.wikipedia.org/wiki/Pangram)是任何一个包含给定字母表中所有字母至少一次的字符串。**

## 2.潘格拉姆

字谜不仅适用于英语，也适用于任何其他具有固定字符集的语言。

例如，一个常见的英语 pangram 是“一只敏捷的棕色狐狸跳过了懒狗”。同样，这些也有其他语言版本。

## 3.使用`for`循环

首先，让我们尝试一个 [`for `循环](/web/20221208143832/https://www.baeldung.com/java-loops) `.` 我们将为字母表中的每个字符填充一个带有**标记的`Boolean `数组。**

当标记数组中的所有值都设置为`true`时，代码返回 *true* :

```java
public static boolean isPangram(String str) {
    if (str == null) {
        return false;
    }
    Boolean[] alphabetMarker = new Boolean[ALPHABET_COUNT];
    Arrays.fill(alphabetMarker, false);
    int alphabetIndex = 0;
    str = str.toUpperCase();
    for (int i = 0; i < str.length(); i++) {
        if ('A' <= str.charAt(i) && str.charAt(i) <= 'Z') {
            alphabetIndex = str.charAt(i) - 'A';
            alphabetMarker[alphabetIndex] = true;
        }
    }
    for (boolean index : alphabetMarker) {
        if (!index) {
            return false;
        }
    }
    return true;
}
```

让我们测试一下我们的实现:

```java
@Test
public void givenValidString_isPanagram_shouldReturnSuccess() {
    String input = "Two driven jocks help fax my big quiz";
    assertTrue(Pangram.isPangram(input));  
}
```

## 4.使用 Java 流

另一种方法是使用 [Java Streams API](/web/20221208143832/https://www.baeldung.com/java-8-streams-introduction) 。我们可以**从给定的输入文本中创建一个过滤后的字符流，并使用流** `.` 创建一个字母表`Map`

如果`Map`的大小等于字母表的大小，代码返回成功。对于英语，预期大小为 26:

```java
public static boolean isPangramWithStreams(String str) {
    if (str == null) {
        return false;
    }
    String strUpper = str.toUpperCase();

    Stream<Character> filteredCharStream = strUpper.chars()
      .filter(item -> ((item >= 'A' && item <= 'Z')))
      .mapToObj(c -> (char) c);

    Map<Character, Boolean> alphabetMap = 
      filteredCharStream.collect(Collectors.toMap(item -> item, k -> Boolean.TRUE, (p1, p2) -> p1));

    return alphabetMap.size() == ALPHABET_COUNT;
}
```

当然，让我们测试一下:

```java
@Test
public void givenValidString_isPangramWithStreams_shouldReturnSuccess() {
    String input = "The quick brown fox jumps over the lazy dog";
    assertTrue(Pangram.isPangramWithStreams(input));
}
```

## 5.为完美的 Pangrams 修改

完美的盘龙和普通的盘龙有点不同。一个完美的字谜由字母表中的每个字母恰好一次组成，而字谜至少有一次。

当`Map`的大小等于字母表的大小并且字母表中每个字符的频率正好为 1 时，代码返回*真*:

```java
public static boolean isPerfectPangram(String str) {
    if (str == null) {
        return false;
    }
    String strUpper = str.toUpperCase();

    Stream<Character> filteredCharStream = strUpper.chars()
        .filter(item -> ((item >= 'A' && item <= 'Z')))
        .mapToObj(c -> (char) c);
    Map<Character, Long> alphabetFrequencyMap = 
      filteredCharStream.collect(Collectors.groupingBy(Function.identity(), Collectors.counting()));

    return alphabetFrequencyMap.size() == ALPHABET_COUNT && 
      alphabetFrequencyMap.values().stream().allMatch(item -> item == 1);
}
```

让我们测试一下:

```java
@Test
public void givenPerfectPangramString_isPerfectPangram_shouldReturnSuccess() {
    String input = "abcdefghijklmNoPqrStuVwxyz";
    assertTrue(Pangram.isPerfectPangram(input));
}
```

一个完美的盘符应该每个字符都出现一次。因此，我们之前的 pangram 应该会失败:

```java
String input = "Two driven jocks help fax my big quiz";
assertFalse(Pangram.isPerfectPangram(input));
```

在上面的代码中，给定的字符串输入有几个重复项，比如它有两个 o。因此输出为`false`。

## 5.结论

在本文中，我们讨论了各种解决方法，以确定给定的字符串是否是有效的 pangram。

我们还讨论了 pangram 的另一种风格，称为 perfect pangram &如何以编程方式识别它。

GitHub 上的[提供了代码示例。](https://web.archive.org/web/20221208143832/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-algorithms)