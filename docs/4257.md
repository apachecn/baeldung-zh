# 检查字符串是否包含子串

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-string-contains-substring>

## 1.概观

在本教程中，我们将回顾几种检查`String`是否包含子串的方法，并比较每种方法的性能。

## 2.`String.indexOf`

让我们首先尝试使用`String.indexOf`方法。 **[`indexOf`](/web/20221206155502/https://www.baeldung.com/string/index-of) 给出找到子串的第一个位置，如果没有找到，则为-1。**

当我们搜索“Rhap”时，它将返回 9:

```
Assert.assertEquals(9, "Bohemian Rhapsodyan".indexOf("Rhap"));
```

当我们搜索“rhap”时，**它将返回-1，因为它区分大小写。**

```
Assert.assertEquals(-1, "Bohemian Rhapsodyan".indexOf("rhap"));
Assert.assertEquals(9, "Bohemian Rhapsodyan".toLowerCase().indexOf("rhap"));
```

同样需要注意的是，如果我们搜索`substring`“an”，它将返回 6，因为它返回的是第一个匹配项:

```
Assert.assertEquals(6, "Bohemian Rhapsodyan".indexOf("an"));
```

## 3.`String.contains`

接下来，我们来试试`String.contains`。 **[`contains`](/web/20221206155502/https://www.baeldung.com/string/contains) 将在整个`String`中搜索一个子串，如果找到则返回`true`，否则返回`false`。**

在本例中，`contains`返回`true`，因为找到了“Hey”。

```
Assert.assertTrue("Hey Ho, let's go".contains("Hey"));
```

如果没有找到字符串，`contains`返回`false`:

```
Assert.assertFalse("Hey Ho, let's go".contains("jey"));
```

在最后一个例子中，没有找到“hey ”,因为 **`String.contains`是区分大小写的。**

```
Assert.assertFalse("Hey Ho, let's go".contains("hey"));
Assert.assertTrue("Hey Ho, let's go".toLowerCase().contains("hey"));
```

有意思的一点是， **`contains`内部调用`indexOf`** 来知道一个`substring`是否包含，或者不包含。

## 4.`StringUtils.containsIgnoreCase`

我们的第三种方法将使用来自 [Apache Commons Lang](https://web.archive.org/web/20221206155502/https://commons.apache.org/proper/commons-lang/) 库的`**StringUtils#**` **`containsIgnoreCase`:**

```
Assert.assertTrue(StringUtils.containsIgnoreCase("Runaway train", "train"));
Assert.assertTrue(StringUtils.containsIgnoreCase("Runaway train", "Train"));
```

我们可以看到它**会检查一个`substring`是否包含在一个`String`中，而忽略大小写**。这就是为什么当我们搜索“trai”以及“失控列车”中的“Trai”时， [`containsIgnoreCase`](/web/20221206155502/https://www.baeldung.com/string-processing-commons-lang) 会返回`true`。

这种方法**不会像以前的方法**那样有效，因为它需要额外的时间来忽略这个案例。`containsIgnoreCase`在内部将每个字母转换成大写字母，并比较转换后的字母而不是原始字母。

## 5.使用`Pattern`

我们的最后一种方法将使用带有正则表达式的 **`Pattern`:**

```
Pattern pattern = Pattern.compile("(?<!\\S)" + "road" + "(?!\\S)");
```

我们可以看到，**我们需要首先构建 [`Pattern`](/web/20221206155502/https://www.baeldung.com/regular-expressions-java) ，然后我们需要创建`Matcher`，最后，我们可以用`find`方法检查是否出现子串:**

```
Matcher matcher = pattern.matcher("Hit the road Jack");
Assert.assertTrue(matcher.find());
```

例如，第一次执行`find`时，它返回`true`,因为单词“road”包含在字符串“Hit the road Jack”中，但是当我们试图在字符串“and don’t back no more”中查找相同的单词时，它返回`false:`

```
Matcher matcher = pattern.matcher("and don't you come back no more");
Assert.assertFalse(matcher.find());
```

## 6.性能比较

我们将使用一个名为 [`Java Microbenchmark Harness`](/web/20221206155502/https://www.baeldung.com/java-microbenchmark-harness) (JMH)的开源微基准框架，以决定哪种方法在执行时间方面最有效。

## 6.1.基准设置

正如在每个 JMH 基准测试中一样，我们有能力编写一个`setup`方法，以便在我们的基准测试运行之前准备好某些东西:

```
@Setup
public void setup() {
    message = "Lorem ipsum dolor sit amet, consectetur adipiscing elit, " + 
      "sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. " + 
      "Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris " + 
      "nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in " + 
      "reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. " + 
      "Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt " + 
      "mollit anim id est laborum";
    pattern = Pattern.compile("(?<!\\S)" + "eiusmod" + "(?!\\S)");
}
```

在`setup`方法中，我们正在初始化`message`字段。我们将用它作为各种搜索实现的源文本。

我们也正在初始化`pattern`，以便在我们的一个基准测试中使用它。

## 6.2.`String.indexOf`基准

我们的第一个基准测试将使用`indexOf`:

```
@Benchmark
public int indexOf() {
    return message.indexOf("eiusmod");
}
```

我们将搜索“eiusmod”出现在`message`变量中的哪个位置。

## 6.3.`String.contains`基准

我们的第二个基准将使用`contains`:

```
@Benchmark
public boolean contains() {
    return message.contains("eiusmod");
}
```

我们将尝试查找`message`值`contains`“eiusmod”，是否与之前基准测试中使用的`substring`相同。

## 6.4.`StringUtils.containsIgnoreCase`基准

我们的第三个基准将使用`StringUtils#` `containsIgnoreCase`:

```
@Benchmark
public boolean containsStringUtilsIgnoreCase() {
    return StringUtils.containsIgnoreCase(message, "eiusmod");
}
```

与之前的基准测试一样，我们将在`message`值中搜索`substring`。

## 6.5.`Pattern`基准

我们最后的基准测试将使用`Pattern`:

```
@Benchmark
public boolean searchWithPattern() {
    return pattern.matcher(message).find();
}
```

我们将使用在`setup`方法中初始化的模式来创建一个`Matcher`，并能够调用`find`方法，使用与前面相同的子串。

## 6.6.基准结果分析

需要注意的是**我们正在评估纳秒级的基准测试结果**。

运行我们的 JMH 测试后，我们可以看到每个测试花费的平均时间:

*   `contains` : 14.736 纳秒
*   `indexOf` : 14.200 纳秒
*   `containsStringUtilsIgnoreCase` : 385.632 纳秒
*   `searchWithPattern` : 1014.633 纳秒

`indexOf`法是效率最高的一种，其次是`contains`。**`contains`花更长时间是有道理的，因为正在内部使用`indexOf`。**

与之前的相比花费了额外的时间，因为它不区分大小写。

`searchWithPattern`，上一次花费了更长的平均时间，**证明使用`Pattern` s 是这个任务最差的选择。**

## 7.结论

在本文中，我们探索了在`String.` 中搜索子串的各种方法，还对不同解决方案的性能进行了基准测试。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20221206155502/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-operations)