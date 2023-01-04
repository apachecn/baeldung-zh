# 在 Java 中设置日期的时区

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-set-date-time-zone>

## 1.概观

在这个快速教程中，我们将看到如何使用 Java 7、Java 8 和 Joda-Time 库设置日期的时区。

## 2.使用 Java 8

Java 8 引入了一个新的日期时间 API ,用于处理日期和时间，它很大程度上基于 Joda-Time 库。

Java Date Time API 中的 [`Instant`](https://web.archive.org/web/20220930182434/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/Instant.html) 类模拟了 UTC 时间线上的一个瞬时点。这表示从世界协调时 1970 年第一时刻开始的纳秒计数。

首先，我们将从系统时钟获取当前的`Instant`和时区名称的 [`ZoneId`](https://web.archive.org/web/20220930182434/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/ZoneId.html) :

```java
Instant nowUtc = Instant.now();
ZoneId asiaSingapore = ZoneId.of("Asia/Singapore");
```

最后，`ZoneId`和`Instant`可以用来创建一个包含时区细节的日期时间对象。 [`ZonedDateTime`](https://web.archive.org/web/20220930182434/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/ZonedDateTime.html) 类表示 ISO-8601 日历系统中带时区的日期时间:

```java
ZonedDateTime nowAsiaSingapore = ZonedDateTime.ofInstant(nowUtc, asiaSingapore);
```

我们使用 Java 8 的`ZonedDateTime`来表示带有时区的日期时间。

## 3.使用 Java 7

在 Java 7 中，设置时区有点复杂。 [`Date`](https://web.archive.org/web/20220930182434/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Date.html) 类(代表特定的时间瞬间)不包含任何时区信息。

首先，让我们获取当前的 UTC 日期和一个 [`TimeZone`](https://web.archive.org/web/20220930182434/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/TimeZone.html) 对象:

```java
Date nowUtc = new Date();
TimeZone asiaSingapore = TimeZone.getTimeZone(timeZone);
```

在 Java 7 中，我们需要使用`[Calendar](https://web.archive.org/web/20220930182434/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Calendar.html)`类来表示带有时区的日期。

最后，我们可以用`asiaSingapore TimeZone`创建一个`nowUtc Calendar`并设置时间:

```java
Calendar nowAsiaSingapore = Calendar.getInstance(asiaSingapore);
nowAsiaSingapore.setTime(nowUtc);
```

建议避免使用 Java 7 日期时间 API，而使用 Java 8 日期时间 API 或 Joda-Time 库。

## 4.使用 Joda-Time

如果 Java 8 不是一个选项，**我们仍然可以从 [Joda-Time](https://web.archive.org/web/20220930182434/http://www.joda.org/joda-time/)** 得到同样的结果，这是 Java 8 出现之前的日期-时间操作的事实上的标准。

首先，我们需要将[的 Joda 时间依赖关系](https://web.archive.org/web/20220930182434/https://search.maven.org/classic/#artifactdetails%7Cjoda-time%7Cjoda-time%7C2.10%7Cjar)添加到`pom.xml:`中

```java
<dependency>
  <groupId>joda-time</groupId>
  <artifactId>joda-time</artifactId>
  <version>2.10</version>
</dependency>
```

为了表示时间线上的一个精确点，我们可以使用`org.joda.time`包中的`[Instant](https://web.archive.org/web/20220930182434/http://joda-time.sourceforge.net/apidocs/org/joda/time/Instant.html) `。在内部，该类保存一段数据，即从 Java 纪元 1970-01-01T00:00:00Z 开始的毫秒时刻:

```java
Instant nowUtc = Instant.now();
```

我们将使用 [`DateTimeZone`](https://web.archive.org/web/20220930182434/https://www.joda.org/joda-time/apidocs/org/joda/time/DateTimeZone.html) 来表示一个时区(对于指定的时区 id):

```java
DateTimeZone asiaSingapore = DateTimeZone.forID("Asia/Singapore");
```

现在使用时区信息将`nowUtc`时间转换为 [`DateTime`](https://web.archive.org/web/20220930182434/https://www.joda.org/joda-time/apidocs/org/joda/time/DateTime.html) 对象:

```java
DateTime nowAsiaSingapore = nowUtc.toDateTime(asiaSingapore);
```

这就是如何使用 Joda-time API 来组合日期和时区信息。

## 5.结论

在本文中，我们了解了如何使用 Java 7、8 和 Joda-Time API 在 Java 中设置时区。要了解更多关于 Java 8 的日期时间支持，请查看我们的 Java 8 日期时间介绍。

与往常一样，代码片段可以在 [GitHub 库](https://web.archive.org/web/20220930182434/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-8-datetime)中获得。