# 如何使用 Java 获得一天的开始和结束

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-day-start-end>

## 1.概观

在这个简短的教程中，**我们将学习如何用 Java** 获得一天的开始和结束，使用简单、直接的例子来描述不同的场景。

我们将使用 [Java 的 8 日期/时间 API](https://web.archive.org/web/20221126224722/http://www.oracle.com/technetwork/articles/java/jf14-date-time-2125367.html) 来构建这些例子。

如果你想在继续之前多读一点关于 Java 8 的日期和时间库，你可以从这里开始。

## 2.从一个`LocalDate`物体

首先，让我们看看如何将一天的开始或结束作为一个`[LocalDate](https://web.archive.org/web/20221126224722/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/LocalDate.html)`对象给我们，例如:

```java
LocalDate localDate = LocalDate.parse("2018-06-23");
```

### 2.1.`atStartOfDay()`

获得代表某一天开始的`[LocalDateTime](https://web.archive.org/web/20221126224722/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/LocalDateTime.html)`的最简单方法是使用`atStartOfDay()`方法:

```java
LocalDateTime startOfDay = localDate.atStartOfDay();
```

这个方法是重载的，因此如果我们想从中获得一个 [`ZonedDateTime`](https://web.archive.org/web/20221126224722/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/ZonedDateTime.html) ，我们可以通过指定`[ZoneId](https://web.archive.org/web/20221126224722/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/ZoneId.html)`来实现:

```java
ZonedDateTime startOfDay = localDate.atStartOfDay(ZoneId.of("Europe/Paris"));
```

### 2.2.`of()`

我们可以实现相同结果的另一种方法是使用`of()`方法，提供一个`LocalDate`和一个 [`LocalTime`](https://web.archive.org/web/20221126224722/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/LocalTime.html) 的静态字段:

```java
LocalDateTime startOfDay = LocalDateTime.of(localDate, LocalTime.MIDNIGHT);
```

`LocalTime`提供以下静态字段:`MIDNIGHT`(00:00)`MIN`(00:00)`NOON`(12:00)`MAX`(23:59:59.999999999)。

因此，如果我们想得到一天的结束时间，我们可以使用`MAX`值。

让我们试试看，但是用不同的方法。

### 2.3.`atTime()`

这个方法是重载的，允许我们使用小时、分钟、秒甚至纳秒来指定所需的时间。

在这种情况下，无论如何，我们将使用`LocalTime`的`MAX`字段作为参数来获取给定一天的最后时刻:

```java
LocalDateTime startOfDay = localDate.atTime(LocalTime.MAX);
```

### 2.4.`atDate()`

这个例子与前面的例子非常相似，但是这一次，我们将使用一个`LocalTime`对象的`atDate()`方法，传递`LocalDate`作为参数:

```java
LocalDateTime endOfDate = LocalTime.MAX.atDate(localDate);
```

## 3.从一个`LocalDateTime`物体

不言而喻，我们可以从中获得`LocalDate`，然后使用第 2 节中的任何方法从中获得一天的结束或开始:

```java
LocalDateTime localDateTime = LocalDateTime
  .parse("2018-06-23T05:55:55");
LocalDateTime endOfDate = localDateTime
  .toLocalDate().atTime(LocalTime.MAX);
```

但是在这一节中，我们将分析另一种方法来直接从另一个给定的`LocalDateTime`对象中获取其时间段设置为一天的开始或结束的对象。

### 3.1.`with()`

所有实现了`[Temporal interface](https://web.archive.org/web/20221126224722/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/temporal/Temporal.html)`的类都可以使用这个方法。

在这种情况下，我们将使用该方法的签名，它将一个`[TemporalField](https://web.archive.org/web/20221126224722/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/temporal/TemporalField.html)`(特别是一个`[ChronoField Enum](https://web.archive.org/web/20221126224722/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/temporal/ChronoField.html)`值)和一个`long `参数作为该字段的新值:

```java
LocalDateTime endOfDate = localDateTime.with(ChronoField.NANO_OF_DAY, LocalTime.MAX.toNanoOfDay());
```

## 4.从一个`ZonedDateTime`物体

如果给我们一个`ZonedDateTime,`，我们可以使用`with()`方法，因为它也实现了`Temporal interface`:

```java
ZonedDateTime startofDay = zonedDateTime.with(ChronoField.HOUR_OF_DAY, 0);
```

## 5.结论

总的来说，我们已经针对许多不同的情况分析了在 Java 中获得一天的开始和结束的许多不同的方法。

最后，我们了解了 Java 的 8 个日期和时间库类，并熟悉了它的许多类和接口。

和往常一样，所有的例子都可以在我们的 GitHub 库中找到。