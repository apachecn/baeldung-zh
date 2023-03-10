# 将日期转换为本地日期或本地日期时间，然后再转换回来

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-date-to-localdate-and-localdatetime>

## 1。概述

从 Java 8 开始，我们有了一个新的日期 API: `java.time`。

然而，有时我们仍然需要执行新旧 API 之间的转换，并处理两者的日期表示。

## 延伸阅读:

## [迁移到新的 Java 8 日期时间 API](/web/20221213225508/https://www.baeldung.com/migrating-to-java-8-date-time-api)

A quick and practical guide on transitioning to Java 8's new DateTime API.[Read more](/web/20221213225508/https://www.baeldung.com/migrating-to-java-8-date-time-api) →

## [Java 8 日期/时间 API 简介](/web/20221213225508/https://www.baeldung.com/java-8-date-time-intro)

In this article we will take a look at the new Java 8 APIs for Date and Time and how much easier it is to construct and manipulate dates and times.[Read more](/web/20221213225508/https://www.baeldung.com/java-8-date-time-intro) →

## 2。将`java.util.Date`转换为`java.time.LocalDate`

让我们从将旧的日期表示转换成新的开始。

在这里，我们可以利用一个新的 **`toInstant()`方法，它是在 Java 8 中添加到`java.util.Date`** 中的。

当我们转换一个`**Instant**` **对象时，需要使用一个 `ZoneId`** **，因为`Instant`对象是时区不可知的**——只是时间轴上的点。

来自`Instant`对象的`atZone(ZoneId zone)` API 返回一个 `ZonedDateTime`，所以我们只需要使用`toLocalDate()`方法从其中提取`LocalDate`。

首先，我们使用默认系统`ZoneId`:

```java
public LocalDate convertToLocalDateViaInstant(Date dateToConvert) {
    return dateToConvert.toInstant()
      .atZone(ZoneId.systemDefault())
      .toLocalDate();
}
```

**和一个类似的解决方案，但是创建`Instant`对象**的方式不同——使用`ofEpochMilli()`方法:

```java
public LocalDate convertToLocalDateViaMilisecond(Date dateToConvert) {
    return Instant.ofEpochMilli(dateToConvert.getTime())
      .atZone(ZoneId.systemDefault())
      .toLocalDate();
}
```

在我们继续之前，让我们也快速地看一下旧的 **`java.sql.Date`**级以及如何将它转换为`LocalDate`。

从 Java 8 开始，我们可以在`java.sql.Date`上找到一个额外的`toLocalDate()` 方法，这也给了我们一个将其转换为`java.time.LocalDate`的简单方法。

在这种情况下，我们不需要担心时区:

```java
public LocalDate convertToLocalDateViaSqlDate(Date dateToConvert) {
    return new java.sql.Date(dateToConvert.getTime()).toLocalDate();
}
```

同样，我们也可以将旧的`Date`对象转换成`LocalDateTime`对象。接下来让我们来看看。

## 3。将`java.util.Date`转换为`java.time.LocalDateTime`

为了获得一个`LocalDateTime`实例，我们可以类似地使用**一个中介`ZonedDateTime`，然后使用`toLocalDateTime()` API。**

就像之前一样，我们可以使用两种可能的解决方案从`java.util.Date`获取一个`Instant`对象:

```java
public LocalDateTime convertToLocalDateTimeViaInstant(Date dateToConvert) {
    return dateToConvert.toInstant()
      .atZone(ZoneId.systemDefault())
      .toLocalDateTime();
}

public LocalDateTime convertToLocalDateTimeViaMilisecond(Date dateToConvert) {
    return Instant.ofEpochMilli(dateToConvert.getTime())
      .atZone(ZoneId.systemDefault())
      .toLocalDateTime();
}
```

注意，对于 1582 年 10 月 10 日之前的日期，需要将 Calendar 设置为公历，并调用方法`**[setGregorianChange](https://web.archive.org/web/20221213225508/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/GregorianCalendar.html#setGregorianChange(java.util.Date))():**`

```java
GregorianCalendar calendar = new GregorianCalendar(); calendar.setGregorianChange(new Date(Long.MIN_VALUE)); Date dateToConvert = calendar.getTime();
```

并且从 Java 8 开始，我们还可以**使用 `java.sql.Timestamp`获得一个`LocalDateTime`** :

```java
ocalDateTime convertToLocalDateTimeViaSqlTimestamp(Date dateToConvert) {
    return new java.sql.Timestamp(
      dateToConvert.getTime()).toLocalDateTime();
}
```

## 4。将`java.time.LocalDate`转换为`java.util.Date`

现在我们已经很好地理解了如何从旧的数据表示转换到新的数据表示，让我们来看看另一个方向的转换。

我们将讨论将`LocalDate`转换为`Date`的两种可能方式。

首先，我们使用 `**java.sql.Date**`对象中提供的一个新的 **`valueOf(LocalDate date)`方法，它将`LocalDate`作为一个参数:**

```java
public Date convertToDateViaSqlDate(LocalDate dateToConvert) {
    return java.sql.Date.valueOf(dateToConvert);
}
```

正如我们所见，这是毫不费力和直观的。它使用当地时区进行转换(所有工作都在幕后完成，因此无需担心)。

在另一个 Java 8 例子中，我们使用一个`Instant`对象，我们将它传递给 `**java.util.Date**`对象的`**from(Instant instant)**` **方法:**

```java
public Date convertToDateViaInstant(LocalDate dateToConvert) {
    return java.util.Date.from(dateToConvert.atStartOfDay()
      .atZone(ZoneId.systemDefault())
      .toInstant());
}
```

注意，我们在这里使用了一个`Instant`对象，在进行转换时，我们还需要考虑时区。

接下来，让我们使用一个非常相似的解决方案将一个`LocalDateTime`转换成一个`Date`对象。

## 5。将`java.time.LocalDateTime`转换为`java.util.Date`

从`LocalDateTime`获得 **a `java.util.Date`的最简单方法是使用对** `**java.sql.Timestamp**`的扩展 Java 8 中可用:

```java
public Date convertToDateViaSqlTimestamp(LocalDateTime dateToConvert) {
    return java.sql.Timestamp.valueOf(dateToConvert);
}
```

当然，另一个解决方案是使用**和`Instant`对象，我们从`ZonedDateTime`和**中获得:

```java
Date convertToDateViaInstant(LocalDateTime dateToConvert) {
    return java.util.Date
      .from(dateToConvert.atZone(ZoneId.systemDefault())
      .toInstant());
}
```

## 6。Java 9 新增功能

在 Java 9 中，有新的方法可以简化`java.util.Date`和`java.time.LocalDate`或`java.time.LocalDateTime`之间的转换。

`**LocalDate.ofInstant(Instant instant, ZoneId zone)**`和`**LocalDateTime.ofInstant(Instant instant, ZoneId zone)**`提供了便捷的快捷方式:

```java
public LocalDate convertToLocalDate(Date dateToConvert) {
    return LocalDate.ofInstant(
      dateToConvert.toInstant(), ZoneId.systemDefault());
}

public LocalDateTime convertToLocalDateTime(Date dateToConvert) {
    return LocalDateTime.ofInstant(
      dateToConvert.toInstant(), ZoneId.systemDefault());
}
```

## 7。结论

在本文中，我们讨论了将旧的`java.util.Date`转换成新的`java.time.LocalDate`和`java.time.LocalDateTime`的可能方法，以及反过来的方法。

这篇文章的完整实现可以在 GitHub 的[上找到。](https://web.archive.org/web/20221213225508/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-datetime-conversion)