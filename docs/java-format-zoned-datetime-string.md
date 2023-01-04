# 将 ZonedDateTime 格式化为字符串

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-format-zoned-datetime-string>

## 1。概述

在这个快速教程中，我们将看到如何将一个`ZonedDateTime`转换成一个`String.`

我们还将看看如何从`String.`解析`ZonedDateTime`

## 2。创造一个`ZonedDateTime`

首先，我们将从时区为 UTC 的`ZonedDateTime`开始。有几种方法可以实现这一点。

我们可以指定年、月、日等:

```
ZonedDateTime zonedDateTimeOf = ZonedDateTime.of(2018, 01, 01, 0, 0, 0, 0, ZoneId.of("UTC"));
```

我们还可以从当前日期和时间创建一个`ZonedDateTime`:

```
ZonedDateTime zonedDateTimeNow = ZonedDateTime.now(ZoneId.of("UTC"));
```

或者，我们可以从现有的`LocalDateTime`创建一个`ZonedDateTime`:

```
LocalDateTime localDateTime = LocalDateTime.now();
ZonedDateTime zonedDateTime = ZonedDateTime.of(localDateTime, ZoneId.of("UTC"));
```

## 3。`ZonedDateTime`到`String`到

现在，让我们把我们的`ZonedDateTime`转换成`String.`为此，**我们将使用`DateTimeFormatter`类。**

我们可以使用一些特殊的格式化程序来显示时区数据。格式化程序的完整列表可以在[这里](https://web.archive.org/web/20220627083749/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/format/DateTimeFormatter.html)找到，但是我们将看几个更常见的。

如果我们想让**显示时区偏移量，我们可以使用**格式化程序**“Z”或“X”**:

```
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("MM/dd/yyyy - HH:mm:ss Z");
String formattedString = zonedDateTime.format(formatter);
```

这会给我们这样的结果:

```
02/01/2018 - 13:45:30 +0000
```

**为了包含时区名称，我们可以使用小写的“z”:**

```
DateTimeFormatter formatter2 = DateTimeFormatter.ofPattern("MM/dd/yyyy - HH:mm:ss z");
String formattedString2 = zonedDateTime.format(formatter2);
```

这样的输出将是:

```
02/01/2018 - 13:45:30 UTC
```

## 4。`String`到`ZonedDateTime`到

这个过程也可以反过来进行。我们可以将一个字符串转换回一个`ZonedDateTime`。

一种方法是使用`ZonedDateTime`:类的**静态`parse()`方法**

```
ZonedDateTime zonedDateTime = ZonedDateTime.parse("2011-12-03T10:15:30+01:00");
```

该方法使用`ISO_ZONED_DATE_TIME`格式化程序。该方法还有一个重载版本，它接受一个`DateTimeFormatter`参数。然而，该字符串必须包含一个区域标识符，否则我们将得到一个异常:

```
assertThrows(DateTimeParseException.class, () -> 
  ZonedDateTime.parse("2011-12-03T10:15:30", DateTimeFormatter.ISO_DATE_TIME));
```

从`String`获取`ZonedDateTime`的第二个选项包括两个步骤:**将字符串转换为`LocalDateTime,`，然后将该对象转换为`ZonedDateTime:`**

```
ZoneId timeZone = ZoneId.systemDefault();
ZonedDateTime zonedDateTime = LocalDateTime.parse("2011-12-03T10:15:30", 
  DateTimeFormatter.ISO_DATE_TIME).atZone(timeZone);

log.info(zonedDateTime.format(DateTimeFormatter.ISO_ZONED_DATE_TIME));
```

这种间接方法只是将日期时间与时区 id 结合起来:

```
INFO: 2011-12-03T10:15:30+02:00[Europe/Athens]
```

要了解更多关于将字符串解析为日期的信息，请查看我们更深入的日期解析文章。

## 5。结论

在本文中，我们已经看到了如何创建一个`ZonedDateTime`，以及如何将它格式化为一个`String.`

我们还快速了解了如何解析日期时间字符串并将其转换成`ZonedDateTime`。

本教程的源代码可以在 Github 上的[处获得。](https://web.archive.org/web/20220627083749/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-datetime-string)