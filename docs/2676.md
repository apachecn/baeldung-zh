# 删除字符串中的前导字符和尾随字符

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-remove-trailing-characters>

## 1.介绍

在这个简短的教程中，我们将看到几种从`String`中移除前导和尾随字符的方法。为了简单起见，我们将在示例中删除零。

对于每个实现，我们将创建两个方法:一个用于前导零，一个用于尾随零。

这个问题有一个边缘情况:当输入只包含零时，我们想做什么？返回一个空的`String`，还是一个包含单个零的`String`？我们将在每个解决方案中看到这两种用例的实现。

我们对每个实现都有单元测试，你可以在 GitHub 上找到[。](https://web.archive.org/web/20220826110932/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-algorithms-2)

## 2.使用`StringBuilder`

在我们的第一个解决方案中，我们将使用原始的`String`创建一个 **`StringBuilder`，并从开头或结尾删除不必要的字符**:

```
String removeLeadingZeroes(String s) {
    StringBuilder sb = new StringBuilder(s);
    while (sb.length() > 0 && sb.charAt(0) == '0') {
        sb.deleteCharAt(0);
    }
    return sb.toString();
}

String removeTrailingZeroes(String s) {
    StringBuilder sb = new StringBuilder(s);
    while (sb.length() > 0 && sb.charAt(sb.length() - 1) == '0') {
        sb.setLength(sb.length() - 1);
    }
    return sb.toString();
}
```

请注意，当我们删除尾随零时，我们使用`StringBuilder.setLength()`而不是`StringBuilder.deleteCharAt()`,因为它也删除了最后几个字符，这样性能更好。

如果我们**不想在输入只包含零的时候返回一个空的`String`** ，我们唯一需要做的就是**在只剩下一个字符**的时候停止循环。

因此，我们改变循环条件:

```
String removeLeadingZeroes(String s) {
    StringBuilder sb = new StringBuilder(s);
    while (sb.length() > 1 && sb.charAt(0) == '0') {
        sb.deleteCharAt(0);
    }
    return sb.toString();
}

String removeTrailingZeroes(String s) {
    StringBuilder sb = new StringBuilder(s);
    while (sb.length() > 1 && sb.charAt(sb.length() - 1) == '0') {
        sb.setLength(sb.length() - 1);
    }
    return sb.toString();
}
```

## 3.使用`String.subString()`

在这个解决方案中，当我们删除前导零或尾随零时，我们**找到第一个或最后一个非零字符的位置。**

之后，我们只需调用`substring()`，返回剩余部分:

```
String removeLeadingZeroes(String s) {
    int index;
    for (index = 0; index < s.length(); index++) {
        if (s.charAt(index) != '0') {
            break;
        }
    }
    return s.substring(index);
}

String removeTrailingZeroes(String s) {
    int index;
    for (index = s.length() - 1; index >= 0; index--) {
        if (s.charAt(index) != '0') {
            break;
        }
    }
    return s.substring(0, index + 1);
}
```

注意，我们必须在 for 循环之前声明变量`index`,因为我们希望在循环范围之外使用该变量。

还要注意，我们必须手动寻找非零字符，因为`String.indexOf()`和`String.lastIndexOf()`仅用于精确匹配。

如果我们**不想返回一个空的`String`** ，我们必须做和之前一样的事情:**改变循环条件**:

```
String removeLeadingZeroes(String s) {
    int index;
    for (index = 0; index < s.length() - 1; index++) {
        if (s.charAt(index) != '0') {
            break;
        }
    }
    return s.substring(index);
}

String removeTrailingZeroes(String s) {
    int index;
    for (index = s.length() - 1; index > 0; index--) {
        if (s.charAt(index) != '0') {
            break;
        }
    }
    return s.substring(0, index + 1);
}
```

## 4.使用 Apache Commons

Apache Commons 有很多有用的类，包括`org.apache.commons.lang.StringUtils`。更准确地说，这个类在 Apache Commons Lang3 中。

### 4.1.属国

我们可以通过将这个依赖关系插入到我们的`pom.xml`文件中来使用 [Apache Commons Lang3](https://web.archive.org/web/20220826110932/https://search.maven.org/search?q=a:commons-lang3) :

```
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

### 4.2.履行

在`StringUtils`类中，我们有方法`stripStart()`和`stripEnd()`。它们分别删除前导字符和尾随字符。

因为这正是我们所需要的，所以我们的解决方案非常简单:

```
String removeLeadingZeroes(String s) {
    return StringUtils.stripStart(s, "0");
}

String removeTrailingZeroes(String s) {
    return StringUtils.stripEnd(s, "0");
}
```

不幸的是，我们不能配置，如果我们想删除所有事件或没有。因此，我们需要手动控制它。

如果输入不为空，但是被剥离的`String`为空，那么我们必须返回一个零:

```
String removeLeadingZeroes(String s) {
    String stripped = StringUtils.stripStart(s, "0");
    if (stripped.isEmpty() && !s.isEmpty()) {
        return "0";
    }
    return stripped;
}

String removeTrailingZeroes(String s) {
    String stripped = StringUtils.stripEnd(s, "0");
    if (stripped.isEmpty() && !s.isEmpty()) {
        return "0";
    }
    return stripped;
}
```

注意，这些方法接受一个`String`作为它们的第二个参数。这个`String`代表一组字符，而不是我们想要删除的序列。

例如，如果我们通过`“01”`，他们将删除任何前导或尾随字符，这些字符不是`‘0'`就是`‘1'`。

## 5.用番石榴

番石榴还提供了许多实用程序类。对于这个问题，我们可以使用`com.google.common.base.CharMatcher`，它提供了与匹配字符进行交互的实用方法。

### 5.1.属国

要使用番石榴，我们应该将下面的[依赖项](https://web.archive.org/web/20220826110932/https://search.maven.org/search?q=a:guava)添加到我们的`pom.xml`文件中:

```
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

注意，如果我们想在 Android 应用程序中使用 Guava，我们应该使用版本`27.0-android`来代替。

### 5.2.履行

在我们的例子中，我们对`trimLeadingFrom()`和`trimTrailingFrom()`感兴趣。

顾名思义，它们分别从与`CharMatcher`匹配的`String`中移除任何前导或尾随字符:

```
String removeLeadingZeroes(String s) {
    return CharMatcher.is('0').trimLeadingFrom(s);
}

String removeTrailingZeroes(String s) {
    return CharMatcher.is('0').trimTrailingFrom(s);
}
```

它们与我们看到的 Apache Commons 方法具有相同的特征。

因此，如果我们不想删除所有的零，我们可以使用相同的技巧:

```
String removeLeadingZeroes(String s) {
    String stripped = CharMatcher.is('0').trimLeadingFrom(s);
    if (stripped.isEmpty() && !s.isEmpty()) {
        return "0";
    }
    return stripped;
}

String removeTrailingZeroes(String s) {
    String stripped = CharMatcher.is('0').trimTrailingFrom(s);
    if (stripped.isEmpty() && !s.isEmpty()) {
        return "0";
    }
    return stripped;
}
```

注意，使用`CharMatcher`我们可以创建更复杂的匹配规则。

## 6.使用正则表达式

由于我们的问题是一个模式匹配问题，我们可以使用正则表达式:**我们希望匹配一个`String`的开头或结尾**的所有零。

最重要的是，我们希望删除那些匹配的零。换句话说，我们想用空的**来代替它们，或者换句话说，一个空的`String`** 。

我们可以通过`String.replaceAll()`方法做到这一点:

```
String removeLeadingZeroes(String s) {
    return s.replaceAll("^0+", "");
}

String removeTrailingZeroes(String s) {
    return s.replaceAll("0+$", "");
}
```

如果我们不想删除所有的零，我们可以使用与 Apache Commons 和 Guava 相同的解决方案。然而，有一个纯正则表达式的方法可以做到这一点:我们必须提供一个模式，它不匹配整个`String`。

这样，如果输入只包含零，regexp 引擎将从匹配中只保留一个。我们可以通过以下模式做到这一点:

```
String removeLeadingZeroes(String s) {
    return s.replaceAll("^0+(?!$)", "");
}

String removeTrailingZeroes(String s) {
    return s.replaceAll("(?!^)0+$", "");
}
```

注意，`“(?!^)”`和`“(?!$)”`分别意味着它不是`String`的开始或结束。

## 7.结论

在本教程中，我们看到了几种从`String`中移除前导和尾随字符的方法。这些实现之间的选择通常只是个人偏好。

像往常一样，这些例子可以在 GitHub 的[上找到。](https://web.archive.org/web/20220826110932/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-algorithms-2)