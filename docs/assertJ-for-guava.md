# 番石榴协会

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/assertJ-for-guava>

[This article is part of a series:](javascript:void(0);)[• Introduction to AssertJ](/web/20220117213330/https://www.baeldung.com/introduction-to-assertj)
• AssertJ for Guava (current article)[• AssertJ’s Java 8 Features](/web/20220117213330/https://www.baeldung.com/assertJ-java-8-features)
[• Custom Assertions with AssertJ](/web/20220117213330/https://www.baeldung.com/assertj-custom-assertion)

## 1。概述

本文主要关注与番石榴相关的断言，是 AssertJ 系列的第二篇文章。如果你想了解一些关于 AssertJ 的一般信息，可以看看这个系列的第一篇文章[AssertJ 简介](/web/20220117213330/https://www.baeldung.com/introduction-to-assertj)。

## 2。Maven 依赖关系

为了将 AssertJ 与 Guava 一起使用，您需要将以下依赖项添加到您的`pom.xml`中:

```java
<dependency>
    <groupId>org.assertj</groupId>
    <artifactId>assertj-guava</artifactId>
    <version>3.0.0</version>
    <scope>test</scope>
</dependency>
```

你可以在这里找到最新版本[。](https://web.archive.org/web/20220117213330/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22assertj-guava%22)

并且注意，从 3.0.0 版本开始，`AssertJ Guava`依赖于`Java 8`和`AssertJ Core 3.x`。

## 3。番石榴断言在行动

`AssertJ`对番石榴类型有自定义断言:`ByteSource`、`Multimap`、`Optional`、`Range`、`RangeMap`和`Table`。

### 3.1。`ByteSource`断言

让我们从创建两个空的临时文件开始:

```java
File temp1 = File.createTempFile("bael", "dung1");
File temp2 = File.createTempFile("bael", "dung2");
```

并从中创建`ByteSource`实例:

```java
ByteSource byteSource1 = Files.asByteSource(temp1);
ByteSource byteSource2 = Files.asByteSource(temp2);
```

现在我们可以写出下面的断言:

```java
assertThat(buteSource1)
  .hasSize(0)
  .hasSameContentAs(byteSource2); 
```

### 3.2。`Multimap` 断言

是可以将一个以上的值与给定的键相关联的映射。`Multimap`断言的工作方式与普通的`Map`实现非常相似。

让我们首先创建一个`Multimap`实例并添加一些条目:

```java
Multimap<Integer, String> mmap = Multimaps
  .newMultimap(new HashMap<>(), Sets::newHashSet);
mmap.put(1, "one");
mmap.put(1, "1");
```

现在我们可以断言:

```java
assertThat(mmap)
  .hasSize(2)
  .containsKeys(1)
  .contains(entry(1, "one"))
  .contains(entry(1, "1"));
```

还有另外两个可用的断言，它们之间有细微的区别:

*   `containsAllEntriesOf`和
*   `hasSameEntriesAs.`

让我们来看看这两个断言；我们将从定义一些地图开始:

```java
Multimap<Integer, String> mmap1 = ArrayListMultimap.create();
mmap1.put(1, "one");
mmap1.put(1, "1");
mmap1.put(2, "two");
mmap1.put(2, "2");

Multimap<Integer, String> mmap1_clone = Multimaps
  .newSetMultimap(new HashMap<>(), HashSet::new);
mmap1_clone.put(1, "one");
mmap1_clone.put(1, "1");
mmap1_clone.put(2, "two");
mmap1_clone.put(2, "2");

Multimap<Integer, String> mmap2 = Multimaps
  .newSetMultimap(new HashMap<>(), HashSet::new);
mmap2.put(1, "one");
mmap2.put(1, "1");
```

如您所见，`mmap1`和`mmap1_clone`包含完全相同的条目，但却是两种不同类型的`Map`的两个不同对象。`Map mmap2`包含所有地图共享的单个条目。现在下面的断言是正确的:

```java
assertThat(mmap1)
  .containsAllEntriesOf(mmap2)
  .containsAllEntriesOf(mmap1_clone)
  .hasSameEntriesAs(mmap1_clone);
```

### 3.3。`Optional` 断言

Guava 的`Optional`断言包括值存在检查和提取内部值的实用程序。

让我们从创建一个`Optional`实例开始:

```java
Optional<String> something = Optional.of("something");
```

现在我们可以检查值的存在并断言`Optional`的内容:

```java
assertThat(something)
  .isPresent()
  .extractingValue()
  .isEqualTo("something");
```

### 3.4。`Range` 断言

Guava 的`Range`类的断言包括检查`Range`的下限和上限，或者某个值是否在给定的范围内。

让我们通过执行以下操作来定义一个简单的字符范围:

```java
Range<String> range = Range.openClosed("a", "g");
```

现在我们可以测试:

```java
assertThat(range)
  .hasOpenedLowerBound()
  .isNotEmpty()
  .hasClosedUpperBound()
  .contains("b");
```

### 3.5。`Table` 断言

AssertJ 的特定于表的断言允许检查行和列计数以及单元格值的存在。

让我们创建一个简单的`Table`实例:

```java
Table<Integer, String, String> table = HashBasedTable.create(2, 2);
table.put(1, "A", "PRESENT");
table.put(1, "B", "ABSENT");
```

现在我们可以执行以下检查:

```java
assertThat(table)
  .hasRowCount(1)
  .containsValues("ABSENT")
  .containsCell(1, "B", "ABSENT");
```

## 4。结论

在 AssertJ 系列的这篇文章中，我们探索了所有与番石榴相关的特性。

所有示例和代码片段的实现都可以在 GitHub 项目中找到。

Next **»**[AssertJ’s Java 8 Features](/web/20220117213330/https://www.baeldung.com/assertJ-java-8-features)**«** Previous[Introduction to AssertJ](/web/20220117213330/https://www.baeldung.com/introduction-to-assertj)