# 在 Java 中比较日期

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-comparing-dates>

## 1.介绍

在本教程中，我们将关注如何使用 [Java 8 日期/时间 API](/web/20220625075455/https://www.baeldung.com/java-8-date-time-intro) 来比较日期。我们将深入研究不同的方法来检查两个日期是否相等，以及如何比较日期。

## 2.比较日期

Java 中表示日期的基本方式是`LocalDate`。让我们考虑两个`LocalDate`对象实例，分别代表 2019 年 8 月 10 日和 2019 年 7 月 1 日:

```
LocalDate firstDate = LocalDate.of(2019, 8, 10);
LocalDate secondDate = LocalDate.of(2019, 7, 1);
```

**我们将利用` isAfter()`、`isBefore()`和`isEqual() methods,`以及`equals()`和`compareTo()`** 来比较两个`LocalDate`对象。

我们使用`isAfter()`方法来检查日期实例是否在另一个指定日期之后。因此，下一个 JUnit 断言将通过:

```
assertThat(firstDate.isAfter(secondDate), is(true));
```

类似地，方法`isBefore()`检查日期实例是否在其他指定日期之前:

```
assertThat(firstDate.isBefore(secondDate), is(false));
```

方法`isEqual()`检查一个日期是否代表本地时间线上与其他指定日期相同的点:

```
assertThat(firstDate.isEqual(firstDate), is(true));
assertThat(firstDate.isEqual(secondDate), is(false));
```

### 2.1.使用`Comparable`界面比较日期

`equals()`方法将给出与`isEqual()`相同的结果，但前提是传递的参数是相同类型的(在本例中是`LocalDate`):

```
assertThat(firstDate.equals(secondDate), is(false));
```

可以使用`isEqual()`方法来比较不同类型的对象，如`JapaneseDate`、`ThaiBuddhistDate`等。

我们可以通过使用由`Comparable`接口定义的`compareTo()` 方法来比较两个日期实例:

```
assertThat(firstDate.compareTo(secondDate), is(1));
assertThat(secondDate.compareTo(firstDate), is(-1));
```

## 3.比较包含时间组件的日期实例

本节将解释如何比较两个`LocalDateTime`实例。`LocalDateTime` 实例包含日期和时间组件。

**与`LocalDate`类似，我们用方法`isAfter()`、`isBefore()`和`isEqual()`** 来比较两个`LocalDateTime`实例。此外，`equals()`和`compareTo()`的使用方式与`LocalDate.`相似

同样，我们可以使用相同的方法来比较两个`ZonedDateTime`实例。让我们比较一下纽约当地时间 8:00 和柏林当地时间 14:00，同一天:

```
ZonedDateTime timeInNewYork = 
  ZonedDateTime.of(2019, 8, 10, 8, 0, 0, 0, ZoneId.of("America/New_York"));
ZonedDateTime timeInBerlin = 
  ZonedDateTime.of(2019, 8, 10, 14, 0, 0, 0, ZoneId.of("Europe/Berlin"));

assertThat(timeInNewYork.isAfter(timeInBerlin), is(false));
assertThat(timeInNewYork.isBefore(timeInBerlin), is(false));
assertThat(timeInNewYork.isEqual(timeInBerlin), is(true));
```

虽然这两个`ZonedDateTime`实例表示同一时刻，但它们并不表示相同的 Java 对象。它们内部有不同的`LocalDateTime`和`ZoneId`字段:

```
assertThat(timeInNewYork.equals(timeInBerlin), is(false)); 
assertThat(timeInNewYork.compareTo(timeInBerlin), is(-1));
```

## 4.附加比较

让我们为稍微复杂一点的比较创建一个简单的实用程序类。

首先，我们将检查`LocalDateTime`和`LocalDate`的实例是否在同一天:

```
public static boolean isSameDay(LocalDateTime timestamp, 
  LocalDate localDateToCompare) {
    return timestamp.toLocalDate().isEqual(localDateToCompare);
}
```

其次，我们将检查`LocalDateTime`的两个实例是否在同一天:

```
public static boolean isSameDay(LocalDateTime timestamp, 
  LocalDateTime timestampToCompare) {
    return timestamp.truncatedTo(DAYS)
      .isEqual(timestampToCompare.truncatedTo(DAYS));
}
```

**`truncatedTo(TemporalUnit)`方法在给定的级别**上截取一个日期，在我们的例子中是一天。

第三，我们可以在一个小时的水平上实现比较:

```
public static boolean isSameHour(LocalDateTime timestamp, 
  LocalDateTime timestampToCompare) {
    return timestamp.truncatedTo(HOURS)
      .isEqual(timestampToCompare.truncatedTo(HOURS));
}
```

最后，以类似的方式，我们可以检查两个`ZonedDateTime`实例是否在同一小时内发生:

```
public static boolean isSameHour(ZonedDateTime zonedTimestamp, 
  ZonedDateTime zonedTimestampToCompare) {
    return zonedTimestamp.truncatedTo(HOURS)
      .isEqual(zonedTimestampToCompare.truncatedTo(HOURS));
}
```

我们可以看到两个`ZonedDateTime`对象实际上在同一小时内发生，即使它们的本地时间不同(分别为 8:30 和 14:00):

```
ZonedDateTime zonedTimestamp = 
  ZonedDateTime.of(2019, 8, 10, 8, 30, 0, 0, ZoneId.of("America/New_York"));
ZonedDateTime zonedTimestampToCompare = 
  ZonedDateTime.of(2019, 8, 10, 14, 0, 0, 0, ZoneId.of("Europe/Berlin"));

assertThat(DateTimeComparisonUtils.
  isSameHour(zonedTimestamp, zonedTimestampToCompare), is(true));
```

## 5.旧 Java 日期 API 中的比较

在 Java 8 之前，我们必须使用`java.util.Date`和`java.util.Calendar`类来操作日期/时间信息。旧的 Java 日期 API 的设计有很多缺陷，比如复杂和不线程安全。`java.util.Date`实例代表一个“瞬间”而不是一个真实的日期。

其中一个解决方案是使用 [Joda Time](/web/20220625075455/https://www.baeldung.com/joda-time) 库。自从 Java 8 发布后，建议[迁移到 Java 8 日期/时间 API](/web/20220625075455/https://www.baeldung.com/migrating-to-java-8-date-time-api) 。

**与`LocalDate`和`LocalDateTime`类似，`java.util.Date`和`java.util.Calendar`对象都有比较两个日期实例**的`after()`、`before()`、`compareTo()`和`equals()`方法。在毫秒级别上，将日期作为瞬间进行比较:

```
Date firstDate = toDate(LocalDateTime.of(2019, 8, 10, 0, 00, 00));
Date secondDate = toDate(LocalDateTime.of(2019, 8, 15, 0, 00, 00));

assertThat(firstDate.after(secondDate), is(false));
assertThat(firstDate.before(secondDate), is(true));
assertThat(firstDate.compareTo(secondDate), is(-1));
assertThat(firstDate.equals(secondDate), is(false));
```

对于更复杂的比较，我们可以使用来自 [Apache Commons Lang](https://web.archive.org/web/20220625075455/https://search.maven.org/search?q=g:org.apache.commons%20AND%20a:commons-lang3) 库的`DateUtils`。这个类包含许多处理`Date`和`Calendar`对象的简便方法:

```
public static boolean isSameDay(Date date, Date dateToCompare) {
    return DateUtils.isSameDay(date, dateToCompare);
}

public static boolean isSameHour(Date date, Date dateToCompare) {
    return DateUtils.truncatedEquals(date, dateToCompare, Calendar.HOUR);
}
```

为了比较来自不同 API 的日期对象，我们应该首先进行适当的转换，然后再进行比较。我们可以在我们的[将日期转换为本地日期或本地日期时间并返回](/web/20220625075455/https://www.baeldung.com/java-date-to-localdate-and-localdatetime)教程中找到更多细节。

## 6.结论

在本文中，我们探索了在 Java 中比较日期实例的不同方法。

Java 8 Date/Time 类具有丰富的 API，可以比较日期，无论有没有时间和时区。我们还看到了如何在日、小时、分钟等粒度上比较日期。

文章中提到的所有代码片段，包括额外的例子，都可以在 GitHub 的[上找到。](https://web.archive.org/web/20220625075455/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-8-datetime)