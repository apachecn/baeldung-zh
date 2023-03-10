# 将 Java 日期转换为 OffsetDateTime

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-convert-date-to-offsetdatetime>

## 1.介绍

在本教程中，我们了解了`Date`和`OffsetDateTime`的区别。我们也学习**如何从一个转换到另一个。**

## 2.`Date`和`OffsetDateTime`的区别

`OffsetDateTime`被引入 JDK 8 作为 [`java.util.Date`](/web/20221128053433/https://www.baeldung.com/java-util-date-sql-date) 的现代替代品。

**`OffsetDateTime`是一个线程安全的类，以纳秒的精度存储日期和时间。另一方面，** `Date`不是线程安全的，它存储时间的精度达到毫秒级。

`OffsetDateTime`是一个基于值的类，这意味着[在比较引用](https://web.archive.org/web/20221128053433/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/doc-files/ValueBased.html)时我们需要使用`equals` 而不是典型的==。

`OffsetDateTime`的`toString`方法的输出是 ISO-8601 格式，而`Date`的`toString`是自定义的非标准格式。

让我们调用两个类中的`toString`来看看区别:

```java
Date: Sat Oct 19 17:12:30 2019
OffsetDateTime: 2019-10-19T17:12:30.174Z
```

`Date`无法存储时区和相应的偏移量。一个`Date`对象唯一包含的是自 1970 年 1 月 1 日 00:00:00 UTC 以来的毫秒数，所以如果我们的时间不是 UTC，我们应该[将时区存储在一个助手类](/web/20221128053433/https://www.baeldung.com/java-set-date-time-zone)中。相反，`OffsetDateTime`将`[ZoneOffset](/web/20221128053433/https://www.baeldung.com/java-zone-offset)`存储在内部。

## 3.将`Date`转换为`OffsetDateTime`

将`Date`转换成`OffsetDateTime`非常简单。如果我们的`Date`是 UTC，我们可以用一个表达式来转换它:

```java
Date date = new Date();
OffsetDateTime offsetDateTime = date.toInstant()
  .atOffset(ZoneOffset.UTC);
```

如果原始的`Date`不在 UTC 中，我们可以提供偏移量(存储在一个 helper 对象中，因为如前所述 Date 类不能存储时区)。

假设我们最初的`Date`是+3:30(德黑兰时间):

```java
int hour = 3;
int minute = 30;
offsetDateTime = date.toInstant()
  .atOffset(ZoneOffset.ofHoursMinutes(hour, minute));
```

`OffsetDateTime`提供了[许多有用的方法](https://web.archive.org/web/20221128053433/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/OffsetDateTime.html)，可以在以后使用。例如，我们可以简单地使用`getDayOfWeek()`、 `getDayOfMonth()`和 `getDayOfYear().` ，用`isAfter`和`isBefore`方法比较两个 OffsetDateTime 对象也非常容易。

最重要的是，**完全避免废弃的`Date`类是一个很好的实践。**

## 4.结论

在本教程中，我们了解了从`Date`转换到`OffsetDateTime`是多么简单。

和往常一样，代码可以在 Github 的[上获得。](https://web.archive.org/web/20221128053433/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-date-operations-2)