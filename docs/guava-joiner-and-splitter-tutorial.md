# 番石榴–合并和分割系列

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/guava-joiner-and-splitter-tutorial>

## 1。概述

在本教程中，我们将学习如何使用番石榴库中的**`Joiner`和`Splitter`。我们将用`Joiner`将集合转换成字符串，并用`Splitter`将字符串拆分成集合。**

## 2。使用`Joiner` 将`List`转换为`String`

让我们从一个简单的例子开始，使用`Joiner`将一个`List`加入到一个`String`中。在下面的例子中，我们使用逗号“，”作为分隔符将一个`List`的名字连接成一个`String`:

```java
@Test
public void whenConvertListToString_thenConverted() {
    List<String> names = Lists.newArrayList("John", "Jane", "Adam", "Tom");
    String result = Joiner.on(",").join(names);

    assertEquals(result, "John,Jane,Adam,Tom");
}
```

## 3。使用`Joiner` 将`Map`转换为`String`

接下来，让我们看看如何使用`Joiner`将一个`Map`转换成一个`String`。在下面的例子中，我们使用`withKeyValueSeparator()`来连接键和它的值:

```java
@Test
public void whenConvertMapToString_thenConverted() {
    Map<String, Integer> salary = Maps.newHashMap();
    salary.put("John", 1000);
    salary.put("Jane", 1500);
    String result = Joiner.on(" , ").withKeyValueSeparator(" = ")
                                    .join(salary);

    assertThat(result, containsString("John = 1000"));
    assertThat(result, containsString("Jane = 1500"));
}
```

## 4。加入嵌套集合

现在，让我们看看如何将嵌套的集合合并成一个`String`。在下面的例子中，我们将每个`List`转换成一个`String`的结果连接起来:

```java
@Test
public void whenJoinNestedCollections_thenJoined() {
    List<ArrayList<String>> nested = Lists.newArrayList(
      Lists.newArrayList("apple", "banana", "orange"),
      Lists.newArrayList("cat", "dog", "bird"),
      Lists.newArrayList("John", "Jane", "Adam"));
    String result = Joiner.on(";").join(Iterables.transform(nested,
      new Function<List<String>, String>() {
          @Override
          public String apply(List<String> input) {
              return Joiner.on("-").join(input);
          }
      }));

    assertThat(result, containsString("apple-banana-orange"));
    assertThat(result, containsString("cat-dog-bird"));
    assertThat(result, containsString("apple-banana-orange"));
}
```

## 5。使用`Joiner` 处理空值

现在，让我们看看使用 Joiner 时处理空值的不同方法。

要在联接集合时**跳过空值**，请使用`skipNulls()`，如下例所示:

```java
@Test
public void whenConvertListToStringAndSkipNull_thenConverted() {
    List<String> names = Lists.newArrayList("John", null, "Jane", "Adam", "Tom");
    String result = Joiner.on(",").skipNulls().join(names);

    assertEquals(result, "John,Jane,Adam,Tom");
}
```

如果你不想跳过空值，而想用**代替**，使用`useForNull()`，如下例所示:

```java
@Test
public void whenUseForNull_thenUsed() {
    List<String> names = Lists.newArrayList("John", null, "Jane", "Adam", "Tom");
    String result = Joiner.on(",").useForNull("nameless").join(names);

    assertEquals(result, "John,nameless,Jane,Adam,Tom");
}
```

注意`useForNull()`不改变原始列表，它只影响连接的输出。

## 6。使用`Splitter` 从`String`创建`List`

现在，让我们看看如何将一个`String`分割成一个`List`。在下面的例子中，我们使用“-”分隔符将输入`String`拆分为`List`:

```java
@Test
public void whenCreateListFromString_thenCreated() {
    String input = "apple - banana - orange";
    List<String> result = Splitter.on("-").trimResults()
                                          .splitToList(input);

    assertThat(result, contains("apple", "banana", "orange"));
}
```

注意`trimResults()`从结果子字符串中删除了前导和尾部的空白。

## 7。使用`Splitter` 从`String`创建`Map`

接下来，让我们看看如何使用拆分器从字符串创建映射。在下面的例子中，我们使用`withKeyValueSeparator()`将一个`String`分割成一个`Map`:

```java
@Test
public void whenCreateMapFromString_thenCreated() {
    String input = "John=first,Adam=second";
    Map<String, String> result = Splitter.on(",")
                                         .withKeyValueSeparator("=")
                                         .split(input);

    assertEquals("first", result.get("John"));
    assertEquals("second", result.get("Adam"));
}
```

## 8。用多个分隔符分割`String`

现在，让我们看看如何用多个分隔符分割一个`String`。在下面的例子中，我们使用两个“.”和“，”来分裂我们的`String`:

```java
@Test
public void whenSplitStringOnMultipleSeparator_thenSplit() {
    String input = "apple.banana,,orange,,.";
    List<String> result = Splitter.onPattern("[.,]")
                                  .omitEmptyStrings()
                                  .splitToList(input);

    assertThat(result, contains("apple", "banana", "orange"));
}
```

注意，`omitEmptyStrings()`忽略空字符串，不将它们添加到结果`List`中。

## 9。以特定长度分割一个`String`

接下来——让我们看看如何以特定长度分割一个`String`。在下面的例子中，我们每隔 3 个字符分割我们的`String`:

```java
@Test
public void whenSplitStringOnSpecificLength_thenSplit() {
    String input = "Hello world";
    List<String> result = Splitter.fixedLength(3).splitToList(input);

    assertThat(result, contains("Hel", "lo ", "wor", "ld"));
}
```

## 10。限制分割结果

最后，让我们看看如何限制分割结果。如果您想让`Splitter`到**在特定数量的项目**后停止分割–使用`limit()`，如下例所示:

```java
@Test
public void whenLimitSplitting_thenLimited() {
    String input = "a,b,c,d,e";
    List<String> result = Splitter.on(",")
                                  .limit(4)
                                  .splitToList(input);

    assertEquals(4, result.size());
    assertThat(result, contains("a", "b", "c", "d,e"));
}
```

## 11。结论

在本教程中，我们展示了如何使用 Guava 中的`Joiner`和`Splitter`在集合和字符串之间进行各种转换。

所有这些示例和代码片段的实现可以在[我的番石榴 github 项目](https://web.archive.org/web/20220703154648/https://github.com/eugenp/tutorials/tree/master/guava-modules/guava-collections "The Github Project with the impl of all examples using Guava") 中找到——这是一个基于 Eclipse 的项目，所以它应该很容易导入和运行。