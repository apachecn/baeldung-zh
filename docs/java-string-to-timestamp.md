# 在字符串和时间戳之间转换

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-string-to-timestamp>

## 1.概观

`Timestamp`是 Java 中为数不多的传统日期时间对象之一。

在本教程中，我们将看到如何从一个`String`值解析为一个`Timestamp`对象，以及如何将一个`Timestamp`对象格式化为一个`String.`

由于`Timestamp`依赖于 Java 专有的格式，我们将看到如何有效地适应。

## 2.将一个`String`解析为一个`Timestamp`

### 2.1.标准书型

**将一个`String `解析成一个`Timestamp`的最简单的方法是它的`valueOf `方法:**

```
Timestamp.valueOf("2018-11-12 01:02:03.123456789")
```

当我们的`String`是 JDBC 时间戳格式时——`yyyy-m[m]-d[d] hh:mm`:`ss``[.f…]`——那就相当简单了。

我们可以这样解释这个模式:

| 模式 | 描述 | 例子 |
| --- | --- | --- |
| `yyyy` | 代表年份，必须有四位数。 | Two thousand and eighteen |
| `m[m]` | 对于月份部分，我们必须有一位或两位数字(从 1 到 12)。 | 1, 11 |
| `d[d]` | 对于日期值，我们必须有一位或两位数字(从 1 到 31)。 | 7, 12 |
| `hh` | 代表一天中的小时，允许值为 0 到 23。 | 01, 16 |
| `mm` | 代表一小时中的分钟，允许值为 0 到 59。 | 02, 45 |
| `ss` | 代表一分钟中的秒，允许值为 0 到 59。 | 03, 52 |
| `[.f…]` | 表示可选的几分之一秒，可以达到纳秒精度，因此允许的值是从 0 到 999999999。 | 12, 1567, 123456789 |

### 2.2.替代格式

现在，如果它不是 JDBC 时间戳格式，那么幸运的是，`valueOf `还接受了一个`LocalDateTime `实例。

**这意味着我们可以采用任何格式的日期，**我们只需要先[将其转换成`LocalDateTime`](/web/20221126221618/https://www.baeldung.com/java-string-to-date) :

```
String pattern = "MMM dd, yyyy HH:mm:ss.SSSSSSSS";
String timestampAsString = "Nov 12, 2018 13:02:56.12345678";
DateTimeFormatter formatter = DateTimeFormatter.ofPattern(pattern);
LocalDateTime localDateTime = LocalDateTime.from(formatter.parse(timestampAsString));
```

然后我们可以使用之前用过的`valueOf`:

```
Timestamp timestamp = Timestamp.valueOf(localDateTime);
assertEquals("2018-11-12 13:02:56.12345678", timestamp.toString());
```

顺便提一下，**不像`Date`对象，`Timestamp`对象能够存储几分之一秒的数据。**

## 3.将 a `Timestamp`格式化为一个`String`

要格式化`Timestamp`，我们将面临同样的挑战，因为它的默认格式是专有的 JDBC 时间戳格式:

```
assertEquals("2018-11-12 13:02:56.12345678", timestamp.toString());
```

但是，同样，使用中间转换，我们可以将结果`String`格式化为不同的日期和时间模式，就像 ISO-8601 标准:

```
Timestamp timestamp = Timestamp.valueOf("2018-12-12 01:02:03.123456789");
DateTimeFormatter formatter = DateTimeFormatter.ISO_LOCAL_DATE_TIME;

String timestampAsString = formatter.format(timestamp.toLocalDateTime());
assertEquals("2018-12-12T01:02:03.123456789", timestampAsString);
```

## 4.结论

在本文中，我们看到了如何在 Java 中的`String`和`Timestamp`对象之间进行转换。此外，我们还看到了如何使用`LocalDateTime`转换作为中间步骤，以便在不同的日期和时间模式之间进行转换。

请务必在 GitHub 上找到所有这些例子和片段[。](https://web.archive.org/web/20221126221618/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-datetime-string)