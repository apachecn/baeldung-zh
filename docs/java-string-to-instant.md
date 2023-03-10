# 将字符串转换为即时

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-string-to-instant>

## 1.概观

在这个快速教程中，**我们将解释如何用 Java** 使用`java.time`包中的类将`String`转换成`Instant`。首先，我们将使用`LocalDateTime`类实现一个解决方案。然后，我们将使用`Instant`类来获取一个时区内的瞬间。

## 2.使用`LocalDateTime`类

`**java.time.LocalDateTime**` **表示没有时区**的日期和/或时间。它是一个本地时间对象，也就是说，它只在特定的上下文中有效，不能在这个上下文之外使用。这个上下文通常是执行代码的机器。

为了从`String`获取时间，我们可以使用`DateTimeFormatter`创建一个格式化的对象，并将这个格式化程序传递给`LocalDateTime`的`parse`方法。此外，我们可以定义自己的格式化程序或使用由`DateTimeFormatter`类提供的预定义格式化程序。

让我们看看如何使用`LocalDateTime.parse()`从`String`获得时间:

```java
String stringDate = "09:15:30 PM, Sun 10/09/2022"; 
String pattern = "hh:mm:ss a, EEE M/d/uuuu"; 
DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern(pattern, Locale.US); 
LocalDateTime localDateTime = LocalDateTime.parse(stringDate, dateTimeFormatter);
```

在上面的例子中，我们使用`LocalDateTime`类来解析日期`String`，该类是用时间表示日期的标准类。我们也可以用 `java.time.LocalDate`来表示没有时间的日期。

## 3.使用`Instant`类

**`java.time.Instant`类，是[日期时间 API](/web/20221106112809/https://www.baeldung.com/java-8-date-time-intro) 的主要类之一，封装了时间轴**上的一个点。此外，它类似于`java.util.Date`类，但给出了纳秒级的精度。

在我们的下一个例子中，我们将使用前面的`LocalDateTime`来获得一个带有赋值`ZoneId`的瞬间:

```java
String stringDate = "09:15:30 PM, Sun 10/09/2022"; 
String pattern = "hh:mm:ss a, EEE M/d/uuuu"; 
DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern(pattern, Locale.US); 
LocalDateTime localDateTime = LocalDateTime.parse(stringDate, dateTimeFormatter); 
ZoneId zoneId = ZoneId.of("America/Chicago"); 
ZonedDateTime zonedDateTime = localDateTime.atZone(zoneId); 
Instant instant = zonedDateTime.toInstant();
```

在上面的例子中，我们首先创建了一个`ZoneId`对象，用于标识一个时区，然后，我们提供了`LocalDateTime`和`Instant`之间的转换规则。

接下来，我们使用`ZonedDateTime`，它用时区和相应的偏移量封装了日期和时间。`ZonedDateTime`类是日期-时间 API 中最接近`java.util.GregorianCalendar`类的类。最后，我们使用`ZonedDateTime.toInstant()` 方法得到一个`Instant`，它将一个时刻从一个时区调整到 UTC。

## 4.结论

在这个快速教程中，我们解释了**如何用 Java** 使用`java.time`包中的类将`String`转换成`Instant`。和往常一样，代码片段可以在 GitHub 上获得[。](https://web.archive.org/web/20221106112809/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-datetime-string-2)