# DateTimeFormatter 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-datetimeformatter>

## 1.概观

在本教程中，我们将回顾 Java 8 `DateTimeFormatter` 类及其格式化模式`.` 我们还将讨论该类可能的用例。

我们可以使用`DateTimeFormatter`以预定义或用户定义的模式统一格式化应用程序中的日期和时间。

## 2.`DateTimeFormatter`带有预定义的实例

**`DateTimeFormatter`带有多个[预定义的日期/时间格式](https://web.archive.org/web/20221027035204/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/format/DateTimeFormatter.html#ISO_LOCAL_DATE)** 遵循 ISO 和 RFC 标准。例如，我们可以使用`ISO_LOCAL_DATE`实例来解析日期，比如“2018-03-09”:

```java
DateTimeFormatter.ISO_LOCAL_DATE.format(LocalDate.of(2018, 3, 9));
```

要解析一个有偏移量的日期，我们可以使用`ISO_OFFSET_DATE`得到一个类似‘2018-03-09-03:00’的输出:

```java
DateTimeFormatter.ISO_OFFSET_DATE.format(LocalDate.of(2018, 3, 9).atStartOfDay(ZoneId.of("UTC-3")));
```

**`DateTimeFormatter`类的大多数预定义实例都集中在 ISO-8601 标准上。** ISO-8601 是日期和时间格式的国际标准。

然而，有一个不同的预定义实例，它解析 IETF 发布的 RFC-1123，对互联网主机的要求:

```java
DateTimeFormatter.RFC_1123_DATE_TIME.format(LocalDate.of(2018, 3, 9).atStartOfDay(ZoneId.of("UTC-3")));
```

此代码片段生成'`Fri, 9 Mar 2018 00:00:00 -0300.`'

有时，我们不得不将收到的数据处理成已知格式的`String`。为此，我们可以利用`parse()`方法:

```java
LocalDate.from(DateTimeFormatter.ISO_LOCAL_DATE.parse("2018-03-09")).plusDays(3);
```

这个代码片段的结果是 2018 年 3 月 12 日的一个`LocalDate`表示。

## 3.`DateTimeFormatter`同`FormatStyle`

有时我们可能希望以人类可读的方式打印日期。

在这种情况下，我们可以将`java.time.format.FormatStyle`枚举(完整、长、中、短)值与我们的`DateTimeFormatter`一起使用:

```java
LocalDate anotherSummerDay = LocalDate.of(2016, 8, 23);
System.out.println(DateTimeFormatter.ofLocalizedDate(FormatStyle.FULL).format(anotherSummerDay));
System.out.println(DateTimeFormatter.ofLocalizedDate(FormatStyle.LONG).format(anotherSummerDay));
System.out.println(DateTimeFormatter.ofLocalizedDate(FormatStyle.MEDIUM).format(anotherSummerDay));
System.out.println(DateTimeFormatter.ofLocalizedDate(FormatStyle.SHORT).format(anotherSummerDay));
```

相同日期的这些不同格式样式的输出是:

```java
Tuesday, August 23, 2016
August 23, 2016
Aug 23, 2016
8/23/16
```

我们也可以使用预定义的日期和时间格式。要将`FormatStyle`与时间一起使用，我们必须使用`ZonedDateTime`实例，否则，将会抛出一个`DateTimeException`:

```java
LocalDate anotherSummerDay = LocalDate.of(2016, 8, 23);
LocalTime anotherTime = LocalTime.of(13, 12, 45);
ZonedDateTime zonedDateTime = ZonedDateTime.of(anotherSummerDay, anotherTime, ZoneId.of("Europe/Helsinki"));
System.out.println(
  DateTimeFormatter.ofLocalizedDateTime(FormatStyle.FULL)
  .format(zonedDateTime));
System.out.println(
  DateTimeFormatter.ofLocalizedDateTime(FormatStyle.LONG)
  .format(zonedDateTime));
System.out.println(
  DateTimeFormatter.ofLocalizedDateTime(FormatStyle.MEDIUM)
  .format(zonedDateTime));
System.out.println(
  DateTimeFormatter.ofLocalizedDateTime(FormatStyle.SHORT)
  .format(zonedDateTime));
```

注意我们这次用的是`DateTimeFormatter`的`ofLocalizedDateTime()`方法。

我们得到的输出是:

```java
Tuesday, August 23, 2016 1:12:45 PM EEST
August 23, 2016 1:12:45 PM EEST
Aug 23, 2016 1:12:45 PM
8/23/16 1:12 PM
```

例如，我们还可以使用`FormatStyle`解析日期时间`String,`并将其转换为`ZonedDateTime`。

然后，我们可以使用解析后的值来操作日期和时间变量:

```java
ZonedDateTime dateTime = ZonedDateTime.from(
  DateTimeFormatter.ofLocalizedDateTime(FormatStyle.FULL)
    .parse("Tuesday, August 23, 2016 1:12:45 PM EET"));
System.out.println(dateTime.plusHours(9));
```

这个片段的输出是“2016-08-23t 22:12:45+03:00[欧洲/布加勒斯特]。”请注意，时间已经更改为“22:12:45”

## 4.`DateTimeFormatter`带自定义格式

**预定义和内置的格式器和样式可以覆盖很多情况**。然而，有时我们需要将日期和时间的格式稍微改变一下。这就是自定义格式模式发挥作用的时候了。

### 4.1.`DateTimeFormatter`为日期

假设我们想要使用 31.12.2018 这样的常规欧洲格式呈现一个`java.time.LocalDate`对象。为此，我们可以调用工厂方法`DateTimeFormatter`。`ofPattern(“dd.MM.yyyy”).`

这将创建一个合适的`DateTimeFormatter`实例，我们可以用它来格式化我们的日期:

```java
String europeanDatePattern = "dd.MM.yyyy";
DateTimeFormatter europeanDateFormatter = DateTimeFormatter.ofPattern(europeanDatePattern);
System.out.println(europeanDateFormatter.format(LocalDate.of(2016, 7, 31)));
```

该代码片段的输出将是“31.07.2016”

我们可以使用许多不同的字母样式来创建符合我们需求的日期格式:

```java
 Symbol  Meaning                     Presentation      Examples
  ------  -------                     ------------      -------
   u       year                        year              2004; 04
   y       year-of-era                 year              2004; 04
   M/L     month-of-year               number/text       7; 07; Jul; July; J
   d       day-of-month                number            10
```

这是从[官方 Java 文档](https://web.archive.org/web/20221027035204/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/format/DateTimeFormatter.html)到`DateTimeFormatter`类的摘录。

**模式格式中的字母数量很重要**。

如果我们对月份使用两个字母的模式，我们将得到一个两位数的月份表示。如果月份数小于 10，则用零填充。当我们不需要上面提到的零填充时，我们可以使用单字母模式“M”，它将一月显示为“1”

如果我们碰巧用四个字母的模式表示这个月，“MMMM”，那么我们将得到一个“完整形式”的表示。在我们的例子中，应该是“七月”五个字母的模式“MMMMM”将使格式化程序使用“窄格式”。在我们的例子中，将使用“J”。

同样，自定义格式模式也可用于解析保存日期的字符串:

```java
DateTimeFormatter europeanDateFormatter = DateTimeFormatter.ofPattern("dd.MM.yyyy");
System.out.println(LocalDate.from(europeanDateFormatter.parse("15.08.2014")).isLeapYear());
```

这段代码检查日期“`15.08.2014`”是否是闰年，如果不是。

### 4.2.`DateTimeFormatter`为时间

也有可用于时间模式的模式字母:

```java
 Symbol  Meaning                     Presentation      Examples
  ------  -------                     ------------      -------
   H       hour-of-day (0-23)          number            0
   m       minute-of-hour              number            30
   s       second-of-minute            number            55
   S       fraction-of-second          fraction          978
   n       nano-of-second              number            987654321
```

使用`DateTimeFormatter`格式化一个`java.time.LocalTime`实例非常简单。假设我们想要显示用冒号分隔的时间(小时、分钟和秒):

```java
String timeColonPattern = "HH:mm:ss";
DateTimeFormatter timeColonFormatter = DateTimeFormatter.ofPattern(timeColonPattern);
LocalTime colonTime = LocalTime.of(17, 35, 50);
System.out.println(timeColonFormatter.format(colonTime));
```

这将生成输出"`17:35:50.`"

如果我们想在输出中增加毫秒，我们应该在模式中增加“SSS ”:

```java
String timeColonPattern = "HH:mm:ss SSS";
DateTimeFormatter timeColonFormatter = DateTimeFormatter.ofPattern(timeColonPattern);
LocalTime colonTime = LocalTime.of(17, 35, 50).plus(329, ChronoUnit.MILLIS);
System.out.println(timeColonFormatter.format(colonTime));
```

这给了我们输出"`17:35:50 329.`"

请注意，“HH”是一天中的小时模式，它生成 0-23 的输出。当我们想要显示 AM/PM 时，我们应该使用小写字母“hh”表示小时，并添加一个“a”模式:

```java
String timeColonPattern = "hh:mm:ss a";
DateTimeFormatter timeColonFormatter = DateTimeFormatter.ofPattern(timeColonPattern);
LocalTime colonTime = LocalTime.of(17, 35, 50);
System.out.println(timeColonFormatter.format(colonTime));
```

生成的输出是“`05:35:50 PM.`

我们可能想用自定义格式化程序解析一个时间`String`,并检查它是否在中午之前:

```java
DateTimeFormatter timeFormatter = DateTimeFormatter.ofPattern("hh:mm:ss a");
System.out.println(LocalTime.from(timeFormatter.parse("12:25:30 AM")).isBefore(LocalTime.NOON));
```

最后一个代码片段的输出显示给定的时间实际上是在中午之前。

### 4.3.`DateTimeFormatter`对于时区

我们通常希望看到某个特定日期时间变量的时区。如果我们使用基于纽约的日期时间(UTC -4)，我们可以使用“z”模式字母作为时区名称:

```java
String newYorkDateTimePattern = "dd.MM.yyyy HH:mm z";
DateTimeFormatter newYorkDateFormatter = DateTimeFormatter.ofPattern(newYorkDateTimePattern);
LocalDateTime summerDay = LocalDateTime.of(2016, 7, 31, 14, 15);
System.out.println(newYorkDateFormatter.format(ZonedDateTime.of(summerDay, ZoneId.of("UTC-4"))));
```

这将生成输出“2016 年 7 月 31 日 14:15 UTC-04:00”

我们可以像前面一样解析带时区的日期时间字符串:

```java
DateTimeFormatter zonedFormatter = DateTimeFormatter.ofPattern("dd.MM.yyyy HH:mm z");
System.out.println(ZonedDateTime.from(zonedFormatter.parse("31.07.2016 14:15 GMT+02:00")).getOffset().getTotalSeconds());
```

如我们所料，这段代码的输出是“7200”秒，即 2 小时。

我们必须确保为`parse()`方法提供正确的日期时间`String`。如果我们将“31.07.2016 14:15”不带时区地传递给最后一段代码中的`zonedFormatter`，我们将得到一个`DateTimeParseException`。

### 4.4.`DateTimeFormatter`瞬间

`DateTimeFormatter`带有一个很棒的 ISO 即时格式化程序，叫做`ISO_INSTANT`。顾名思义，这个格式化程序提供了一种方便的方式**来格式化或解析 UTC 中的一个瞬间。**

根据[官方文档](https://web.archive.org/web/20221027035204/https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/time/format/DateTimeFormatter.html#ISO_INSTANT)，**在没有指定时区的情况下，瞬间不能被格式化为日期或时间**。因此，试图在`LocalDateTime`或`LocalDate`对象上使用`ISO_INSTANT`将导致一个异常:

```java
@Test(expected = UnsupportedTemporalTypeException.class)
public void shouldExpectAnExceptionIfInputIsLocalDateTime() {
    DateTimeFormatter.ISO_INSTANT.format(LocalDateTime.now());
}
```

然而，我们可以使用`ISO_INSTANT`来格式化`ZonedDateTime`实例，没有任何问题:

```java
@Test
public void shouldPrintFormattedZonedDateTime() {
    ZonedDateTime zonedDateTime = ZonedDateTime.of(2021, 02, 15, 0, 0, 0, 0, ZoneId.of("Europe/Paris"));
    String formattedZonedDateTime = DateTimeFormatter.ISO_INSTANT.format(zonedDateTime);

    Assert.assertEquals("2021-02-14T23:00:00Z", formattedZonedDateTime);
}
```

如我们所见，我们用“欧洲/巴黎”时区创建了我们的`ZonedDateTime`。但是，格式化后的结果采用 UTC 格式。

同样，**解析到`ZonedDateTime`时，我们需要指定时区**:

```java
@Test
public void shouldParseZonedDateTime() {
    DateTimeFormatter formatter = DateTimeFormatter.ISO_INSTANT.withZone(ZoneId.systemDefault());
    ZonedDateTime zonedDateTime = ZonedDateTime.parse("2021-10-01T05:06:20Z", formatter);

    Assert.assertEquals("2021-10-01T05:06:20Z", DateTimeFormatter.ISO_INSTANT.format(zonedDateTime));
}
```

不这样做就会导致 [`DateTimeParseException`](https://web.archive.org/web/20221027035204/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/format/DateTimeParseException.html) :

```java
@Test(expected = DateTimeParseException.class)
public void shouldExpectAnExceptionIfTimeZoneIsMissing() {
    ZonedDateTime zonedDateTime = ZonedDateTime.parse("2021-11-01T05:06:20Z", DateTimeFormatter.ISO_INSTANT);
}
```

同样值得一提的是，**解析要求至少指定`seconds`字段**。否则会抛出`DateTimeParseException`。

## 5.结论

**在本文中，我们讨论了如何使用`DateTimeFormatter `类来格式化日期和时间。**我们还研究了现实生活中的例子模式，这些模式经常在我们处理日期时间实例时出现。

我们可以在[之前的教程](/web/20221027035204/https://www.baeldung.com/java-8-date-time-intro)中找到更多关于 Java 8 的`Date/Time` API。和往常一样，本文中使用的源代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20221027035204/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-datetime-string)