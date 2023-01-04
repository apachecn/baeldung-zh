# 番石榴牧场地图指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/guava-rangemap>

## 1。概述

在本教程中，我们将展示如何使用谷歌番石榴的`RangeMap`界面及其实现。

`RangeMap`是从不相交的非空范围到非空值的一种特殊映射。使用查询，我们可以在该图中查找任何特定范围的值。

`RangeMap`的基本实现是一个`TreeRangeMap`。在内部，map 利用一个`TreeMap`将键存储为一个范围，将值存储为任何定制的 Java 对象。

## 2。谷歌番石榴的`RangeMap`

让我们来看看如何使用`RangeMap`类。

### 2.1。Maven 依赖关系

让我们从在`pom.xml`中添加 Google 的番石榴库依赖项开始:

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>29.0-jre</version>
</dependency>
```

依赖关系的最新版本可以在[这里](https://web.archive.org/web/20211130063123/https://search.maven.org/classic/#search|gav|1|g%3A%22com.google.guava%22%20AND%20a%3A%22guava%22)查看。

## 3。创建

我们创建`RangeMap`实例的一些方法是:

*   使用`TreeRangeMap`类中的`create`方法创建一个可变映射:

```java
RangeMap<Integer, String> experienceRangeDesignationMap
  = TreeRangeMap.create();
```

*   如果我们打算创建一个不可变的范围映射，使用`ImmutableRangeMap`类(它遵循一个构建器模式):

```java
RangeMap<Integer, String> experienceRangeDesignationMap
  = new ImmutableRangeMap.<Integer, String>builder()
  .put(Range.closed(0, 2), "Associate")
  .build(); 
```

## 4。使用

让我们从一个简单的例子开始，展示`RangeMap`的用法。

### 4.1。基于范围内输入的检索

我们可以得到一个与某个整数范围内的值相关联的值:

```java
@Test
public void givenRangeMap_whenQueryWithinRange_returnsSucessfully() {
    RangeMap<Integer, String> experienceRangeDesignationMap 
     = TreeRangeMap.create();

    experienceRangeDesignationMap.put(
      Range.closed(0, 2), "Associate");
    experienceRangeDesignationMap.put(
      Range.closed(3, 5), "Senior Associate");
    experienceRangeDesignationMap.put(
      Range.closed(6, 8),  "Vice President");
    experienceRangeDesignationMap.put(
      Range.closed(9, 15), "Executive Director");

    assertEquals("Vice President", 
      experienceRangeDesignationMap.get(6));
    assertEquals("Executive Director", 
      experienceRangeDesignationMap.get(15));
}
```

注意:

*   `Range`类的`closed`方法假定整数值的范围在 0 到 2 之间(包括 0 和 2)
*   上例中的`Range`由整数组成。我们可以使用任何类型的范围，只要它实现了`Comparable`接口，如`String`、`Character`、浮点小数等。
*   当我们试图获取地图中不存在的范围的值时，`RangeMap`返回`Null`
*   在`ImmutableRangeMap`的情况下，一个键的范围不能与需要插入的键的范围重叠。如果发生这种情况，我们会得到一个`IllegalArgumentException`
*   `RangeMap`中的键和值都不能是`null`。如果他们中的任何一个是`null,`，我们就会得到一个`NullPointerException`

### 4.2。移除基于`Range` 的值

让我们看看如何删除值。在本例中，我们展示了如何删除与整个范围相关联的值。我们还展示了如何基于部分键范围删除值:

```java
@Test
public void givenRangeMap_whenRemoveRangeIsCalled_removesSucessfully() {
    RangeMap<Integer, String> experienceRangeDesignationMap 
      = TreeRangeMap.create();

    experienceRangeDesignationMap.put(
      Range.closed(0, 2), "Associate");
    experienceRangeDesignationMap.put(
      Range.closed(3, 5), "Senior Associate");
    experienceRangeDesignationMap.put(
      Range.closed(6, 8), "Vice President");
    experienceRangeDesignationMap.put(
      Range.closed(9, 15), "Executive Director");

    experienceRangeDesignationMap.remove(Range.closed(9, 15));
    experienceRangeDesignationMap.remove(Range.closed(1, 4));

    assertNull(experienceRangeDesignationMap.get(9));
    assertEquals("Associate", 
      experienceRangeDesignationMap.get(0));
    assertEquals("Senior Associate", 
      experienceRangeDesignationMap.get(5));
    assertNull(experienceRangeDesignationMap.get(1));
}
```

可以看出，即使从一个范围中删除了部分值，如果该范围仍然有效，我们仍然可以获得这些值。

### 4.3。键范围的跨度

如果我们想知道一个`RangeMap` 的总跨度是多少，我们可以使用`span`方法:

```java
@Test
public void givenRangeMap_whenSpanIsCalled_returnsSucessfully() {
    RangeMap<Integer, String> experienceRangeDesignationMap = TreeRangeMap.create();
    experienceRangeDesignationMap.put(Range.closed(0, 2), "Associate");
    experienceRangeDesignationMap.put(Range.closed(3, 5), "Senior Associate");
    experienceRangeDesignationMap.put(Range.closed(6, 8), "Vice President");
    experienceRangeDesignationMap.put(Range.closed(9, 15), "Executive Director");
    experienceRangeDesignationMap.put(Range.closed(16, 30), "Managing Director");
    Range<Integer> experienceSpan = experienceRangeDesignationMap.span();

    assertEquals(0, experienceSpan.lowerEndpoint().intValue());
    assertEquals(30, experienceSpan.upperEndpoint().intValue());
}
```

### 4.4。`SubRangeMap`虎子

当我们想从`RangeMap`中选择一个零件时，我们可以使用`subRangeMap`方法:

```java
@Test
public void givenRangeMap_whenSubRangeMapIsCalled_returnsSubRangeSuccessfully() {
    RangeMap<Integer, String> experienceRangeDesignationMap = TreeRangeMap.create();

    experienceRangeDesignationMap
      .put(Range.closed(0, 2), "Associate");
    experienceRangeDesignationMap
      .put(Range.closed(3, 5), "Senior Associate");
    experienceRangeDesignationMap
      .put(Range.closed(6, 8), "Vice President");
    experienceRangeDesignationMap
      .put(Range.closed(8, 15), "Executive Director");
    experienceRangeDesignationMap
      .put(Range.closed(16, 30), "Managing Director");
    RangeMap<Integer, String> experiencedSubRangeDesignationMap
      = experienceRangeDesignationMap.subRangeMap(Range.closed(4, 14));

    assertNull(experiencedSubRangeDesignationMap.get(3));
    assertTrue(experiencedSubRangeDesignationMap.asMapOfRanges().values()
      .containsAll(Arrays.asList("Executive Director", "Vice President", "Executive Director")));
}
```

该方法返回给定参数`Range`与`RangeMap`的交集。

### 4.5。`Entry`虎子

最后，如果我们从一个`RangeMap`中寻找一个`Entry`，我们使用`getEntry`方法:

```java
@Test
public void givenRangeMap_whenGetEntryIsCalled_returnsEntrySucessfully() {
    RangeMap<Integer, String> experienceRangeDesignationMap 
      = TreeRangeMap.create();

    experienceRangeDesignationMap.put(
      Range.closed(0, 2), "Associate");
    experienceRangeDesignationMap.put(
      Range.closed(3, 5), "Senior Associate");
    experienceRangeDesignationMap.put(
      Range.closed(6, 8), "Vice President");
    experienceRangeDesignationMap.put(
      Range.closed(9, 15), "Executive Director");
    Map.Entry<Range<Integer>, String> experienceEntry 
      = experienceRangeDesignationMap.getEntry(10);

    assertEquals(Range.closed(9, 15), experienceEntry.getKey());
    assertEquals("Executive Director", experienceEntry.getValue());
}
```

## 5。结论

在本教程中，我们举例说明了在番石榴库中使用`RangeMap`的例子。它主要用于根据映射中指定为的键获取值。

这些例子的实现可以在 GitHub 项目中找到——这是一个基于 Maven 的项目，所以它应该很容易导入和运行。