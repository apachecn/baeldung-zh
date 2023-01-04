# 在 Java 中生成随机日期

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-random-dates>

## 1.概观

在本教程中，我们将看到如何以有界和无界的方式生成随机的日期和时间。

我们将看看如何使用 Java 8 中的传统 [`java.util.Date`](/web/20220807065448/https://www.baeldung.com/java-util-date-sql-date) API 和[新的日期时间库](/web/20220807065448/https://www.baeldung.com/java-8-date-time-intro)来生成这些值。

## 2.随机日期和时间

**与[纪元时间](https://web.archive.org/web/20220807065448/https://en.wikipedia.org/wiki/Unix_time)** 相比，日期和时间只不过是 32 位整数，因此我们可以通过以下简单算法生成随机时态值:

1.  生成一个随机的 32 位数字，一个`int`
2.  将生成的随机值传递给适当的日期和时间构造器或构建器

### 2.1.有界的`Instant`

**`java.time.I` `nstant` 是 Java 8 中新增的日期和时间之一。**它们代表时间线上的瞬时点。

为了在另外两个之间产生一个随机的`Instant `,我们可以:

1.  在给定`Instants`的历元秒之间生成一个随机数
2.  通过将随机数传递给`[ofEpochSecond()](https://web.archive.org/web/20220807065448/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/Instant.html#ofEpochSecond(long)) `方法来创建随机数`Instant `

```
public static Instant between(Instant startInclusive, Instant endExclusive) {
    long startSeconds = startInclusive.getEpochSecond();
    long endSeconds = endExclusive.getEpochSecond();
    long random = ThreadLocalRandom
      .current()
      .nextLong(startSeconds, endSeconds);

    return Instant.ofEpochSecond(random);
}
```

为了在多线程环境中实现更高的吞吐量，我们使用`[ThreadLocalRandom](/web/20220807065448/https://www.baeldung.com/java-thread-local-random)`来生成随机数。

我们可以验证生成的`Instant`总是大于或等于第一个`Instant and `小于第二个`Instant:`

```
Instant hundredYearsAgo = Instant.now().minus(Duration.ofDays(100 * 365));
Instant tenDaysAgo = Instant.now().minus(Duration.ofDays(10));
Instant random = RandomDateTimes.between(hundredYearsAgo, tenDaysAgo);
assertThat(random).isBetween(hundredYearsAgo, tenDaysAgo);
```

当然，请记住，测试随机性本质上是不确定的，通常不建议在实际应用中使用。

类似地，也可以在另一个之后或之前生成一个随机的`Instant `:

```
public static Instant after(Instant startInclusive) {
    return between(startInclusive, Instant.MAX);
}

public static Instant before(Instant upperExclusive) {
    return between(Instant.MIN, upperExclusive);
}
```

### 2.2.有界的`Date`

**其中一个`java.util.Date `构造函数获取纪元后的毫秒数。**所以，我们可以用同样的算法在另外两个之间产生一个随机的`Date `:

```
public static Date between(Date startInclusive, Date endExclusive) {
    long startMillis = startInclusive.getTime();
    long endMillis = endExclusive.getTime();
    long randomMillisSinceEpoch = ThreadLocalRandom
      .current()
      .nextLong(startMillis, endMillis);

    return new Date(randomMillisSinceEpoch);
}
```

同样，我们应该能够验证这种行为:

```
long aDay = TimeUnit.DAYS.toMillis(1);
long now = new Date().getTime();
Date hundredYearsAgo = new Date(now - aDay * 365 * 100);
Date tenDaysAgo = new Date(now - aDay * 10);
Date random = LegacyRandomDateTimes.between(hundredYearsAgo, tenDaysAgo);
assertThat(random).isBetween(hundredYearsAgo, tenDaysAgo);
```

### 2.3.无界`Instant`

为了生成一个完全随机的`Instant`，我们可以简单地生成一个随机整数并将其传递给`ofEpochSecond() `方法:

```
public static Instant timestamp() {
    return Instant.ofEpochSecond(ThreadLocalRandom.current().nextInt());
}
```

**使用 32 位秒，因为纪元时间产生更合理的随机时间，**因此我们在这里使用`nextInt() `方法**。**

此外，这个值应该仍然在 Java 可以处理的最小和最大可能`Instant `值之间:

```
Instant random = RandomDateTimes.timestamp();
assertThat(random).isBetween(Instant.MIN, Instant.MAX);
```

### 2.4.无界`Date`

与有界示例类似，我们可以将一个随机值传递给`Date's `构造函数来生成一个随机的`Date:`

```
public static Date timestamp() {
    return new Date(ThreadLocalRandom.current().nextInt() * 1000L);
}
```

由于构造函数的时间单位是毫秒，我们将 32 位纪元秒乘以 1000 转换成毫秒。

当然，该值仍在最小和最大可能`Date `值之间:

```
Date MIN_DATE = new Date(Long.MIN_VALUE);
Date MAX_DATE = new Date(Long.MAX_VALUE);
Date random = LegacyRandomDateTimes.timestamp();
assertThat(random).isBetween(MIN_DATE, MAX_DATE);
```

## 3.随机日期

到目前为止，我们生成了包含日期和时间成分的随机临时变量。类似地，**我们可以使用纪元日的概念来生成只包含日期部分的随机临时值。**

大纪元日等于自 1970 年 1 月 1 日以来的天数。因此，为了生成一个随机日期，**我们只需生成一个随机数，并使用该数作为纪元日。**

### 3.1.有界限的

我们需要一个只包含日期成分的时态抽象，所以`java.time.LocalDate `似乎是一个不错的选择:

```
public static LocalDate between(LocalDate startInclusive, LocalDate endExclusive) {
    long startEpochDay = startInclusive.toEpochDay();
    long endEpochDay = endExclusive.toEpochDay();
    long randomDay = ThreadLocalRandom
      .current()
      .nextLong(startEpochDay, endEpochDay);

    return LocalDate.ofEpochDay(randomDay);
}
```

这里我们使用`[toEpochDay()](https://web.archive.org/web/20220807065448/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/chrono/ChronoLocalDate.html#toEpochDay()) `方法将每个`LocalDate `转换成它对应的纪元日。同样，我们可以验证这种方法是正确的:

```
LocalDate start = LocalDate.of(1989, Month.OCTOBER, 14);
LocalDate end = LocalDate.now();
LocalDate random = RandomDates.between(start, end);
assertThat(random).isAfterOrEqualTo(start, end);
```

### 3.2.无限的

为了生成不考虑任何范围的随机日期，我们可以简单地生成一个随机纪元日:

```
public static LocalDate date() {
    int hundredYears = 100 * 365;
    return LocalDate.ofEpochDay(ThreadLocalRandom
      .current().nextInt(-hundredYears, hundredYears));
}
```

我们的随机日期生成器从纪元前后的 100 年中随机选择一天。同样，这背后的基本原理是生成合理的日期值:

```
LocalDate randomDay = RandomDates.date();
assertThat(randomDay).isBetween(LocalDate.MIN, LocalDate.MAX);
```

## 4.任意时间

类似于我们对日期所做的，我们可以生成只包含时间成分的随机临时变量。为了做到这一点，我们可以使用一天中的第二个概念。也就是说，**一个随机时间等于一个随机数，代表从一天开始的秒数。**

### 4.1.有界限的

`java.time.LocalTime `类是一个时态抽象，只封装了时间组件:

```
public static LocalTime between(LocalTime startTime, LocalTime endTime) {
    int startSeconds = startTime.toSecondOfDay();
    int endSeconds = endTime.toSecondOfDay();
    int randomTime = ThreadLocalRandom
      .current()
      .nextInt(startSeconds, endSeconds);

    return LocalTime.ofSecondOfDay(randomTime);
}
```

为了在两个其他时间之间生成一个随机时间，我们可以:

1.  在给定时间的第二天之间生成一个随机数
2.  使用该随机数创建一个随机时间

我们可以很容易地验证这种随机时间生成算法的行为:

```
LocalTime morning = LocalTime.of(8, 30);
LocalTime randomTime = RandomTimes.between(LocalTime.MIDNIGHT, morning);
assertThat(randomTime)
  .isBetween(LocalTime.MIDNIGHT, morning)
  .isBetween(LocalTime.MIN, LocalTime.MAX);
```

### 4.2.无限的

即使是无限制的时间值也应该在 00:00:00 到 23:59:59 的范围内，因此我们可以通过委托简单地实现这个逻辑:

```
public static LocalTime time() {
    return between(LocalTime.MIN, LocalTime.MAX);
}
```

## 5.结论

在本教程中，我们将随机日期和时间的定义简化为随机数。然后，我们看到这种减少如何帮助我们生成像时间戳、日期或时间一样的随机时间值。

像往常一样，示例代码可以在 [GitHub](https://web.archive.org/web/20220807065448/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-8-datetime-2) 上获得。