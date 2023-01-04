# 检查字符串是否是重复的子字符串

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-repeated-substring>

## 1。简介

在本教程中，我们将展示如何在 Java 中检查一个`String`是否是一系列重复的[子串](/web/20221205205601/https://www.baeldung.com/java-substring)。

## 2.问题是

在我们继续实现之前，让我们设置一些条件。首先，我们假设我们的`String`至少有两个字符。

其次，一个子串至少有一次重复。

通过检查几个重复的子字符串，可以用一些例子很好地说明这一点:

```
"aa"
"ababab"
"barrybarrybarry"
```

还有几个不重复的:

```
"aba"
"cbacbac"
"carlosxcarlosy"
```

我们现在将展示几个解决问题的方法。

## 3.天真的解决方法

让我们实现第一个解决方案。

这个过程相当简单:我们将检查`String`的长度，并从一开始就删除单个字符`String`。

然后，**由于子串的长度不能大于字符串长度的一半，**我们将遍历`String`的一半，并在每次迭代中通过将下一个字符追加到前一个子串来创建子串。

接下来，我们将从原始的`String`中移除这些子字符串，并检查被“剥离”的子字符串的长度是否为零。这意味着它只由它的子串组成:

```
public static boolean containsOnlySubstrings(String string) {

    if (string.length() < 2) {
        return false;
    }

    StringBuilder substr = new StringBuilder();
    for (int i = 0; i < string.length() / 2; i++) {
        substr.append(string.charAt(i));

        String clearedFromSubstrings 
          = string.replaceAll(substr.toString(), "");

        if (clearedFromSubstrings.length() == 0) {
            return true;
        }
    }

    return false;
}
```

让我们创建一些`String`来测试我们的方法:

```
String validString = "aa";
String validStringTwo = "ababab";
String validStringThree = "baeldungbaeldung";

String invalidString = "aca";
String invalidStringTwo = "ababa";
String invalidStringThree = "baeldungnonrepeatedbaeldung";
```

最后，我们可以很容易地检查它的有效性:

```
assertTrue(containsOnlySubstrings(validString));
assertTrue(containsOnlySubstrings(validStringTwo));
assertTrue(containsOnlySubstrings(validStringThree));

assertFalse(containsOnlySubstrings(invalidString));
assertFalse(containsOnlySubstrings(invalidStringTwo));
assertFalse(containsOnlySubstrings(invalidStringThree));
```

虽然这个解决方案有效，**但是它不是很有效**，因为我们迭代了`String`的一半，并且在每次迭代中使用`replaceAll()`方法。

显然，它伴随着性能方面的成本。它会准时运行。

## 4.高效的解决方案

现在，我们将说明另一种方法。

也就是说，我们应该利用这样一个事实，即 **a `String`是由重复的子串组成的，当且仅当它是自身**的非平凡旋转。

这里的旋转意味着我们从 S `tring`的开头移除一些字符，并将它们放在末尾。比如“eldungba”就是“baeldung”的旋转。如果我们旋转一个`String`并得到原始的一个，那么我们可以反复应用这个旋转并得到由重复的子串组成的`String`。

接下来，我们需要检查我们的示例是否是这种情况。为了做到这一点，我们将利用定理，即如果 **`String` A 和`String` B 具有相同的长度，那么我们可以说 A 是 B 的旋转当且仅当 A 是 BB 的子串。**如果我们继续上一段的例子，我们可以证实这个定理:`ba**eldungba**eldung`。

因为我们知道我们的`String` A 总是 AA 的子串，所以我们只需要检查`String` A 是否是 AA 的子串，不包括第一个字符:

```
public static boolean containsOnlySubstringsEfficient(String string) {
    return ((string + string).indexOf(string, 1) != string.length());
}
```

我们可以用和上一个方法一样的方法来测试这个方法。这一次，我们有了`O(n)`时间复杂度。

我们可以在 [`String`分析研究](https://web.archive.org/web/20221205205601/https://www.sciencedirect.com/science/article/pii/S0304397508002880?via%3Dihub)中找到一些关于这个话题的有用定理。

## 5.结论

在本文中，我们用 Java 展示了两种检查`String`是否只包含其子字符串的方法。

本文中使用的所有代码示例都可以从 GitHub 上的[处获得。](https://web.archive.org/web/20221205205601/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-algorithms)