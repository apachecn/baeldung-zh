# 在 Java 中将时间转换为毫秒

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-time-milliseconds>

## 1。概述

在这个快速教程中，**我们将举例说明在 Java** 中将时间转换成 Unix 纪元毫秒的多种方法。

更具体地说，我们将使用:

*   核心 Java 的`java.util.Date` 和 `Calendar`
*   Java 8 的日期和时间 API
*   joda-时间图书馆

## 2。核心 Java

### 2.1。使用`Date`

首先，让我们定义一个保存毫秒随机值的`millis`属性:

```
long millis = 1556175797428L; // April 25, 2019 7:03:17.428 UTC
```

我们将使用这个值来初始化我们的各种对象，并验证我们的结果。

接下来，让我们从一个`Date`对象开始:

```
Date date = // implementation details
```

现在，我们准备好通过简单地调用`getTime()`方法将`date`转换成毫秒:

```
Assert.assertEquals(millis, date.getTime());
```

### 2.2。使用`Calendar`

同样，如果我们有一个`Calendar`对象，我们可以使用`getTimeInMillis()`方法:

```
Calendar calendar = // implementation details
Assert.assertEquals(millis, calendar.getTimeInMillis());
```

## 3。Java 8 日期时间 API

### 3.1。使用`Instant`

简单来说， [`Instant`](/web/20220815045938/https://www.baeldung.com/current-date-time-and-timestamp-in-java-8) 是 Java 的纪元时间轴上的一个点。

我们可以从`Instant`获得以毫秒为单位的当前时间:

```
java.time.Instant instant = // implementation details
Assert.assertEquals(millis, instant.toEpochMilli());
```

因此，`toEpochMilli()`方法返回的毫秒数与我们之前定义的相同。

### 3.2。使用`LocalDateTime`

类似地，我们可以使用 Java 8 的[日期和时间 API](/web/20220815045938/https://www.baeldung.com/java-8-date-time-intro) 将`LocalDateTime`转换成毫秒:

```
LocalDateTime localDateTime = // implementation details
ZonedDateTime zdt = ZonedDateTime.of(localDateTime, ZoneId.systemDefault());
Assert.assertEquals(millis, zdt.toInstant().toEpochMilli());
```

首先，我们创建了当前日期的一个实例。之后，我们使用`toEpochMilli()`方法将 *ZonedDateTime* 转换成毫秒。

我们知道，`LocalDateTime`不包含关于时区的信息。换句话说，**我们不能直接从`LocalDateTime`实例**中获得毫秒。

## 4。Joda-Time

虽然 Java 8 增加了许多 Joda-Time 的功能，但是如果我们使用的是 Java 7 或更早的版本，我们可能希望使用这个选项。

### 4.1。使用`Instant`

首先，我们可以使用`getMillis()`方法从 **[Joda-Time](/web/20220815045938/https://www.baeldung.com/joda-time)** `Instant`类实例中获取当前系统毫秒数:

```
Instant jodaInstant = // implementation details
Assert.assertEquals(millis, jodaInstant.getMillis());
```

### 4.2。使用`DateTime`

此外，如果我们有一个 Joda-Time `DateTime`实例:

```
DateTime jodaDateTime = // implementation details
```

然后我们可以用`getMillis()`方法检索毫秒:

```
Assert.assertEquals(millis, jodaDateTime.getMillis());
```

## 5。结论

总之，本文演示了如何在 Java 中将时间转换成毫秒。

最后，和往常一样，本文的完整代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220815045938/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-datetime-conversion)