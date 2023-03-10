# ZonedDateTime 和 OffsetDateTime 之间的差异

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-zoneddatetime-offsetdatetime>

## 1.概观

`ZonedDateTime`和`OffsetDateTime`是 [Java 8 `DateTime` API](/web/20220826203827/https://www.baeldung.com/java-8-date-time-intro) `.`中非常受欢迎的类，此外，**都在时间线上存储一个瞬间，精度高达纳秒`.`** ，起初，在它们之间做出选择可能会令人困惑。

在这个快速教程中，我们将看看`ZonedDateTime`和`OffsetDateTime`之间的区别。

## 2.`ZonedDateTime`

在 ISO-8601 日历系统中， `ZonedDateTime`是带有时区的日期时间的不可变表示，例如`2007-12-03T10:15:30+01:00 Europe/Pari` s. **它保存相当于三个独立对象的状态:a `LocalDateTime`、a `ZoneId`和已解析的 [`ZoneOffset`](/web/20220826203827/https://www.baeldung.com/java-zone-offset) 。**

这里，`ZoneId`决定了偏移如何以及何时改变。因此，偏移量不能自由设置，因为区域控制哪些偏移量有效。

为了获得特定区域的当前`ZonedDateTime`,我们将使用:

```java
ZoneId zone = ZoneId.of("Europe/Berlin");
ZonedDateTime zonedDateTime = ZonedDateTime.now(zone);
```

`ZonedDateTime`类还提供了将给定日期从一个时区转换到另一个时区的内置方法:

```java
ZonedDateTime destZonedDateTime = sourceZonedDateTime.withZoneSameInstant(destZoneId);
```

最后，它完全支持夏令时，并处理夏令时调整。当我们想要显示特定时区的日期-时间字段时，这通常会很方便。

## 3.`OffsetDateTime`

在 ISO-8601 日历系统中， `OffsetDateTime`是日期时间的不可变表示，与 UTC/Greenwich 有一个偏移量，例如`2007-12-03T10:15:30+01:00`。换句话说，**存储了** **所有的日期和时间字段，精确到纳秒，以及 GMT/UTC** 的偏移量。

让我们从 GMT/UTC 得到当前的`OffsetDateTime `,有两个小时的时差:

```java
ZoneOffset zoneOffSet= ZoneOffset.of("+02:00");
OffsetDateTime offsetDateTime = OffsetDateTime.now(zoneOffSet);
```

## 4.主要区别

首先，用完整的时区信息直接比较两个日期是没有意义的(没有转换)。因此，**我们应该总是倾向于在数据库中存储 `OffsetDateTime` 而不是`ZonedDateTime`，因为具有本地时间偏移的日期总是表示相同的时刻。**

此外，与`ZonedDateTime`不同，在存储`OffsetDateTime`的列上添加索引不会改变日期的含义。

让我们快速总结一下关键区别。

**:**

*   存储所有日期和时间字段，精确到纳秒，存储一个时区，并使用时区偏移量来处理不明确的本地日期时间
*   不能自由设置偏移，因为区域控制有效的偏移值
*   完全支持夏令时，并处理夏令时调整
*   在特定于用户的时区中显示日期时间字段非常方便

`**OffsetDateTime**`**:T2**

*   存储所有日期和时间字段，精度为纳秒，以及相对于 GMT/UTC 的偏移量(无时区信息)
*   应该用于在数据库中存储日期或通过网络进行通信

## 5.结论

在本教程中，我们讨论了`ZonedDateTime`和`OffsetDateTime`之间的区别。

和往常一样，完整的源代码可以在 Github 的[上找到。](https://web.archive.org/web/20220826203827/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-8-datetime)