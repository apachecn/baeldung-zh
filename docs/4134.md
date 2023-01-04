# Java 中正则表达式性能概述

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-regex-performance>

## 1.概观

在这个快速教程中，我们将展示模式匹配引擎是如何工作的。我们还将介绍在 Java 中优化`regular expressions`的不同方法。

关于`regular expressions`的使用介绍，请参考[这里的](/web/20221013140011/https://www.baeldung.com/regular-expressions-java)这篇文章。

## 2.模式匹配引擎

`java.util.regex`包使用一种叫做 **`Nondeterministic Finite Automaton` (NFA)的模式匹配引擎。之所以被认为是`nondeterministic`,是因为当试图匹配给定字符串上的正则表达式时，输入中的每个字符可能会被正则表达式的不同部分检查几次。**

在后台，上面提到的引擎用的是`backtracking`。这个通用算法试图穷尽所有的可能性，直到宣告失败。考虑下面的例子来更好地理解`NFA`:

```
"tra(vel|ce|de)m"
```

输入`String``travel`，引擎首先会寻找`tra`，并立即找到。

之后，它会尝试从第四个字符开始匹配“`vel`”。这个会匹配，所以它会往前走，尝试匹配“`m`”。

这不会匹配，因此，它将返回到第四个字符并搜索“`ce`”。同样，这不会匹配，所以它将再次返回到第四个位置，并尝试使用“`de`”。该字符串也不匹配，因此它将返回到输入字符串中的第二个字符，并尝试搜索另一个“`tra`”。

对于最后一次失败，算法将返回失败。

在最后一个简单的例子中，当试图将输入`String`匹配到正则表达式时，引擎必须回溯几次。正因为如此，**最大限度地减少回溯是很重要的。**

## 3.优化的方法`Regular Expressions`

### 3.1.避免重新编译

Java 中的正则表达式被编译成内部数据结构。这种编译是耗时的过程。

每次我们调用`String.matches(String regex) `方法时，指定的正则表达式都会被重新编译:

```
if (input.matches(regexPattern)) {
    // do something
}
```

正如我们所看到的，每次计算条件时，都会编译正则表达式。

为了优化，可以先编译模式，然后创建一个`Matcher`来查找值中的重合:

```
Pattern pattern = Pattern.compile(regexPattern);
for(String value : values) {
    Matcher matcher = pattern.matcher(value);
    if (matcher.matches()) {
        // do something
    }
}
```

上述优化的替代方法是使用同一个`Matcher`实例及其`reset()`方法:

```
Pattern pattern = Pattern.compile(regexPattern);
Matcher matcher = pattern.matcher("");
for(String value : values) {
    matcher.reset(value);
    if (matcher.matches()) {
      // do something
    }
}
```

**由于`Matcher`不是线程安全的，我们必须小心使用这个变体。在多线程的情况下，这可能很危险。**

总而言之，在我们确定在任何时间点都只有一个用户使用`Matcher`的任何情况下，都可以用`reset`重用它。对于其余部分，重用预编译的就足够了。

### 3.2.使用交替

正如我们在上一节中检查的那样，交替的不充分使用可能对性能有害。将更有可能发生的选项放在前面很重要，这样它们可以更快地匹配。

此外，我们必须提取它们之间的共同模式。这是不一样的:

```
(travel | trade | trace)
```

比:

```
tra(vel | de | ce)
```

后者更快，因为`NFA`将尝试匹配“`tra`”，如果找不到，它不会尝试任何替代。

### 3.3.捕获组

每一次我们捕获群体，我们都会招致一点小小的惩罚。

如果我们不需要捕获组内的文本，我们应该考虑使用非捕获组。请用“`(?:M)`”代替“`(M)`”。

## 4.结论

在这篇简短的文章中，我们简要回顾了`NFA`是如何工作的。然后，我们继续探索如何通过预编译我们的模式和重用一个`Matcher`来优化正则表达式的性能。

最后，我们指出了在使用选项和组时需要记住的一些注意事项。

像往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221013140011/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-regex)