# 用 Java 语言从日期中减去天数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-subtract-days-from-date>

## 1.概观

在本教程中，我们将发现用 Java 从日期对象中减去天数的各种方法。

我们将从使用 Java 8 引入的[日期时间 API](/web/20221208143856/https://www.baeldung.com/java-8-date-time-intro) 开始。在这之后，我们将通过使用`java.util`包中的类来学习如何做，最后，我们将在 [Joda-Time](/web/20221208143856/https://www.baeldung.com/joda-time) 库的帮助下完成同样的事情。

## 2.`java.time.LocalDateTime`

Java 8 中引入的日期/时间 API 是目前最可行的日期和时间计算选项。

让我们看看如何从 Java 8 的`java.util.LocalDateTime` 对象`:`中减去天数

```java
@Test
public void givenLocalDate_whenSubtractingFiveDays_dateIsChangedCorrectly() {
    LocalDateTime localDateTime = LocalDateTime.of(2022, 4, 20, 0, 0);

    localDateTime = localDateTime.minusDays(5);

    assertEquals(15, localDateTime.getDayOfMonth());
    assertEquals(4, localDateTime.getMonthValue());
    assertEquals(2022, localDateTime.getYear());
}
```

## 3.`java.util.Calendar`

来自`java.util` 的`Date`和`Calendar` 是 Java 8 之前最常用的日期操作实用程序类。

让我们用`java.util.Calendar`从一个日期中减去五天:

```java
@Test
public void givenCalendarDate_whenSubtractingFiveDays_dateIsChangedCorrectly() {
    Calendar calendar = Calendar.getInstance();
    calendar.set(2022, Calendar.APRIL, 20);

    calendar.add(Calendar.DATE, -5);

    assertEquals(15, calendar.get(Calendar.DAY_OF_MONTH));
    assertEquals(Calendar.APRIL, calendar.get(Calendar.MONTH));
    assertEquals(2022, calendar.get(Calendar.YEAR));
}
```

我们在使用它们时应该小心，因为它们有一些设计缺陷，而且不是线程安全的。

在关于[迁移到新的 Java 8 日期时间 API](https://web.archive.org/web/20221208143856/https://baeldung.com/migrating-to-java-8-date-time-api) 的文章中，我们可以读到更多关于与遗留代码的交互以及两个 API 之间的差异。

## 4.Joda-Time

对于日期和时间处理，我们可以使用 Joda-Time 作为 Java 初始解决方案的更好替代方案。该库提供了更直观的 API、多个日历系统、线程安全和不可变对象。

为了使用 [Joda-Time](https://web.archive.org/web/20221208143856/https://search.maven.org/search?q=g:joda-time%20AND%20a:joda-time) ，我们需要将它作为一个依赖项包含在`pom.xml`文件中:

```java
<dependency>
    <groupId>joda-time</groupId>
    <artifactId>joda-time</artifactId>
    <version>2.10.14</version>
</dependency>
```

让我们从 Joda-Time 的`DateTime`对象中减去五天:

```java
@Test
public void givenJodaDateTime_whenSubtractingFiveDays_dateIsChangedCorrectly() {
    DateTime dateTime = new DateTime(2022, 4, 20, 12, 0, 0);

    dateTime = dateTime.minusDays(5);

    assertEquals(15, dateTime.getDayOfMonth());
    assertEquals(4, dateTime.getMonthOfYear());
    assertEquals(2022, dateTime.getYear());
} 
```

Joda-Time 是遗留代码的一个很好的解决方案。**然而，该项目正式“结束”，库的作者建议迁移到 Java 8 的日期/时间 API。**

## 5.结论

在这篇短文中，我们探索了几种从 date 对象中减去天数的方法。

我们已经使用`java.time.LocalDateTime `和 Java 8 之前的解决方案:`java.util.Calendar` 和 Joda-Time 库实现了这一点。

和往常一样，源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221208143856/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-date-operations-2)