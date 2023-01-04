# 在 Java 中给日期添加小时

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-add-hours-date>

## 1.概观

在 Java 8 之前，`java.util.Date`是 Java 中最常用的表示日期时间值的类之一。

然后 Java 8 引入了`java.time.LocalDateTime` 和`java.time.ZonedDateTime.` Java 8 还允许我们使用 `java.time.Instant.` 在时间线上表示特定的时间

在本教程中，我们将学习在 Java 中给定的日期时间加上或减去 `n`小时。我们将首先查看一些标准的 Java 日期时间相关类，然后展示一些第三方选项。

要了解更多关于 Java 8 DateTime API 的信息，我们建议阅读本文。

## 2.`java.util.Date`

如果我们使用的是 Java 7 或更低版本，我们可以使用 [`java.util.Date`](https://web.archive.org/web/20220524003447/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Date.html) 和 [`java.util.Calendar`](https://web.archive.org/web/20220524003447/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Calendar.html) 类来进行大多数日期时间相关的处理。

让我们看看如何将`n`小时添加到给定的`Date`对象:

```java
public Date addHoursToJavaUtilDate(Date date, int hours) {
    Calendar calendar = Calendar.getInstance();
    calendar.setTime(date);
    calendar.add(Calendar.HOUR_OF_DAY, hours);
    return calendar.getTime();
}
```

注意 **`Calendar.HOUR_OF_DAY`指的是 24 小时制**。

上面的方法返回一个新的`Date`对象，它的值可以是 `(date + hours)`或 `(date – hours)`，这取决于我们是分别传递一个正值还是负值的`hours` 。

**假设我们有一个 Java 8 应用程序，但是我们仍然想用自己的方式处理`java.util.Date`实例。**

对于这种情况，我们可以选择采取以下替代方法:

1.  使用 `java.util.Date` `toInstant()`方法将`Date`对象转换为`java.time.Instant`实例
2.  使用`plus()`方法给`java.time.Instant`对象添加一个特定的`Duration`
3.  通过将`java.time.Instant`对象传递给`java.util.Date.from()`方法来恢复我们的 `java.util.Date`实例

让我们快速看一下这种方法:

```java
@Test
public void givenJavaUtilDate_whenUsingToInstant_thenAddHours() {
    Date actualDate = new GregorianCalendar(2018, Calendar.JUNE, 25, 5, 0)
      .getTime();
    Date expectedDate = new GregorianCalendar(2018, Calendar.JUNE, 25, 7, 0)
      .getTime();

    assertThat(Date.from(actualDate.toInstant().plus(Duration.ofHours(2))))
      .isEqualTo(expectedDate);
}
```

然而，请注意，对于 Java 8 或更高版本上的所有应用程序，总是建议使用新的 DateTime API。

## 3.`java.time.LocalDateTime/ZonedDateTime`

在 Java 8 或更高版本中，向任一个`[java.time.LocalDateTime](https://web.archive.org/web/20220524003447/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/LocalDateTime.html) or [java.time.ZonedDateTime](https://web.archive.org/web/20220524003447/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/ZonedDateTime.html)`实例添加时间非常简单，并且使用了`plusHours()`方法:

```java
@Test
public void givenLocalDateTime_whenUsingPlusHours_thenAddHours() {
    LocalDateTime actualDateTime = LocalDateTime
      .of(2018, Month.JUNE, 25, 5, 0);
    LocalDateTime expectedDateTime = LocalDateTime.
      of(2018, Month.JUNE, 25, 10, 0);

    assertThat(actualDateTime.plusHours(5)).isEqualTo(expectedDateTime);
}
```

如果我们想减去几个小时呢？

将负的小时值传递给`plusHours()`方法就可以了。但是，建议使用`minusHours()`方法:

```java
@Test
public void givenLocalDateTime_whenUsingMinusHours_thenSubtractHours() {
    LocalDateTime actualDateTime = LocalDateTime
      .of(2018, Month.JUNE, 25, 5, 0);
    LocalDateTime expectedDateTime = LocalDateTime
      .of(2018, Month.JUNE, 25, 3, 0);

    assertThat(actualDateTime.minusHours(2)).isEqualTo(expectedDateTime);

}
```

`java.time.ZonedDateTime`中的 `plusHours()`和`minusHours()`方法的工作方式完全相同。

## 4.`java.time.Instant`

我们知道，Java 8 DateTime API 中引入的 [`java.time.Instant`](https://web.archive.org/web/20220524003447/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/Instant.html) 表示时间线上的某个特定时刻。

**要给一个`Instant`对象添加一些小时，我们可以使用它的 `plus()`方法和一个 [`java.time.temporal.TemporalAmount`](https://web.archive.org/web/20220524003447/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/temporal/TemporalAmount.html) :**

```java
@Test
public void givenInstant_whenUsingAddHoursToInstant_thenAddHours() {
    Instant actualValue = Instant.parse("2018-06-25T05:12:35Z");
    Instant expectedValue = Instant.parse("2018-06-25T07:12:35Z");

    assertThat(actualValue.plus(2, ChronoUnit.HOURS))
      .isEqualTo(expectedValue);
}
```

类似地， `minus()`方法可用于减去特定的`TemporalAmount`。

## 5.Apache common〔t0〕

来自 Apache Commons Lang 库的 [`DateUtils`](https://web.archive.org/web/20220524003447/https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/time/DateUtils.html#addHours-java.util.Date-int-) 类公开了一个`static` `addHours()`方法:

```java
public static Date addHours(Date date, int amount)
```

该方法接受一个 `java.util.Date`对象和一个我们希望添加到它的`amount`，它的值可以是正的也可以是负的。

一个新的 `java.util.Date`对象作为结果返回:

```java
@Test
public void givenJavaUtilDate_whenUsingApacheCommons_thenAddHours() {
    Date actualDate = new GregorianCalendar(2018, Calendar.JUNE, 25, 5, 0)
      .getTime();
    Date expectedDate = new GregorianCalendar(2018, Calendar.JUNE, 25, 7, 0)
      .getTime();

    assertThat(DateUtils.addHours(actualDate, 2)).isEqualTo(expectedDate);
}
```

最新版本的 `Apache Commons Lang`可在 [Maven Central 获得。](https://web.archive.org/web/20220524003447/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.commons%22%20AND%20a%3A%22commons-lang3%22)

## 6.Joda 时间

[Joda Time](https://web.archive.org/web/20220524003447/http://www.joda.org/joda-time/) 是 Java 8 DateTime API 的替代品，并提供了自己的`DateTime`实现。

它的大多数`DateTime`相关类公开了 `plusHours()`和`minusHours()`方法来帮助我们从`DateTime`对象中增加或减少给定的小时数。

让我们看一个例子:

```java
@Test
public void givenJodaDateTime_whenUsingPlusHoursToDateTime_thenAddHours() {
    DateTime actualDateTime = new DateTime(2018, 5, 25, 5, 0);
    DateTime expectedDateTime = new DateTime(2018, 5, 25, 7, 0);

    assertThat(actualDateTime.plusHours(2)).isEqualTo(expectedDateTime);
}
```

我们可以在 [Maven Central](https://web.archive.org/web/20220524003447/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22joda-time%22%20AND%20a%3A%22joda-time%22) 轻松查看`Joda Time`的最新可用版本。

## 7.结论

在本教程中，我们介绍了几种从标准 Java 日期时间值中增加或减去给定小时数的方法。

我们还研究了一些第三方库作为替代。和往常一样，完整的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220524003447/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-date-operations-1)