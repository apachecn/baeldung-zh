# 检查字符在 Java 中是否是元音

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-check-character-vowel>

2。使用 indexOf 方法检查元音

## 1.概观

当处理来自`String,`的字符时，我们可能希望根据它们是否属于特定的组来对它们进行分类。例如，英语字母表中的字符不是元音就是辅音。

在本教程中，我们将看看一些方法来检查一个字符是否是元音。我们可以很容易地将这些方法扩展到其他字符组。

## 2.使用`indexOf`方法检查元音

由于我们知道所有的元音，我们可以将它们添加到`String`:

```
String VOWELS = "aeiouAEIOU";
```

**我们可以使用`String`类中的 [`indexOf`方法来查看角色是否出现](https://web.archive.org/web/20220810231115/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html#indexOf(int))**:

```
boolean isInVowelsString(char c) {
    return VOWELS.indexOf(c) != -1;
}
```

如果字符存在，索引将不是`-1`。如果是`-1`，那么这个字符不在元音集中。让我们来测试一下:

```
assertThat(isInVowelsString('e')).isTrue();
assertThat(isInVowelsString('z')).isFalse();
```

这里，我们在 Java 中使用了一个`char`。如果我们的角色是一个单字符`String`对象，我们可以使用不同的实现:

```
boolean isInVowelsString(String c) {
    return VOWELS.contains(c);
}
```

它会通过同样的测试:

```
assertThat(isInVowelsString("e")).isTrue();
assertThat(isInVowelsString("z")).isFalse();
```

正如我们所见，这种方法的实现开销很小。然而，我们必须遍历元音字符串中的 10 个可能的元音，以确定某个内容是否在该组中。

## 3.使用`switch`检查元音

相反，**可以使用`switch`语句，其中每个元音都是单独的`case`** :

```
boolean isVowelBySwitch(char c) {
    switch (c) {
        case 'a':            
        case 'e':           
        case 'i':           
        case 'o':            
        case 'u':            
        case 'A':
        case 'E':            
        case 'I':           
        case 'O':            
        case 'U':
            return true;
        default:
            return false;
    }
}
```

我们也可以测试这个:

```
assertThat(isVowelBySwitch('e')).isTrue();
assertThat(isVowelBySwitch('z')).isFalse();
```

由于 Java 支持在`switch `语句中使用`String`，我们也可以用单字符字符串来实现这一点。

## 4.使用正则表达式检查元音

虽然我们可以实现自己的字符串匹配算法，**Java[正则表达式](/web/20220810231115/https://www.baeldung.com/tag/regex/)引擎允许我们强有力地匹配字符串**。

让我们构建一个正则表达式来识别元音:

```
Pattern VOWELS_PATTERN = Pattern.compile("[aeiou]", Pattern.CASE_INSENSITIVE);
```

`[]`用来表示一个字符类。我们在这个类中只使用小写字母，因为我们可以用不区分大小写的方式匹配它们。

让我们实现我们的匹配算法，用单个字符匹配`String`对象，如下所示:

```
boolean isVowelByRegex(String c) {
    return VOWELS_PATTERN.matcher(c).matches();
}
```

让我们来测试一下:

```
assertThat(isVowelByRegex("e")).isTrue();
assertThat(isVowelByRegex("E")).isTrue();
```

正如我们所见，正则表达式是不区分大小写的。

我们应该注意，这要求输入是一个`String,`而不是一个字符。虽然**我们可以借助`Character`类的`toString`方法**将一个角色转换为`String`:

```
assertThat(isVowelByRegex(Character.toString('e'))).isTrue();
```

使用正则表达式可以简单地处理这个问题的一般情况。我们可以使用字符类指定任何字符分组，包括字符范围。

## 5.我们应该使用哪种解决方案？

基于*字符串*的解决方案可能是最容易理解的，并且执行得相当好，因为它只需要为它分类的每个字符检查最多 10 个选项。

然而，我们通常期望`switch`语句比`String`查找执行得更快。

正则表达式解决方案应该执行得很好，因为正则表达式是在`Pattern`的`compile`方法中优化的。然而，正则表达式实现起来可能更复杂，对于像检测元音这样简单的事情来说，可能不值得这么复杂。类似地，如果我们使用`char`值，那么正则表达式需要一些转换，而其他方法不需要。

然而，**使用正则表达式允许我们实现复杂的表达式来分类字符**。

## 6.结论

在这篇文章中，我们看到了几种不同的方法来识别一个字符是否是一个元音。我们看到了如何使用包含所有元音的字符串，以及如何实现一个`switch`语句。

最后，我们看到了如何使用正则表达式来解决这个问题以及更一般的情况。

一如既往，本教程的完整代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220810231115/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-operations-4)