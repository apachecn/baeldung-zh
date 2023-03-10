# 在 Java 中查找回文的子字符串

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-palindrome-substrings>

## 1.概观

在这个快速教程中，我们将通过不同的方法来寻找给定字符串中所有回文的子字符串。我们还将注意每种方法的时间复杂度。

## 2.强力方法

在这种方法中，我们将简单地迭代输入字符串以找到所有的子字符串。同时，我们将检查子字符串是否是回文:

```java
public Set<String> findAllPalindromesUsingBruteForceApproach(String input) {
    Set<String> palindromes = new HashSet<>();
    for (int i = 0; i < input.length(); i++) {
        for (int j = i + 1; j <= input.length(); j++) {
            if (isPalindrome(input.substring(i, j))) {
                palindromes.add(input.substring(i, j));
            }
        }
    }
    return palindromes;
}
```

在上面的例子中，我们只是比较子串和它的反串，看看它是否是一个回文:

```java
private boolean isPalindrome(String input) {
    StringBuilder plain = new StringBuilder(input);
    StringBuilder reverse = plain.reverse();
    return (reverse.toString()).equals(input);
}
```

当然，我们可以很容易地从其他几种方法中选择。

这种方法的时间复杂度是 O(n^3).虽然这对于小的输入字符串来说是可以接受的，但是如果我们要在大量文本中检查回文，我们就需要一种更有效的方法。

## 3.集中化方法

中心化方法的思想是**将每个字符视为支点，并向两个方向扩展以找到回文**。

只有当左右两边的字符匹配时，我们才会展开，使字符串成为回文。否则，我们继续下一个字符。

让我们来看一个快速演示，其中我们将把每个字符视为一个回文的中心:

```java
public Set<String> findAllPalindromesUsingCenter(String input) {
    Set<String> palindromes = new HashSet<>();
    for (int i = 0; i < input.length(); i++) {
        palindromes.addAll(findPalindromes(input, i, i + 1));
        palindromes.addAll(findPalindromes(input, i, i));
    }
    return palindromes;
}
```

在上面的循环中，我们向两个方向扩展，得到所有以每个位置为中心的回文集合。我们将通过在循环`:`中调用方法`findPalindromes` 两次来找到偶数和奇数长度的回文

```java
private Set<String> findPalindromes(String input, int low, int high) {
    Set<String> result = new HashSet<>();
    while (low >= 0 && high < input.length() && input.charAt(low) == input.charAt(high)) {
        result.add(input.substring(low, high + 1));
        low--;
        high++;
    }
    return result;
}
```

这种方法的时间复杂度是 O(n^2).这是对我们强力方法的改进，但我们可以做得更好，我们将在下一节看到。

## 4.Manacher 算法

[**Manacher 的算法**](/web/20220707143819/https://www.baeldung.com/cs/manachers-algorithm) **找到线性时间内最长的回文子串**。我们将使用这个算法找到所有回文的子串。

在我们深入算法之前，我们将初始化几个变量。

首先，在将结果字符串转换为字符数组之前，我们将在开头和结尾用边界字符保护输入字符串:

```java
String formattedInput = "@" + input + "#";
char inputCharArr[] = formattedInput.toCharArray();
```

然后，我们将使用一个有两行的二维数组`radius`——一行存储奇数长度的回文，另一行存储偶数长度的回文:

```java
int radius[][] = new int[2][input.length() + 1];
```

接下来，我们将遍历输入数组，找到以位置`i `为中心的回文长度，并将该长度存储在 `radius[][]`中:

```java
Set<String> palindromes = new HashSet<>();
int max;
for (int j = 0; j <= 1; j++) {
    radius[j][0] = max = 0;
    int i = 1;
    while (i <= input.length()) {
        palindromes.add(Character.toString(inputCharArr[i]));
        while (inputCharArr[i - max - 1] == inputCharArr[i + j + max])
            max++;
        radius[j][i] = max;
        int k = 1;
        while ((radius[j][i - k] != max - k) && (k < max)) {
            radius[j][i + k] = Math.min(radius[j][i - k], max - k);
            k++;
        }
        max = Math.max(max - k, 0);
        i += k;
    }
}
```

最后，我们将遍历数组`radius[][]` 来计算以每个位置为中心的回文子字符串:

```java
for (int i = 1; i <= input.length(); i++) {
    for (int j = 0; j <= 1; j++) {
        for (max = radius[j][i]; max > 0; max--) {
            palindromes.add(input.substring(i - max - 1, max + j + i - 1));
        }
    }
}
```

该方法的时间复杂度为 O(n)。

## 5.结论

在这篇简短的文章中，我们讨论了寻找回文子串的不同方法的时间复杂度。

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20220707143819/https://github.com/eugenp/tutorials/tree/master/algorithms-modules/algorithms-miscellaneous-4)