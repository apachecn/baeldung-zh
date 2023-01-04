# 在 LocalDateTime 和 ZonedDateTime 之间转换

> 原文::1230]https://web . archive . org/web/202209930061024/https://www . BAE message . com/Java-localdatetime-zoneddatetime

## 1.概观

Java `LocalDateTime` API 表示并操作日期和时间的组合。`ZonedDateTime`是一个不可变的对象，它保存一个精确到纳秒的日期时间值，一个基于的 [ISO 8601 日历系统的时区值，以及一个用于处理不明确的本地日期时间的](https://web.archive.org/web/20221128092927/https://en.wikipedia.org/wiki/ISO_8601) [`ZoneOffSet`](/web/20221128092927/https://www.baeldung.com/java-zone-offset) 。

在本教程中，我们将看到如何从`[LocalDateTime](/web/20221128092927/https://www.baeldung.com/java-8-date-time-intro#3-working-with-localdatetime)`转换到 `[ZonedDateTime](/web/20221128092927/https://www.baeldung.com/java-8-date-time-intro#zonedDateTime)`，反之亦然。

## 2.将`LocalDateTime`转换为`ZonedDateTime`

让我们从将`LocalDateTime`的实例转换为`ZonedDateTime.`开始

### 2.1.使用`atZone()`方法

**来自`LocalDateTime`实例的`atZone()`方法执行到`ZonedDateTime` 的转换，并保持相同的日期时间值**:

```java
LocalDateTime localDateTime = LocalDateTime.of(2022, 1, 1, 0, 30, 22);
ZonedDateTime zonedDateTime = localDateTime.atZone(ZoneId.of("Canada/Atlantic"));

assertEquals(localDateTime.getYear(), zonedDateTime.getYear());
assertEquals(localDateTime.getMonth(), zonedDateTime.getMonth());
assertEquals(localDateTime.getDayOfMonth(), zonedDateTime.getDayOfMonth());
assertEquals(localDateTime.getHour(), zonedDateTime.getHour());
assertEquals(localDateTime.getMinute(), zonedDateTime.getMinute());
assertEquals(localDateTime.getSecond(), zonedDateTime.getSecond());
```

`atZone()`方法接收根据 ISO 8601 日历系统指定时区的`ZoneId`值。

**调用`withZoneSameInstant()`方法使用`ZoneOffSet`时间差**转换成实际的日期时间值:

```java
LocalDateTime localDateTime = LocalDateTime.of(2022, 1, 1, 0, 30, 22);
ZonedDateTime zonedDateTime = localDateTime.atZone(ZoneId.of("Africa/Lagos")).withZoneSameInstant(ZoneId.of("Canada/Atlantic"));

assertEquals("2021-12-31T19:30:22-04:00[Canada/Atlantic]", zonedDateTime.toString());
assertEquals("-04:00", zonedDateTime.getOffset().toString());
```

我们可以通过调用静态的 `ZoneId.getAvailableZoneIds()` 方法来获得可用的`ZoneId`。该方法返回一组所有可用的基于区域的 id，如`String`，我们可以从中选择以创建一个`ZoneId`对象。

此外，使用`atZone()` 的转换还带有一个`ZoneOffSet`值，该值提供了`ZonedDateTime`对象和 UTC (GMT)之间的时差(上例中为`-04:00` )。

### 2.2.使用`ZonedDateTime.of()`方法

`ZonedDateTime`类还提供了一个静态的`of()`方法来创建一个`ZonedDateTime` 对象。该方法接受`LocalDateTime`和`ZoneId`的实例作为参数，并返回一个`ZonedDateTime`对象:

```java
LocalDateTime localDateTime = LocalDateTime.of(2022, 11, 5, 7, 30, 22);
ZonedDateTime zonedDateTime = ZonedDateTime.of(localDateTime, ZoneId.of("Africa/Accra")).withZoneSameInstant(ZoneId.of("Africa/Lagos"));

assertEquals("2022-11-05T08:30:22+01:00[Africa/Lagos]", zonedDateTime.toString()); 
assertEquals(localDateTime.getYear(), zonedDateTime.getYear());
```

在这种情况下，正如我们前面看到的，我们可以通过调用`withZoneSameInstant()`方法获得给定区域的实际日期时间值。

### 2.3.使用`ZonedDateTime.ofInstant()`方法

我们也可以结合使用一个`[ZoneOffSet](/web/20221128092927/https://www.baeldung.com/java-zone-offset)`对象和`LocalDateTime`来创建一个 `ZonedDateTime`对象。

静态的`ofInstant()`方法接受`LocalDateTime`、`ZoneOffSet`和`ZoneId`对象作为参数:

```java
LocalDateTime localDateTime = LocalDateTime.of(2022, 1, 5, 17, 30, 22);
ZoneId zoneId = ZoneId.of("Africa/Lagos");
ZoneOffset zoneOffset = zoneId.getRules().getOffset(localDateTime);
ZonedDateTime zonedDateTime = ZonedDateTime.ofInstant(localDateTime, zoneOffset, zoneId);
```

```java
assertEquals("2022-01-05T17:30:22+01:00[Africa/Lagos]", zonedDateTime.toString());
```

**`ZonedDateTime`对象是从通过组合`LocalDateTime`和`ZoneOffSet`对象**隐式形成的`Instant`对象创建的。

### 2.4.使用`ZonedDateTime.ofLocal()`方法

静态的`ofLocal()`方法从`LocalDateTime`对象创建一个`ZonedDateTime`，首选的`ZoneOffSet`对象作为参数传递:

```java
LocalDateTime localDateTime = LocalDateTime.of(2022, 8 , 25, 8, 35, 22);
ZoneId zoneId = ZoneId.of("Africa/Lagos");
ZoneOffset zoneOffset = zoneId.getRules().getOffset(localDateTime);
ZonedDateTime zonedDateTime = ZonedDateTime.ofLocal(localDateTime, zoneId, zoneOffset);
```

```java
assertEquals("2022-08-25T08:35:22+01:00[Africa/Lagos]", zonedDateTime.toString());
```

**通常，本地日期时间**只存在一个有效的偏移。当时间重叠发生时，将会有两个有效的偏移。

如果作为参数传递的首选`ZoneOffset`是有效的偏移量之一，则使用它。否则，转换会保持之前的有效偏移。

### 2.5.使用`ZonedDateTime.ofStrict()`方法

类似地，静态`ofStrict()`方法通过严格验证`LocalDateTime`、`ZoneOffSet`和`ZoneID`参数的组合来返回一个`ZonedDateTime`对象:

```java
LocalDateTime localDateTime = LocalDateTime.of(2022, 12, 25, 6, 18, 2);
ZoneId zoneId = ZoneId.of("Asia/Tokyo");
ZoneOffset zoneOffset = zoneId.getRules().getOffset(localDateTime);
ZonedDateTime zonedDateTime = ZonedDateTime.ofStrict(localDateTime, zoneOffset, zoneId);
```

```java
assertEquals("2002-12-25T06:18:02+09:00[Asia/Tokyo]", zonedDateTime.toString());
```

如果我们提供了无效的参数组合，该方法将抛出一个`DateTimeException`:

```java
zoneId = ZoneId.of("Asia/Tokyo");
zoneOffset = ZoneOffset.UTC;
assertThrows(DateTimeException.class, () -> ZonedDateTime.ofStrict(localDateTime, zoneOffset, zoneId));
```

上面的例子显示，当我们试图使用来自`Asia/Tokyo`的`ZoneId`和代表默认 UTC ( `GMT + 0`)的`ZoneOffSet`值的组合来创建一个`ZonedDateTime`对象时，会抛出一个异常。

## 3.将`ZonedDateTime` 转换为`LocalDateTime`

一个`ZonedDateTime`对象维护三个不同的对象:一个`LocalDateTime`、一个`ZoneId`和一个`ZoneOffset`。

我们可以使用`toLocalDateTime()`方法将`ZonedDateTime`的实例转换为`LocalDateTime` :

```java
ZonedDateTime zonedDateTime = ZonedDateTime.of(2011, 2, 12, 6, 14, 1, 58086000, ZoneId.of("Asia/Tokyo"));
LocalDateTime localDateTime = zonedDateTime.toLocalDateTime();

assertEquals("2011-02-12T06:14:01.058086+09:00[Asia/Tokyo]", zonedDateTime.toString());
```

该方法检索作为`ZonedDateTime`属性的一部分存储的`LocalDateTime`对象。

## 4.结论

在本文中，我们学习了如何将`LocalDateTime`的实例转换为`ZonedDateTime`，反之亦然。

与往常一样，GitHub 上的[提供了示例的源代码。](https://web.archive.org/web/20221128092927/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-datetime-conversion)