# 番石榴系列指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/guava-rangeset>

## 1。概述

在本教程中，我们将展示如何使用谷歌番石榴的`RangeSet`界面及其实现。

`RangeSet`是由零个或多个非空的、不连续的范围组成的集合。当添加一个范围到一个可变的`RangeSet`时，任何连接的范围被合并在一起，而空的范围被忽略。

`RangeSet`的基本实现是一个`TreeRangeSet`。

## 2。谷歌番石榴的`RangeSet`

让我们来看看如何使用`RangeSet`类。

### 2.1。Maven 依赖关系

让我们从在`pom.xml`中添加 Google 的番石榴库依赖项开始:

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

依赖关系的最新版本可以在[这里](https://web.archive.org/web/20220812065256/https://search.maven.org/classic/#search|gav|1|g%3A%22com.google.guava%22%20AND%20a%3A%22guava%22)查看。

## 3。创作

让我们探索一下创建`RangeSet`实例的一些方法。

首先，我们可以使用类`TreeRangeSet`中的`create`方法来创建一个可变集合:

```java
RangeSet<Integer> numberRangeSet = TreeRangeSet.create();
```

如果我们已经有了集合，使用类`TreeRangeSet`的`create`方法通过传递集合来创建一个可变集合:

```java
List<Range<Integer>> numberList = Arrays.asList(Range.closed(0, 2));
RangeSet<Integer> numberRangeSet = TreeRangeSet.create(numberList);
```

最后，如果我们需要创建一个不可变的范围集合，使用`ImmutableRangeSet`类(按照构建器模式创建):

```java
RangeSet<Integer> numberRangeSet 
  = new ImmutableRangeSet.<Integer>builder().add(Range.closed(0, 2)).build(); 
```

## 4。用途

让我们从一个简单的例子开始，展示一下`RangeSet`的用法。

### 4.1。添加到范围

我们可以检查所提供的输入是否在集合中任何范围项目中的范围内:

```java
@Test
public void givenRangeSet_whenQueryWithinRange_returnsSucessfully() {
    RangeSet<Integer> numberRangeSet = TreeRangeSet.create();

    numberRangeSet.add(Range.closed(0, 2));
    numberRangeSet.add(Range.closed(3, 5));
    numberRangeSet.add(Range.closed(6, 8));

    assertTrue(numberRangeSet.contains(1));
    assertFalse(numberRangeSet.contains(9));
}
```

注意事项:

*   `Range`类的`closed`方法假定整数值的范围在 0 到 2 之间(包括 0 和 2)
*   上例中的`Range`由整数组成。我们可以使用由任何类型组成的范围，只要它实现了`Comparable`接口，如`String`、`Character`、浮点小数等
*   在`ImmutableRangeSet`的情况下，集合中存在的范围项目不能与想要添加的范围项目重叠。如果发生这种情况，我们会得到一个`IllegalArgumentException`
*   输入到`RangeSet`的范围不能为空。如果输入是`null`，我们将得到一个`NullPointerException`

### 4.2。删除范围

让我们看看如何从`RangeSet`中移除值:

```java
@Test
public void givenRangeSet_whenRemoveRangeIsCalled_removesSucessfully() {
    RangeSet<Integer> numberRangeSet = TreeRangeSet.create();

    numberRangeSet.add(Range.closed(0, 2));
    numberRangeSet.add(Range.closed(3, 5));
    numberRangeSet.add(Range.closed(6, 8));
    numberRangeSet.add(Range.closed(9, 15));
    numberRangeSet.remove(Range.closed(3, 5));
    numberRangeSet.remove(Range.closed(7, 10));

    assertTrue(numberRangeSet.contains(1));
    assertFalse(numberRangeSet.contains(9));
    assertTrue(numberRangeSet.contains(12));
}
```

可以看出，在删除之后，我们仍然可以访问集合中剩余的任何范围项中的值。

### 4.3。范围跨度

现在让我们看看`RangeSet` 的总跨度是多少:

```java
@Test
public void givenRangeSet_whenSpanIsCalled_returnsSucessfully() {
    RangeSet<Integer> numberRangeSet = TreeRangeSet.create();

    numberRangeSet.add(Range.closed(0, 2));
    numberRangeSet.add(Range.closed(3, 5));
    numberRangeSet.add(Range.closed(6, 8));
    Range<Integer> experienceSpan = numberRangeSet.span();

    assertEquals(0, experienceSpan.lowerEndpoint().intValue());
    assertEquals(8, experienceSpan.upperEndpoint().intValue());
}
```

### 4.4。获取子范围

如果我们希望根据给定的`Range`得到`RangeSet`的一部分，我们可以使用`subRangeSet`方法:

```java
@Test
public void 
  givenRangeSet_whenSubRangeSetIsCalled_returnsSubRangeSucessfully() {

    RangeSet<Integer> numberRangeSet = TreeRangeSet.create();

    numberRangeSet.add(Range.closed(0, 2));
    numberRangeSet.add(Range.closed(3, 5));
    numberRangeSet.add(Range.closed(6, 8));
    RangeSet<Integer> numberSubRangeSet 
      = numberRangeSet.subRangeSet(Range.closed(4, 14));

    assertFalse(numberSubRangeSet.contains(3));
    assertFalse(numberSubRangeSet.contains(14));
    assertTrue(numberSubRangeSet.contains(7));
}
```

### 4.5。补码方法

接下来，让我们使用`complement`方法获得除了`RangeSet`中的值之外的所有值:

```java
@Test
public void givenRangeSet_whenComplementIsCalled_returnsSucessfully() {
    RangeSet<Integer> numberRangeSet = TreeRangeSet.create();

    numberRangeSet.add(Range.closed(0, 2));
    numberRangeSet.add(Range.closed(3, 5));
    numberRangeSet.add(Range.closed(6, 8));
    RangeSet<Integer> numberRangeComplementSet
      = numberRangeSet.complement();

    assertTrue(numberRangeComplementSet.contains(-1000));
    assertFalse(numberRangeComplementSet.contains(2));
    assertFalse(numberRangeComplementSet.contains(3));
    assertTrue(numberRangeComplementSet.contains(1000));
}
```

### 4.6。与范围的交集

最后，当我们想要检查`RangeSet`中出现的范围区间是否与另一个给定范围中的一些或所有值相交时，我们可以利用`intersect`方法:

```java
@Test
public void givenRangeSet_whenIntersectsWithinRange_returnsSucessfully() {
    RangeSet<Integer> numberRangeSet = TreeRangeSet.create();

    numberRangeSet.add(Range.closed(0, 2));
    numberRangeSet.add(Range.closed(3, 10));
    numberRangeSet.add(Range.closed(15, 18));

    assertTrue(numberRangeSet.intersects(Range.closed(4, 17)));
    assertFalse(numberRangeSet.intersects(Range.closed(19, 200)));
}
```

## 5。结论

在本教程中，我们用一些例子说明了番石榴库的`RangeSet`。`RangeSet`主要用于检查一个值是否落在集合中的某个范围内。

这些例子的实现可以在 GitHub 项目中找到——这是一个基于 Maven 的项目，所以应该很容易导入和运行。