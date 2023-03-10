# 在 Java 中查找字符串中的第一个非重复字符

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-find-the-first-non-repeating-character>

## 1.概观

在本教程中，我们将看看在 Java 中查找字符串中第一个不重复字符的不同方法。

我们还将尝试分析解决方案的运行时复杂性。

## 2.问题陈述

**给定一个字符串作为输入，在字符串**中找到第一个不重复的字符。这里有几个例子:

例 1: `Lullaby`

在这个例子中，L 重复了三次。当我们到达字符`u. `时，出现第一个不重复的字符

例 2: `Baeldung`

本例中的所有字符都是不重复的。根据问题陈述，我们取第一个，b。

例 3: `mahimahi`

本例中没有不重复的字符，所有字符都重复一次。所以这里的输出是`null.`

最后，关于这个问题，还有几点需要记住:

*   输入字符串可以是任意长度，并且**可以混合使用大写和小写字符**
*   我们的解决方案应该关注空的或空的输入
*   **输入字符串可以没有不重复的字符**，或者换句话说，可以有一个所有字符至少重复一次的输入，在这种情况下，输出是`null`

有了这个认识，让我们试着去接近这个问题。

## 3.解决方法

### 3.1.强力解决方案

首先，我们试图找到一个强力的解决方案来找到一个字符串的第一个不重复的字符。我们从字符串的开头开始，一次取一个字符，将该字符与字符串中的其他字符进行比较。如果我们找到一个匹配，这意味着这个字符在字符串的其他地方重复，所以我们继续下一个字符。如果没有匹配的字符，我们已经找到了我们的解决方案，我们用这个字符退出程序。

代码如下所示:

```java
public Character firstNonRepeatingCharBruteForceNaive(String inputString) {
    if (null == inputString || inputString.isEmpty()) {
        return null;
    }
    for (int outer = 0; outer < inputString.length(); outer++) {
        boolean repeat = false;
        for (int inner = 0; inner < inputString.length(); inner++) {
            if (inner != outer && inputString.charAt(outer) == inputString.charAt(inner)) {
                repeat = true;
                break;
            }
        }
        if (!repeat) {
            return inputString.charAt(outer);
        }
    }
    return null;
}
```

**上述解决方案的[时间复杂度](/web/20221127062903/https://www.baeldung.com/java-algorithm-complexity)为 O(n )** ，因为我们有两个嵌套循环。对于我们访问的每个字符，我们访问的是输入字符串的所有字符。

下面给出了相同代码的一个更紧凑的解决方案，它利用了`String`类的 [`lastIndexOf `](/web/20221127062903/https://www.baeldung.com/string/last-index-of) 方法。**当我们找到一个字符，它在字符串中的第一个索引也是最后一个索引，这就建立了这个字符只存在于字符串中的那个索引处，因此成为第一个不重复的字符。**

其时间复杂度也是 O(n)。应该注意的是，`lastIndexOf`方法在另一个 O(n)时间内运行，除了已经在运行的外循环，我们每次获取一个字符，因此这是一个 O(n)的解决方案，类似于前一个。

```java
public Character firstNonRepeatingCharBruteForce(String inputString) {
    if (null == inputString || inputString.isEmpty()) {
        return null;
    }
    for (Character c : inputString.toCharArray()) {
        int indexOfC = inputString.indexOf(c);
        if (indexOfC == inputString.lastIndexOf(c)) {
            return c;
        }
    }
    return null;
}
```

### 3.2.优化解决方案

让我们看看我们是否能做得更好。我们讨论的方法的瓶颈是，我们将每个字符与字符串中出现的每个其他字符进行比较，并且我们继续这样做，除非我们到达字符串的末尾或者我们找到答案。相反，如果我们能记住每个字符出现的次数，我们就不需要每次都进行比较，而是只需要查找字符的频率。为此，我们可以使用一个`Map`，更具体地说，一个 [`HashMap`](/web/20221127062903/https://www.baeldung.com/java-hashmap) 。

该映射将字符存储为键，并在其值中存储其频率。当我们访问每个角色时，我们有两个选择:

1.  如果角色已经出现在地图上，我们将当前位置附加到它的值上
2.  如果这个字符没有出现在目前构建的地图中，这就是一个新的字符，我们增加这个字符被看到的次数

在我们完成整个字符串的计算后，我们有一个映射，它告诉我们字符串中每个字符的计数。我们剩下要做的就是再次迭代字符串，第一个在 map 中值的大小为 1 的字符就是我们的答案。

代码如下所示:

```java
public Character firstNonRepeatingCharWithMap(String inputString) {
    if (null == inputString || inputString.isEmpty()) {
        return null;
    }
    Map<Character, Integer> frequency = new HashMap<>();
    for (int outer = 0; outer < inputString.length(); outer++) {
        char character = inputString.charAt(outer);
        frequency.put(character, frequency.getOrDefault(character, 0) + 1);
    }
    for (Character c : inputString.toCharArray()) {
        if (frequency.get(c) == 1) {
            return c;
        }
    }
    return null;
}
```

上面的解决方案要快得多，**假设在地图上查找是一个常数时间操作，或者 O(1)** 。这意味着获得结果的时间不会随着输入字符串大小的增加而增加。

### 3.3.关于优化解决方案的附加说明

我们应该讨论前面讨论过的优化解决方案的一些注意事项。原问题假设输入可以是任意长度，可以包含任意字符。这使得选择`Map`用于查找目的是有效的。

然而，如果我们可以将输入字符集限制为小写字符/大写字符/英文字母字符等。，在 map 上使用固定大小的数组将是更好的设计选择。

例如，如果输入仅限于小写英文字符，我们可以使用大小为 26 的数组，其中数组中的每个索引指的是一个字母表，值可以表示字符串中字符的频率。最后，字符串中值为 1 的第一个字符就是答案。下面是它的代码:

```java
public Character firstNonRepeatingCharWithArray(String inputString) {
    if (null == inputString || inputString.isEmpty()) {
        return null;
    }
    int[] frequency = new int[26];
    for (int outer = 0; outer < inputString.length(); outer++) {
        char character = inputString.charAt(outer);
        frequency[character - 'a']++;
    }
    for (Character c : inputString.toCharArray()) {
        if (frequency[c - 'a'] == 1) {
            return c;
        }
    }
    return null;
}
```

**注意时间复杂度还是 O(n)，但是我们把空间复杂度提高到了常数空间。**这是因为，无论字符串的长度是多少，我们用来存储频率的辅助空间(数组)的长度都会是常数。

## 4.结论

在本文中，我们讨论了查找字符串中第一个不重复字符的不同方法。

像往常一样，你可以在 GitHub 上找到所有的代码样本[。](https://web.archive.org/web/20221127062903/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-algorithms-3)