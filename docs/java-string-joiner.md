# Java 8 字符串连接器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-string-joiner>

## 1。简介

`StringJoiner`是 Java 8 中在`java.util`包下新增的一个类。

简而言之，**它可以通过分隔符、前缀和后缀来连接`Strings`。**

## 2。添加元素

我们可以使用`add()`方法添加`Strings`:

```java
@Test
public void whenAddingElements_thenJoinedElements() {
    StringJoiner joiner = new StringJoiner(",", PREFIX, SUFFIX);
    joiner.add("Red")
      .add("Green")
      .add("Blue");

    assertEquals(joiner.toString(), "[Red,Green,Blue]");
}
```

如果我们想连接一个列表中的所有元素，我们就必须遍历这个列表。不幸的是，使用`StringJoiner`很难做到这一点:

```java
@Test
public void whenAddingListElements_thenJoinedListElements() {
    List<String> rgbList = new ArrayList<>();
    rgbList.add("Red");
    rgbList.add("Green");
    rgbList.add("Blue");

    StringJoiner rgbJoiner = new StringJoiner(
      ",", PREFIX, SUFFIX);

    for (String color : rgbList) {
        rgbJoiner.add(color);
    }

    assertEquals(rgbJoiner.toString(), "[Red,Green,Blue]");
}
```

## 3。施工

为了构造一个`StringJoiner,`的实例，我们需要提到分隔符。可选地，我们还可以指定应该出现在结果中的前缀和后缀:

```java
private String PREFIX = "[";
private String SUFFIX = "]";

@Test
public void whenEmptyJoinerWithoutPrefixSuffix_thenEmptyString() {
    StringJoiner joiner = new StringJoiner(",");

    assertEquals(0, joiner.toString().length());
}

@Test
public void whenEmptyJoinerJoinerWithPrefixSuffix_thenPrefixSuffix() {
    StringJoiner joiner = new StringJoiner(
      ",", PREFIX, SUFFIX);

    assertEquals(joiner.toString(), PREFIX + SUFFIX);
}
```

我们使用`toString()`从 joiner 获取当前值。

注意由联接器返回的默认值。**没有前缀和后缀的`Joiner`返回一个空的`String`，而有前缀和后缀的 joiner 返回一个包含前缀和后缀的`String`。**

我们可以通过使用`setEmptyValue()`来改变默认返回的`String`:

```java
@Test
public void whenEmptyJoinerWithEmptyValue_thenDefaultValue() {
    StringJoiner joiner = new StringJoiner(",");
    joiner.setEmptyValue("default");

    assertEquals(joiner.toString(), "default");
}

@Test
public void whenEmptyJoinerWithPrefixSuffixAndEmptyValue_thenDefaultValue() {
    StringJoiner joiner = new StringJoiner(",", PREFIX, SUFFIX);
    joiner.setEmptyValue("default");

    assertEquals(joiner.toString(), "default");
}
```

这里，两个 joiners 都返回常数`EMPTY_JOINER`。

**只有当`StringJoiner`为空时才返回默认值。**

## 4。合并加入者

我们可以使用`merge()`来合并两个 joiners。它将给定的`StringJoiner` **的内容加上不带前缀和后缀的**作为下一个元素:

```java
@Test
public void whenMergingJoiners_thenReturnMerged() {
    StringJoiner rgbJoiner = new StringJoiner(
      ",", PREFIX, SUFFIX);
    StringJoiner cmybJoiner = new StringJoiner(
      "-", PREFIX, SUFFIX);

    rgbJoiner.add("Red")
      .add("Green")
      .add("Blue");
    cmybJoiner.add("Cyan")
      .add("Magenta")
      .add("Yellow")
      .add("Black");

    rgbJoiner.merge(cmybJoiner);

    assertEquals(
      rgbJoiner.toString(), 
      "[Red,Green,Blue,Cyan-Magenta-Yellow-Black]");
}
```

**注意如何使用`“-“`来连接`cmybJoiner`的内容，而`rgbJoiner`仍然使用`“,”.`**

## 5。`Stream` API

这就是我们对`StringJoiner`所能做的一切。

在`Stream` API 中还可以找到一种间接用法:

```java
@Test
public void whenUsedWithinCollectors_thenJoined() {
    List<String> rgbList = Arrays.asList("Red", "Green", "Blue");
    String commaSeparatedRGB = rgbList.stream()
      .map(color -> color.toString())
      .collect(Collectors.joining(","));

    assertEquals(commaSeparatedRGB, "Red,Green,Blue");
}
```

**`Collectors.joining()`内部使用`StringJoiner`执行加入操作。**

## 6。结论

在这个快速教程中，我们演示了如何使用`StringJoiner`类。总的来说，`StringJoiner`看起来非常原始，没有解决一些基本的用例，比如连接列表的元素。它似乎主要是为`Collectors`设计的。

如果`StringJoiner`不符合我们的要求，还有其他流行且强大的库，比如`Guava`。

和往常一样，所有的资源都可以在 GitHub 上找到[。](https://web.archive.org/web/20220523143250/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-apis)