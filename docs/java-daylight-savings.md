# 在 Java 中处理夏令时

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-daylight-savings>

## 1。概述

[夏令时](https://web.archive.org/web/20220930174835/https://en.wikipedia.org/wiki/Daylight_saving_time)或 DST，是一种在夏季月份将时钟提前的做法，以便利用额外的一个小时自然光(节省加热功率、照明功率、提升情绪等)。

它被几个国家使用，在处理日期和时间戳时需要考虑到这一点。

在本教程中，我们将看到如何根据不同的位置在 Java 中正确处理 DST。

## 2。JRE 和 DST 可变性

首先，理解世界各地的夏令时区[经常变化](https://web.archive.org/web/20220930174835/http://www.oracle.com/technetwork/java/javase/dst-faq-138158.html#change)并且没有中央机构来协调它是非常重要的。

一个国家，或者在某些情况下甚至是一个城市，可以决定是否以及如何应用或撤销它。

每次它发生时，更改都会被记录在 [IANA 时区数据库](https://web.archive.org/web/20220930174835/https://www.iana.org/time-zones)中，并且更新将在 JRE 的未来[版本中推出。](https://web.archive.org/web/20220930174835/http://www.oracle.com/technetwork/java/javase/tzdata-versions-138805.html)

**如果不可能等待，我们可以通过一个名为 [Java 时区更新工具](https://web.archive.org/web/20220930174835/http://www.oracle.com/technetwork/java/javase/documentation/timezones-137583.html)的 Oracle 官方工具，将包含新 DST 设置的修改后的时区数据强制导入 JRE** ，该工具可从 [Java SE 下载页面](https://web.archive.org/web/20220930174835/http://www.oracle.com/technetwork/java/javase/downloads/index.html)获得。

## 3。错误的方式:三个字母的时区 ID

回到 JDK 1.1 时代，API 允许三个字母的时区 id，但是这导致了几个问题。

首先，这是因为同一个三个字母的 ID 可以指代多个时区。例如，`CST`可以是美国“中部标准时间”，也可以是“中国标准时间”。Java 平台只能识别其中一个。

另一个问题是标准时区从不考虑夏令时。多个地区/地区/城市可以在同一个标准时区内使用其本地 DST，因此标准时间不会遵守它。

由于向后兼容，**仍然可以用三个字母的 ID 实例化一个`[java.util.Timezone](https://web.archive.org/web/20220930174835/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/TimeZone.html)`。然而，这种方法已被弃用，不应再使用。**

## 4。正确的方法:TZDB 时区 ID

在 Java 中处理 DST 的正确方法是用特定的 TZDB 时区 ID 实例化一个`Timezone` ，例如`“Europe/Rome”.`

然后，我们将把它与特定于时间的类(如`java.util.Calendar`)结合使用，以获得`TimeZone's`原始偏移量(到 GMT 时区)的正确配置，以及自动 DST 偏移调整。

让我们看看当使用右`TimeZone:`时，如何自动处理从`GMT+1`到`GMT+2`的转换(发生在意大利，2018 年 3 月 25 日，凌晨 02:00)

```java
TimeZone tz = TimeZone.getTimeZone("Europe/Rome");
TimeZone.setDefault(tz);
Calendar cal = Calendar.getInstance(tz, Locale.ITALIAN);
DateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm", Locale.ITALIAN);
Date dateBeforeDST = df.parse("2018-03-25 01:55");
cal.setTime(dateBeforeDST);

assertThat(cal.get(Calendar.ZONE_OFFSET)).isEqualTo(3600000);
assertThat(cal.get(Calendar.DST_OFFSET)).isEqualTo(0);
```

我们可以看到，`ZONE_OFFSET`是 60 分钟(因为义大利是`GMT+1`)，而`DST_OFFSET`当时是 0。

让我们给`Calendar`增加十分钟:

```java
cal.add(Calendar.MINUTE, 10); 
```

现在`DST_OFFSET`也变成了 60 分钟，这个国家已经把当地时间从`CET`(中欧时间)过渡到`CEST`(中欧夏令时)，也就是`GMT+2`:

```java
Date dateAfterDST = cal.getTime();

assertThat(cal.get(Calendar.DST_OFFSET))
  .isEqualTo(3600000);
assertThat(dateAfterDST)
  .isEqualTo(df.parse("2018-03-25 03:05")); 
```

如果我们在控制台中显示这两个日期，我们也会看到时区的变化:

```java
Before DST (00:55 UTC - 01:55 GMT+1) = Sun Mar 25 01:55:00 CET 2018
After DST (01:05 UTC - 03:05 GMT+2) = Sun Mar 25 03:05:00 CEST 2018
```

作为最后的测试，我们可以测量两个`Date`之间的距离，1:55 和 3:05:

```java
Long deltaBetweenDatesInMillis = dateAfterDST.getTime() - dateBeforeDST.getTime();
Long tenMinutesInMillis = (1000L * 60 * 10);

assertThat(deltaBetweenDatesInMillis)
  .isEqualTo(tenMinutesInMillis); 
```

正如我们所料，距离是 10 分钟而不是 70 分钟。

我们已经看到了如何通过正确使用`TimeZone`和`Locale`来避免陷入使用`Date`时可能遇到的常见陷阱。

## 5。最好的方法:Java 8 日期/时间 API

使用这些线程不安全且不总是用户友好的`java.util`类总是很困难，尤其是由于兼容性问题，这使得它们无法被正确重构。

出于这个原因，Java 8 引入了一个全新的包`java.time`，以及一个全新的 API 集[日期/时间 API。](/web/20220930174835/https://www.baeldung.com/java-8-date-time-intro)这是以 ISO 为中心的，完全线程安全的，并且深受著名的库 Joda-Time 的启发。

让我们仔细看看这个新的类，从`java.util.Date`、`java.time.LocalDateTime`的继承者开始:

```java
LocalDateTime localDateTimeBeforeDST = LocalDateTime
  .of(2018, 3, 25, 1, 55);

assertThat(localDateTimeBeforeDST.toString())
  .isEqualTo("2018-03-25T01:55"); 
```

我们可以观察到一个`LocalDateTime`是如何符合`ISO8601`轮廓的，这是一个标准的、被广泛采用的日期时间符号。

它完全不知道`Zones`和`Offsets`，但是，这就是为什么我们需要把它转换成**一个完全 DST 感知的`java.time.ZonedDateTime`** :

```java
ZoneId italianZoneId = ZoneId.of("Europe/Rome");
ZonedDateTime zonedDateTimeBeforeDST = localDateTimeBeforeDST
  .atZone(italianZoneId);

assertThat(zonedDateTimeBeforeDST.toString())
  .isEqualTo("2018-03-25T01:55+01:00[Europe/Rome]"); 
```

正如我们所见，现在日期包含了两个基本的尾部信息:`+01:00`是`ZoneOffset`，而`[Europe/Rome]`是`ZoneId`。

像前面的例子一样，让我们通过增加十分钟来触发 DST:

```java
ZonedDateTime zonedDateTimeAfterDST = zonedDateTimeBeforeDST
  .plus(10, ChronoUnit.MINUTES);

assertThat(zonedDateTimeAfterDST.toString())
  .isEqualTo("2018-03-25T03:05+02:00[Europe/Rome]"); 
```

我们再次看到时间和时区偏移是如何向前移动的，并且仍然保持相同的距离:

```java
Long deltaBetweenDatesInMinutes = ChronoUnit.MINUTES
  .between(zonedDateTimeBeforeDST,zonedDateTimeAfterDST);
assertThat(deltaBetweenDatesInMinutes)
  .isEqualTo(10); 
```

## 6。结论

我们已经通过不同版本的 Java core API 中的一些实际例子了解了什么是夏令时以及如何处理夏令时。

当使用 Java 8 和更高版本时，由于其易用性和标准的线程安全特性，鼓励使用新的`java.time`包。

和往常一样，完整的源代码可以在 Github 上找到[。](https://web.archive.org/web/20220930174835/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-date-operations-1)