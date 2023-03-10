# AssertJ 的 Java 8 特性

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/assertJ-java-8-features>

[This article is part of a series:](javascript:void(0);)[• Introduction to AssertJ](/web/20220123160032/https://www.baeldung.com/introduction-to-assertj)
[• AssertJ for Guava](/web/20220123160032/https://www.baeldung.com/assertJ-for-guava)
• AssertJ’s Java 8 Features (current article)[• Custom Assertions with AssertJ](/web/20220123160032/https://www.baeldung.com/assertj-custom-assertion)

## 1。概述

本文主要关注 [AssertJ](https://web.archive.org/web/20220123160032/https://joel-costigliola.github.io/assertj/) 的 Java8 相关特性，是该系列的第三篇文章。

如果你正在寻找关于它的主要特征的一般信息，可以看看 AssertJ 系列的第一篇文章[简介](/web/20220123160032/https://www.baeldung.com/introduction-to-assertj)，然后看看《番石榴的 T2》。

## 2。Maven 依赖关系

从版本 3.5.1 开始，Java 8 的支持包含在主 AssertJ 核心模块中。为了使用该模块，您需要在您的`pom.xml`文件中包含以下部分:

```java
<dependency>
    <groupId>org.assertj</groupId>
    <artifactId>assertj-core</artifactId>
    <version>3.5.1</version>
    <scope>test</scope>
</dependency>
```

这种依赖只包括基本的 Java 断言。如果您想要使用高级断言，您将需要单独添加额外的模块。

最新的核心版本可以在[这里](https://web.archive.org/web/20220123160032/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22assertj-core%22)找到。

## 3。Java 8 特性

AssertJ 通过为 Java 8 类型提供特殊的助手方法和新的断言来利用 Java 8 的特性。

### 3.1。`Optional`断言

让我们创建一个简单的`Optional`实例:

```java
Optional<String> givenOptional = Optional.of("something");
```

我们现在可以很容易地检查一个`Optional`是否包含一些值，以及包含的值是什么:

```java
assertThat(givenOptional)
  .isPresent()
  .hasValue("something");
```

### 3.2。`Predicate`断言

让我们通过检查`String`的长度来创建一个简单的`Predicate`实例:

```java
Predicate<String> predicate = s -> s.length() > 4;
```

现在，您可以轻松检查哪些`String`被`Predicate:`拒绝或接受

```java
assertThat(predicate)
  .accepts("aaaaa", "bbbbb")
  .rejects("a", "b")
  .acceptsAll(asList("aaaaa", "bbbbb"))
  .rejectsAll(asList("a", "b"));
```

### 3.3。`LocalDate`断言

让我们从定义两个`LocalDate`对象开始:

```java
LocalDate givenLocalDate = LocalDate.of(2016, 7, 8);
LocalDate todayDate = LocalDate.now();
```

现在，您可以轻松检查给定日期是在给定日期之前/之后，还是在今天:

```java
assertThat(givenLocalDate)
  .isBefore(LocalDate.of(2020, 7, 8))
  .isAfterOrEqualTo(LocalDate.of(1989, 7, 8));

assertThat(todayDate)
  .isAfter(LocalDate.of(1989, 7, 8))
  .isToday();
```

### 3.4。`LocalDateTime`断言

`LocalDateTime`断言的工作方式与`LocalDate`的`,`类似，但不共享`isToday` 方法。

让我们创建一个示例`LocalDateTime`对象:

```java
LocalDateTime givenLocalDate = LocalDateTime.of(2016, 7, 8, 12, 0);
```

现在您可以检查:

```java
assertThat(givenLocalDate)
  .isBefore(LocalDateTime.of(2020, 7, 8, 11, 2));
```

### 3.5。`LocalTime`断言

`LocalTime`断言的工作方式与其他 `java.util.time.*`断言类似，但是它们有一个唯一的方法:`hasSameHourAs.`

让我们创建一个示例`LocalTime`对象:

```java
LocalTime givenLocalTime = LocalTime.of(12, 15);
```

现在你可以断言:

```java
assertThat(givenLocalTime)
  .isAfter(LocalTime.of(1, 0))
  .hasSameHourAs(LocalTime.of(12, 0));
```

### 3.6。`FlatExtracting` 帮手法

`FlatExtracting`是一个特殊的实用方法，它利用 Java 8 的 lambdas 从 `Iterable`元素`.`中提取属性

让我们用`LocalDate`对象创建一个简单的`List`:

```java
List<LocalDate> givenList = asList(ofYearDay(2016, 5), ofYearDay(2015, 6));
```

现在我们可以很容易地检查这个`List`是否包含至少一个 2015 年的`LocalDate`对象:

```java
assertThat(givenList)
  .flatExtracting(LocalDate::getYear)
  .contains(2015);
```

`flatExtracting`方法并不局限于字段提取。我们总能为它提供任何功能:

```java
assertThat(givenList)
  .flatExtracting(LocalDate::isLeapYear)
  .contains(true);
```

或者甚至:

```java
assertThat(givenList)
  .flatExtracting(Object::getClass)
  .contains(LocalDate.class);
```

您也可以一次提取多个属性:

```java
assertThat(givenList)
  .flatExtracting(LocalDate::getYear, LocalDate::getDayOfMonth)
  .contains(2015, 6);
```

### 3.7。`Satisfies`帮手法

`Satisfies`方法允许您快速检查一个对象是否满足所有提供的断言。

让我们创建一个示例`String`实例:

```java
String givenString = "someString";
```

现在我们可以将断言作为 lambda 主体提供:

```java
assertThat(givenString)
  .satisfies(s -> {
    assertThat(s).isNotEmpty();
    assertThat(s).hasSize(10);
  });
```

### 3.8。*hasonlyoneelementsatisfacting*助手方法

`HasOnlyOneElement` helper 方法允许检查一个`Iterable`实例是否只包含一个满足所提供断言的元素。

让我们创建一个例子`List:`

```java
List<String> givenList = Arrays.asList("");
```

现在你可以断言:

```java
assertThat(givenList)
  .hasOnlyOneElementSatisfying(s -> assertThat(s).isEmpty());
```

### 3.9。`Matches` 帮手法

`Matches`助手方法允许检查给定的对象是否匹配给定的`Predicate`函数。

让我们空出一个`String:`

```java
String emptyString = "";
```

现在我们可以通过提供适当的`Predicate` lambda 函数来检查它的状态:

```java
assertThat(emptyString)
  .matches(String::isEmpty);
```

## 4。结论

在 AssertJ 系列的最后一篇文章中，我们探索了 AssertJ Java 8 的所有高级特性，这是本系列的最后一篇文章。

所有示例和代码片段的实现都可以在 [GitHub 项目](https://web.archive.org/web/20220123160032/https://github.com/eugenp/tutorials/tree/master/testing-modules/assertion-libraries)中找到。

Next **»**[Custom Assertions with AssertJ](/web/20220123160032/https://www.baeldung.com/assertj-custom-assertion)**«** Previous[AssertJ for Guava](/web/20220123160032/https://www.baeldung.com/assertJ-for-guava)