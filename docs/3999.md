# ZoneOffset in Java

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-zone-offset>

## 1.介绍

在我们的世界里，每个国家都遵循一定的时区。这些时区对于方便有效地表达时间至关重要。然而，由于夏令时等变量的影响，时区有时并不明确。

此外，在我们的代码中表示这些时区时，事情可能会变得混乱。过去，Java 提供了多个类，如`Date`、`Time`和`DateTime`来处理时区问题。

然而，新的 Java 版本已经提出了更有用和更有表现力的类，比如用于管理时区的`ZoneId`和`ZoneOffset`。

在这篇文章中，**我们将讨论`ZoneId`和`ZoneOffset`以及相关的`DateTime `类**。

在我们的[上一篇文章](/web/20221129002943/https://www.baeldung.com/java-8-date-time-intro)中，我们还可以读到 Java 8 中引入的一组新的`DateTime`类。

## 2.`ZoneId`和`ZoneOffset`

随着 [JSR-310](https://web.archive.org/web/20221129002943/https://jcp.org/en/jsr/detail?id=310) 的出现，增加了一些管理日期、时间和时区的有用 API。作为此次更新的一部分，还添加了`ZoneId`和`ZoneOffset` 类。

### 2.1.`ZoneId`

如上所述， **`ZoneId`是时区**的表示，如“`Europe/Paris`”。

`ZoneId`有 2 个实现。首先，与 GMT/UTC 相比有一个固定的时差。第二，作为一个地理区域，它有一套用 GMT/UTC 计算时差的规则。

让我们为德国柏林创造一个`ZoneId` :

```
ZoneId zone = ZoneId.of("Europe/Berlin");
```

### 2.2.`ZoneOffset`

**`ZoneOffset` 扩展`ZoneId`** **用 GMT/UTC** 定义 **当前时区的固定时差，如+02:00。**

这意味着这个数字代表固定的小时和分钟，代表当前时区和 GMT/UTC 之间的时差:

```
LocalDateTime now = LocalDateTime.now();
ZoneId zone = ZoneId.of("Europe/Berlin");
ZoneOffset zoneOffSet = zone.getRules().getOffset(now);
```

**如果一个国家有 2 种不同的补偿——在夏季和冬季，同一地区将有 2 种不同的`ZoneOffset `实施，因此需要指定一个`LocalDateTime`** 。

## 3.`DateTime`类别

接下来让我们讨论一些实际上利用了`ZoneId`和`ZoneOffset`的`DateTime`类。

### 3.1.`ZonedDateTime`

`ZonedDateTime`是 ISO-8601 日历系统中带有时区的日期时间的不可变表示，例如`2007-12-03T10:15:30+01:00 Europe/Pari` s. **A `ZonedDateTime`持有相当于三个独立对象的状态，a `LocalDateTime`，a `ZoneId`和已解析的`ZoneOffset`。**

这个类存储所有的日期和时间字段，精确到纳秒，还存储一个时区，用一个`ZoneOffset`来处理不明确的本地日期时间。例如，`ZonedDateTime` 可以存储值“欧洲/巴黎时区 2007 年 10 月 2 日 13:45.30.123456789 +02:00”。

让我们获取前一个区域的当前`ZonedDateTime` :

```
ZoneId zone = ZoneId.of("Europe/Berlin");
ZonedDateTime date = ZonedDateTime.now(zone);
```

`ZonedDateTime also `提供内置函数，将给定日期从一个时区转换到另一个时区:

```
ZonedDateTime destDate = sourceDate.withZoneSameInstant(destZoneId);
```

### 3.2.`OffsetDateTime`

`OffsetDateTime`是 ISO-8601 日历系统中带有偏移量的日期时间的不可变表示，比如`2007-12-03T10:15:30+01:00`。

**这个类存储所有的日期和时间字段，精确到纳秒，以及 GMT/UTC** 的偏移量。例如，`OffsetDateTime` 可以存储值“2007 年 10 月 2 日 13:45.30.123456789 +02:00”。

让我们从 GMT/UTC 得到当前的`OffsetDateTime `,偏移 2 小时:

```
ZoneOffset zoneOffSet= ZoneOffset.of("+02:00");
OffsetDateTime date = OffsetDateTime.now(zoneOffSet);
```

### 3.3.`OffsetTime`

`OffsetTime`是一个不可变的日期-时间对象，表示 ISO-8601 日历系统中的时间，通常被视为小时-分钟-秒偏移，例如`10:15:30+01:00`。

这个类存储所有的时间域，精确到纳秒，还有一个时区偏移量。例如，`OffsetTime` 可以存储值“13:45.30.123456789+02:00”。

让我们用 2 小时的偏移得到当前的`OffsetTime` :

```
ZoneOffset zoneOffSet = ZoneOffset.of("+02:00");
OffsetTime time = OffsetTime.now(zoneOffSet);
```

## 4.结论

回到焦点，`ZoneOffset`是根据 GMT/UTC 和给定时间之间的差异表示的时区。这是一种表示时区的简便方法，尽管也有其他表示方法。

而且，`ZoneId`和`ZoneOffset`不仅可以独立使用，还可以被某些`DateTime` Java 类使用，比如`ZonedDateTime`、`OffsetDateTime`和`OffsetTime`。

像往常一样，代码可以在我们的 [GitHub 库](https://web.archive.org/web/20221129002943/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-8-datetime)中获得。