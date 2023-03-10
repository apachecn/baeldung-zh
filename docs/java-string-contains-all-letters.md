# 用 Java 检查一个字符串是否包含字母表中的所有字母

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-string-contains-all-letters>

## 1。概述

在本教程中，我们将了解如何检查一个字符串是否包含字母表中的所有字母。

这里有一个简单的例子:“`Farmer jack realized that big yellow quilts were expensive.`”——它实际上包含了字母表中的所有字母。

我们将讨论三种方法。

首先，我们将使用命令式方法对算法进行建模。然后会使用正则表达式。最后，我们将利用 Java 8 更具声明性的方法。

此外，我们将讨论所采用方法的高度复杂性。

## 2。命令式算法

让我们实现一个命令式算法。为此，首先，我们将创建一个布尔数组。然后，我们将逐个字符地遍历输入字符串，并将字符标记为已访问。

请注意`Uppercase`和`Lowercase`被认为是相同的。所以索引 0 代表 A 和 A，同样，索引 25 代表 Z 和 Z。

最后，我们将检查被访问数组中的所有字符是否都设置为 true:

```java
public class EnglishAlphabetLetters {

    public static boolean checkStringForAllTheLetters(String input) {
        int index = 0;
        boolean[] visited = new boolean[26];

        for (int id = 0; id < input.length(); id++) {
            if ('a' <= input.charAt(id) && input.charAt(id) <= 'z') {
                index = input.charAt(id) - 'a';
            } else if ('A' <= input.charAt(id) && input.charAt(id) <= 'Z') {
                index = input.charAt(id) - 'A';
            }
            visited[index] = true;
        }

        for (int id = 0; id < 26; id++) {
            if (!visited[id]) {
                return false;
            }
        }
        return true;
    }
}
```

这个程序的复杂度是 O(n ),其中`n`是字符串的长度。

请注意，有许多方法可以优化算法，例如从集合中删除字母，并在`Set`为空时立即中断。不过，对于这个练习来说，这个算法已经足够好了。

## 3。使用正则表达式

使用正则表达式，我们可以用几行代码轻松获得相同的结果:

```java
public static boolean checkStringForAllLetterUsingRegex(String input) {
    return input.toLowerCase()
      .replaceAll("[^a-z]", "")
      .replaceAll("(.)(?=.*\\1)", "")
      .length() == 26;
}
```

这里，我们首先从`input`中删除除了字母以外的所有字符。然后我们删除重复的字符。最后，我们正在数字母，确保我们有所有的，26 个。

尽管这种方法的性能较差，但其大复杂度也趋向于 O(n)。

## 4。Java 8 流

使用 Java 8 的特性，我们可以使用 Stream 的`filter `和`distinct`方法以更紧凑和声明性的方式轻松实现相同的结果:

```java
public static boolean checkStringForAllLetterUsingStream(String input) {
    long c = input.toLowerCase().chars()
      .filter(ch -> ch >= 'a' && ch <= 'z')
      .distinct()
      .count();
    return c == 26;
}
```

这种方法的复杂度也是 O(n)。

## 4。测试

让我们为我们的算法测试一条快乐的路径:

```java
@Test
public void givenString_whenContainsAllCharacter_thenTrue() {
    String sentence = "Farmer jack realized that big yellow quilts were expensive";
    assertTrue(EnglishAlphabetLetters.checkStringForAllTheLetters(sentence));
}
```

这里，`sentence`包含了字母表中的所有字母，因此，我们期望得到结果`true`。

## 5。结论

在本教程中，我们已经讲述了如何检查一个字符串是否包含字母表 ***中的所有字母。***

我们看到了实现这一点的几种方法，首先是使用传统的命令式编程、正则表达式和 Java 8 流。

完整的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220524121722/https://github.com/eugenp/tutorials/tree/master/algorithms-modules/algorithms-miscellaneous-4)