# 查找没有重复字符的最长子字符串

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-longest-substring-without-repeated-characters>

## 1.概观

在本教程中，比较使用 Java 查找唯一字母的最长子串的方法。例如，“CODINGISAWESOME”中唯一字母的最长子串是“NGISAWE”。

## 2.强力方法

让我们从一个天真的方法开始。首先，**我们可以检查每个子串是否包含唯一的字符:**

```java
String getUniqueCharacterSubstringBruteForce(String input) {
    String output = "";
    for (int start = 0; start < input.length(); start++) {
        Set<Character> visited = new HashSet<>();
        int end = start;
        for (; end < input.length(); end++) {
            char currChar = input.charAt(end);
            if (visited.contains(currChar)) {
                break;
            } else {
                visited.add(currChar);
            }
        }
        if (output.length() < end - start + 1) {
            output = input.substring(start, end);
        }
    }
    return output;
}
```

由于有`n*(n+1)/2`个可能的子串，**这种方法的时间复杂度是`O(n^2)`。**

## 3.优化方法

现在，让我们来看看一种优化的方法。我们开始从左到右遍历字符串，并跟踪:

1.  在`start`和`end`索引的帮助下，当前子字符串包含不重复的字符
2.  最长的不重复子串`output`
3.  **一个已经有`visited`个字符的查找表**

```java
String getUniqueCharacterSubstring(String input) {
    Map<Character, Integer> visited = new HashMap<>();
    String output = "";
    for (int start = 0, end = 0; end < input.length(); end++) {
        char currChar = input.charAt(end);
        if (visited.containsKey(currChar)) {
            start = Math.max(visited.get(currChar)+1, start);
        }
        if (output.length() < end - start + 1) {
            output = input.substring(start, end + 1);
        }
        visited.put(currChar, end);
    }
    return output;
}
```

对于每一个新角色，我们都在已经访问过的角色中寻找。如果该字符已经被访问过，并且是包含不重复字符的当前子串的一部分，我们就更新起始索引。否则，我们将继续遍历字符串。

由于我们只遍历字符串一次，**时间复杂度将是线性的，或`O(n)`。**

**这种方法也被称为[滑动窗口模式](/web/20221129015522/https://www.baeldung.com/cs/sliding-window-algorithm)。**

## 4.测试

最后，让我们彻底测试一下我们的实现，以确保它能够工作:

```java
@Test
void givenString_whenGetUniqueCharacterSubstringCalled_thenResultFoundAsExpected() {
    assertEquals("", getUniqueCharacterSubstring(""));
    assertEquals("A", getUniqueCharacterSubstring("A"));
    assertEquals("ABCDEF", getUniqueCharacterSubstring("AABCDEF"));
    assertEquals("ABCDEF", getUniqueCharacterSubstring("ABCDEFF"));
    assertEquals("NGISAWE", getUniqueCharacterSubstring("CODINGISAWESOME"));
    assertEquals("be coding", getUniqueCharacterSubstring("always be coding"));
}
```

在这里，我们尝试并**测试边界条件以及更典型的用例**。

## 5.结论

在本教程中，我们学习了如何使用滑动窗口技术来查找包含不重复字符的最长子串。

和往常一样，源代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20221129015522/https://github.com/eugenp/tutorials/tree/master/algorithms-modules/algorithms-miscellaneous-4)