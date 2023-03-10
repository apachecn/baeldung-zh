# Java 中两个日期的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-date-difference>

## 1。概述

在这个快速教程中，我们将探索用 Java 计算两个日期之间差异的多种可能性。

## 延伸阅读:

## [Java 中的增量日期](/web/20220811124302/https://www.baeldung.com/java-increment-date)

An overview of various core and 3rd party methods for adding days to a date[Read more](/web/20220811124302/https://www.baeldung.com/java-increment-date) →

## [在 Java 中检查一个字符串是否是有效的日期](/web/20220811124302/https://www.baeldung.com/java-string-valid-date)

Have a look at different ways to check if a String is a valid date in Java[Read more](/web/20220811124302/https://www.baeldung.com/java-string-valid-date) →

## 2。核心 Java

### 2.1。使用`java.util.Date`找出天数的差异

让我们首先使用核心 Java APIs 进行计算，并确定两个日期之间的天数:

```java
@Test
public void givenTwoDatesBeforeJava8_whenDifferentiating_thenWeGetSix()
  throws ParseException {

    SimpleDateFormat sdf = new SimpleDateFormat("MM/dd/yyyy", Locale.ENGLISH);
    Date firstDate = sdf.parse("06/24/2017");
    Date secondDate = sdf.parse("06/30/2017");

    long diffInMillies = Math.abs(secondDate.getTime() - firstDate.getTime());
    long diff = TimeUnit.DAYS.convert(diffInMillies, TimeUnit.MILLISECONDS);

    assertEquals(6, diff);
}
```

### 2.2。使用`java.time.temporal.ChronoUnit`找出差异

Java 8 中的时间 API 使用`[TemporalUnit](https://web.archive.org/web/20220811124302/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/temporal/TemporalUnit.html)`接口表示日期时间单位，例如秒或天。

**每个单元都提供了一个名为 [`between`](https://web.archive.org/web/20220811124302/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/temporal/TemporalUnit.html#between(java.time.temporal.Temporal,java.time.temporal.Temporal)) 的方法的实现，用于计算两个时态对象之间的时间。**

例如，为了计算两个`LocalDateTime`之间的秒数:

```java
@Test
public void givenTwoDateTimesInJava8_whenDifferentiatingInSeconds_thenWeGetTen() {
    LocalDateTime now = LocalDateTime.now();
    LocalDateTime tenSecondsLater = now.plusSeconds(10);

    long diff = ChronoUnit.SECONDS.between(now, tenSecondsLater);

    assertEquals(10, diff);
}
```

[`ChronoUnit`](https://web.archive.org/web/20220811124302/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/temporal/ChronoUnit.html) 通过实现`TemporalUnit`接口提供了一组具体的时间单位。**强烈推荐`static import``ChronoUnit``enum`值，以达到更好的可读性**:

```java
import static java.time.temporal.ChronoUnit.SECONDS;

// omitted
long diff = SECONDS.between(now, tenSecondsLater);
```

此外，我们可以将任意两个兼容的时态对象传递给`between `方法，甚至是`ZonedDateTime`。

`ZonedDateTime`的伟大之处在于，即使他们被设置为不同的时区，计算也能工作:

```java
@Test
public void givenTwoZonedDateTimesInJava8_whenDifferentiating_thenWeGetSix() {
    LocalDateTime ldt = LocalDateTime.now();
    ZonedDateTime now = ldt.atZone(ZoneId.of("America/Montreal"));
    ZonedDateTime sixMinutesBehind = now
      .withZoneSameInstant(ZoneId.of("Asia/Singapore"))
      .minusMinutes(6);

    long diff = ChronoUnit.MINUTES.between(sixMinutesBehind, now);

    assertEquals(6, diff);
} 
```

### 2.3。使用`Temporal#until()`

任何 [`Temporal`](https://web.archive.org/web/20220811124302/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/temporal/Temporal.html) 对象，如`LocalDate or ZonedDateTime`，**提供了一个 [`until`](https://web.archive.org/web/20220811124302/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/temporal/Temporal.html#until(java.time.temporal.Temporal,java.time.temporal.TemporalUnit)) 方法来计算直到另一个`Temporal`的时间量，以指定的单位**来计算:

```java
@Test
public void givenTwoDateTimesInJava8_whenDifferentiatingInSecondsUsingUntil_thenWeGetTen() {
    LocalDateTime now = LocalDateTime.now();
    LocalDateTime tenSecondsLater = now.plusSeconds(10);

    long diff = now.until(tenSecondsLater, ChronoUnit.SECONDS);

    assertEquals(10, diff);
}
```

`Temporal#until`和`TemporalUnit#between `是相同功能的两个不同的 API。

### 2.4。使用`java.time.Duration`和`java.time.Period`和

在 Java 8 中，Time API 引入了两个新类: [`Duration` 和`Period`](/web/20220811124302/https://www.baeldung.com/java-8-date-time-intro#period) 。

**如果我们想要计算基于时间(小时、分钟或秒)的两个日期时间之间的差异，我们可以使用`Duration `类**:

```java
@Test
public void givenTwoDateTimesInJava8_whenDifferentiating_thenWeGetSix() {
    LocalDateTime now = LocalDateTime.now();
    LocalDateTime sixMinutesBehind = now.minusMinutes(6);

    Duration duration = Duration.between(now, sixMinutesBehind);
    long diff = Math.abs(duration.toMinutes());

    assertEquals(6, diff);
}
```

然而，**如果我们试图使用`Period`类来表示两个日期之间的差异，我们应该小心一个陷阱。**

一个例子将很快解释这个陷阱。

让我们使用`Period`类来计算两个日期之间的天数:

```java
@Test
public void givenTwoDatesInJava8_whenUsingPeriodGetDays_thenWorks()  {
    LocalDate aDate = LocalDate.of(2020, 9, 11);
    LocalDate sixDaysBehind = aDate.minusDays(6);

    Period period = Period.between(aDate, sixDaysBehind);
    int diff = Math.abs(period.getDays());

    assertEquals(6, diff);
}
```

如果我们运行上面的测试，它会通过。我们可能认为`Period`类可以方便地解决我们的问题。到目前为止，一切顺利。

如果这种方法在 6 天的差异内有效，我们不怀疑它也能在 60 天内有效。

因此，让我们将上面测试中的`6`改为`60`，看看会发生什么:

```java
@Test
public void givenTwoDatesInJava8_whenUsingPeriodGetDays_thenDoesNotWork() {
    LocalDate aDate = LocalDate.of(2020, 9, 11);
    LocalDate sixtyDaysBehind = aDate.minusDays(60);

    Period period = Period.between(aDate, sixtyDaysBehind);
    int diff = Math.abs(period.getDays());

    assertEquals(60, diff);
}
```

现在，如果我们再次运行测试，我们会看到:

```java
java.lang.AssertionError: 
Expected :60
Actual   :29
```

哎呀！为什么`Period`班报差为`29`天？

**这是因为`Period`类以“x 年，y 月，z 天”的格式表示基于日期的时间量。** **当我们调用它的`getDays() `方法时，它只返回“z 天”部分。**

因此，上面测试中的`period `对象持有值“0 年 1 个月 29 天”:

```java
@Test
public void givenTwoDatesInJava8_whenUsingPeriod_thenWeGet0Year1Month29Days() {
    LocalDate aDate = LocalDate.of(2020, 9, 11);
    LocalDate sixtyDaysBehind = aDate.minusDays(60);
    Period period = Period.between(aDate, sixtyDaysBehind);
    int years = Math.abs(period.getYears());
    int months = Math.abs(period.getMonths());
    int days = Math.abs(period.getDays());
    assertArrayEquals(new int[] { 0, 1, 29 }, new int[] { years, months, days });
}
```

**如果我们想用 Java 8 的 Time API 计算天数的差异，`ChronoUnit.DAYS.between() `方法是最直接的方法。**

## 3。外部库

### 3.1。`JodaTime`

我们也可以用`JodaTime`做一个相对简单的实现:

```java
<dependency>
    <groupId>joda-time</groupId>
    <artifactId>joda-time</artifactId>
    <version>2.9.9</version>
</dependency>
```

可以从 Maven Central 获得最新版本的 [Joda-time](https://web.archive.org/web/20220811124302/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22joda-time%22) 。

`LocalDate`案例:

```java
@Test
public void givenTwoDatesInJodaTime_whenDifferentiating_thenWeGetSix() {
    org.joda.time.LocalDate now = org.joda.time.LocalDate.now();
    org.joda.time.LocalDate sixDaysBehind = now.minusDays(6);

    long diff = Math.abs(Days.daysBetween(now, sixDaysBehind).getDays());
    assertEquals(6, diff);
} 
```

同样，用`LocalDateTime`:

```java
@Test
public void givenTwoDateTimesInJodaTime_whenDifferentiating_thenWeGetSix() {
    org.joda.time.LocalDateTime now = org.joda.time.LocalDateTime.now();
    org.joda.time.LocalDateTime sixMinutesBehind = now.minusMinutes(6);

    long diff = Math.abs(Minutes.minutesBetween(now, sixMinutesBehind).getMinutes());
    assertEquals(6, diff);
} 
```

### 3.2。`Date4J`

`Date4j`还提供了一个简单明了的实现——注意，在这种情况下，我们需要显式地提供一个`TimeZone`。

让我们从 Maven 依赖性开始:

```java
<dependency>
    <groupId>com.darwinsys</groupId>
    <artifactId>hirondelle-date4j</artifactId>
    <version>1.5.1</version>
</dependency>
```

这里有一个使用标准`DateTime`的快速测试:

```java
@Test
public void givenTwoDatesInDate4j_whenDifferentiating_thenWeGetSix() {
    DateTime now = DateTime.now(TimeZone.getDefault());
    DateTime sixDaysBehind = now.minusDays(6);

    long diff = Math.abs(now.numDaysFrom(sixDaysBehind));

    assertEquals(6, diff);
}
```

## 4。结论

在本文中，我们用普通 Java 和使用外部库举例说明了几种计算日期差异的方法(有和没有时间)。

The full source code of the article is available [over on GitHub](https://web.archive.org/web/20220811124302/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-date-operations-1).