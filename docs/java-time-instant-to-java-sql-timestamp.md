# 在 java.time.Instant 和 java.sql.Timestamp 之间转换

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-time-instant-to-java-sql-timestamp>

## 1.概观

`java.time.Instant`和`java.sql.Timestamp`类都代表 UTC 时间线上的一个点。换句话说，它们代表了自 Java 时代以来[的纳秒数。](https://web.archive.org/web/20220525135037/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/Instant.html)

在这个快速教程中，我们将通过使用内置的 Java 方法进行转换。

## 2.将`Instant`转换为`Timestamp`并返回

**我们可以使用`Timestamp.from()`将`Instant`转换成时间戳:**

```
Instant instant = Instant.now();
Timestamp timestamp = Timestamp.from(instant);
assertEquals(instant.toEpochMilli(), timestamp.getTime());
```

反之亦然，我们可以用`Timestamp.toInstant()`把`Timestamp` s 转换成`Instant` s:

```
instant = timestamp.toInstant();
assertEquals(instant.toEpochMilli(), timestamp.getTime());
```

**不管怎样，`Instant`和`Timestamp`都代表时间线上的同一点。**

接下来，让我们看看这两个类和时区之间的交互。

## 3.`toString()`方法差异

**在`Instant`和`Timestamp`上调用`toString()`在时区方面表现不同。** `Instant.toString()`返回 UTC 时区的时间。另一方面，`Timezone.toString()`返回本地机器时区的时间。

让我们看看分别在`instant`和`timestamp` 上调用`toString()`时会得到什么:

```
Instant (in UTC): 2018-10-18T00:00:57.907Z
Timestamp (in GMT +05:30): 2018-10-18 05:30:57.907
```

`Here, timestamp.toString()` 得到的时间比`instant.toString().` 返回的时间晚 5 小时 30 分钟，这是因为本地机器的时区是 GMT +5:30 时区。

**方法`toString()`的输出是不同的，但是`timestamp`和`instant`都表示时间轴**上的同一点。

我们也可以通过将`Timestamp`转换为 UTC 时区来验证这一点:

```
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd'T'HH:mm:ss.SSS'Z'");
formatter = formatter.withZone(TimeZone.getTimeZone("UTC").toZoneId());
DateFormat df = new SimpleDateFormat("yyyy-MM-dd'T'HH:mm:ss.SSS'Z'");

assertThat(formatter.format(instant)).isEqualTo(df.format(timestamp));
```

## 4.结论

在这个快速教程中，我们看到了如何使用内置方法在 Java 中的`java.time.Instant`和`java.sql.Timestamp`类之间进行转换。

我们还了解了时区如何影响输出的变化。

和往常一样，GitHub 上的[提供了完整的代码示例。](https://web.archive.org/web/20220525135037/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-datetime-conversion)