# 用 Java 从 Unix 时间戳创建日期

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-date-unix-timestamp>

## 1.介绍

在这个快速教程中，我们将学习如何解析来自 Unix 时间戳的日期表示。 [**Unix 时间**](https://web.archive.org/web/20220930182427/https://en.wikipedia.org/wiki/Unix_time) **是从 1970 年 1 月 1 日起经过的秒数。然而，时间戳可以表示低至纳秒精度的时间。**因此，我们将看到可用的工具，并创建一个方法来将任意范围的时间戳转换成 Java 对象。

## 2.老方法(Java 8 之前)

**在 Java 8 之前，我们最简单的选项是`Date`和`Calendar`。**`Date`类有一个直接接受以毫秒为单位的时间戳的构造函数:

```java
public static Date dateFrom(long input) {
    return new Date(input);
}
```

有了`Calendar`，我们不得不在`getInstance()`之后叫`setTimeInMillis()` :

```java
public static Calendar calendarFrom(long input) {
    Calendar calendar = Calendar.getInstance();
    calendar.setTimeInMillis(input);
    return calendar;
}
```

换句话说，我们必须知道我们的输入是以秒、纳秒还是其他介于两者之间的精度为单位。然后，我们必须手动将时间戳转换为毫秒。

## 3.新方式(Java 8+)

Java 8 推出`Instant`。**这个类有实用方法来创建秒和毫秒的实例。**还有，其中一个接受纳秒调整参数:

```java
Instant.ofEpochSecond(seconds, nanos);
```

但是我们仍然必须事先知道时间戳的精度。因此，例如，如果我们知道我们的时间戳以纳秒为单位，就需要进行一些计算:

```java
public static Instant fromNanos(long input) {
    long seconds = input / 1_000_000_000;
    long nanos = input % 1_000_000_000;

    return Instant.ofEpochSecond(seconds, nanos);
}
```

首先，我们将时间戳除以十亿得到秒。然后，我们用它的[余数](/web/20220930182427/https://www.baeldung.com/modulo-java)得到秒后的部分。

## 4.即时通用解决方案

为了避免额外的工作，让我们创建一个可以将任何输入转换成毫秒的方法，这是大多数类都可以解析的。首先，我们检查我们的时间戳在什么范围内。然后，我们执行计算来提取毫秒。此外，我们将使用科学符号使我们的条件更具可读性。

另外，记住时间戳是用[符号](/web/20220930182427/https://www.baeldung.com/java-unsigned-arithmetic)表示的，所以我们必须检查正负范围(负时间戳意味着它们从 1970 年开始向后计数)。

**那么，让我们从检查我们的输入是否以纳秒为单位开始**:

```java
private static long millis(long timestamp) {
    if (millis >= 1E16 || millis <= -1E16) {
        return timestamp / 1_000_000;
    }

    // next range checks
}
```

首先，我们检查它是否在`1E16`范围内，即 1 后面跟着 16 个 0。**负值表示 1970 年之前的日期，所以我们也要检查。**然后，我们将我们的值除以一百万得到毫秒。

同样，微秒在`1E14`范围内。这一次，我们除以一千:

```java
if (timestamp >= 1E14 || timestamp <= -1E14) {
    return timestamp / 1_000;
}
```

**当我们的值在 1E11 到-3E10 范围内**时，我们不需要改变任何东西。这意味着我们的输入已经是毫秒精度了:

```java
if (timestamp >= 1E11 || timestamp <= -3E10) {
    return timestamp;
}
```

最后，如果我们的输入不是这些范围中的任何一个，那么它必须以秒为单位，所以我们需要将其转换为毫秒:

```java
return timestamp * 1_000;
```

### 4.1.将`Instant`的输入标准化

**现在，让我们创建一个方法，用`Instant.ofEpochMilli()`** 以任意精度从输入中返回一个`Instant`:

```java
public static Instant fromTimestamp(long input) {
    return Instant.ofEpochMilli(millis(input));
}
```

请注意，每次我们对值进行除法或乘法运算时，精度都会降低。

### 4.2.当地时间与`LocalDateTime`

**An `Instant`代表时间上的一个瞬间。但是，如果没有[时区](/web/20220930182427/https://www.baeldung.com/java-set-date-time-zone)，它就不容易阅读，因为它取决于我们在世界上的位置。**所以，让我们创建一个生成本地时间表示的方法。我们将使用 UTC 来避免测试中出现不同的结果:

```java
public static LocalDateTime localTimeUtc(Instant instant) {
    return LocalDateTime.ofInstant(instant, ZoneOffset.UTC);
}
```

**现在，我们可以测试当方法需要特定格式时，使用错误的精度会导致完全不同的日期。**首先，让我们传递一个我们已经知道正确日期的以纳秒为单位的时间戳，但是将它转换成微秒并使用我们之前创建的`fromNanos()`方法:

```java
@Test
void givenWrongPrecision_whenInstantFromNanos_thenUnexpectedTime() {
    long microseconds = 1660663532747420283l / 1000;
    Instant instant = fromNanos(microseconds);
    String expectedTime = "2022-08-16T15:25:32";

    LocalDateTime time = localTimeUtc(instant);
    assertThat(!time.toString().startsWith(expectedTime));
    assertEquals("1970-01-20T05:17:43.532747420", time.toString());
}
```

**当我们使用在前面小节中创建的`fromTimestamp()`方法**时，这个问题不会发生:

```java
@Test
void givenMicroseconds_whenInstantFromTimestamp_thenLocalTimeMatches() {
    long microseconds = 1660663532747420283l / 1000;

    Instant instant = fromTimestamp(microseconds);
    String expectedTime = "2022-08-16T15:25:32";

    LocalDateTime time = localTimeUtc(instant);
    assertThat(time.toString().startsWith(expectedTime));
}
```

## 5.结论

在本文中，我们学习了如何用核心 Java 类转换时间戳。**然后，我们看到了它们如何具有不同的精度水平，以及这如何影响我们的结果。**最后，我们创建了一个简单的方法来标准化我们的输入并获得一致的结果。

和往常一样，源代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220930182427/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-date-operations-3)