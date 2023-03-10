# 在 Java 中获取不带时间的日期

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-date-without-time>

## 1。简介

在这个简短的教程中，我们将展示如何在 Java 中获得一个没有时间的`Date`。

我们将展示在 Java 8 之前和之后如何做到这一点，因为在 Java 8 中发布新的时间 API 之后，事情变得有些不同。

## 2。Java 8 之前

在 Java 8 之前，除非我们使用第三方库，比如 Joda-time，否则没有一种直接的方法可以获得一个没有时间的`Date`。

这是因为**Java 中的`Date`类是一个特定时刻的表示，用毫秒**表示。因此，这使得无法忽略`Date`上的时间。

在接下来的部分中，我们将展示一些常见的解决这个问题的方法。

### 2.1。使用`Calendar`

获得没有时间的`Date`的最常见方法之一是**使用`Calendar`类将时间设置为零**。通过这样做，我们得到一个干净的日期，时间设置在一天的开始。

让我们看看它的代码:

```java
public static Date getDateWithoutTimeUsingCalendar() {
    Calendar calendar = Calendar.getInstance();
    calendar.set(Calendar.HOUR_OF_DAY, 0);
    calendar.set(Calendar.MINUTE, 0);
    calendar.set(Calendar.SECOND, 0);
    calendar.set(Calendar.MILLISECOND, 0);

    return calendar.getTime();
}
```

如果我们调用这个方法，我们会得到这样一个日期:

```java
Sat Jun 23 00:00:00 CEST 2018
```

正如我们所见，它返回一个完整的日期，时间设置为零，但是我们不能忽略时间。

同样，为了确保在返回的`Date`中没有设置时间，我们可以创建下面的测试:

```java
@Test
public void whenGettingDateWithoutTimeUsingCalendar_thenReturnDateWithoutTime() {
    Date dateWithoutTime = DateWithoutTime.getDateWithoutTimeUsingCalendar();

    Calendar calendar = Calendar.getInstance();
    calendar.setTime(dateWithoutTime);
    int day = calendar.get(Calendar.DAY_OF_MONTH);

    calendar.setTimeInMillis(dateWithoutTime.getTime() + MILLISECONDS_PER_DAY - 1);
    assertEquals(day, calendar.get(Calendar.DAY_OF_MONTH));

    calendar.setTimeInMillis(dateWithoutTime.getTime() + MILLISECONDS_PER_DAY);
    assertNotEquals(day, calendar.get(Calendar.DAY_OF_MONTH));
}
```

正如我们所看到的，当我们将一天的毫秒数加 1 时，我们仍然得到同一天，但是当我们加一整天时，我们得到第二天。

### 2.2。使用格式化程序

另一种方法是通过**将一个`Date`格式化成一个没有时间的`String`，然后将那个`String`转换回一个`Date`** 。由于`String`被格式化时没有时间，转换后的`Date`将时间设置为零。

让我们实现它，看看它是如何工作的:

```java
public static Date getDateWithoutTimeUsingFormat() 
  throws ParseException {
    SimpleDateFormat formatter = new SimpleDateFormat(
      "dd/MM/yyyy");
    return formatter.parse(formatter.format(new Date()));
}
```

该实现返回的结果与上一节中显示的方法相同:

```java
Sat Jun 23 00:00:00 CEST 2018
```

同样，我们可以像以前一样使用测试来确保返回的`Date`中没有时间。

## 3。使用 Java 8

随着 Java 8 中新的时间 API 的发布，有了一种更简单的方法来获取没有时间的日期。这个新 API 带来的一个新特性是，有几个类可以处理有时间或没有时间的日期，甚至可以只处理时间。

出于本文的考虑，我们将把重点放在类`LocalDate`上，它代表了我们所需要的，一个没有时间的日期。

让我们看看这个例子:

```java
public static LocalDate getLocalDate() {
    return LocalDate.now();
}
```

这个方法返回一个带有这个日期表示的`LocalDate`对象:

```java
2018-06-23
```

正如我们所见，它返回的只是一个日期，**时间被完全忽略**。

同样，让我们像以前一样测试它，以确保该方法按预期工作:

```java
@Test
public void whenGettingLocalDate_thenReturnDateWithoutTime() {
    LocalDate localDate = DateWithoutTime.getLocalDate();

    long millisLocalDate = localDate
      .atStartOfDay()
      .toInstant(OffsetDateTime
        .now()
        .getOffset())
      .toEpochMilli();

    Calendar calendar = Calendar.getInstance();

    calendar.setTimeInMillis(
      millisLocalDate + MILLISECONDS_PER_DAY - 1);
    assertEquals(
      localDate.getDayOfMonth(), 
      calendar.get(Calendar.DAY_OF_MONTH));

    calendar.setTimeInMillis(millisLocalDate + MILLISECONDS_PER_DAY);
    assertNotEquals(
      localDate.getDayOfMonth(), 
      calendar.get(Calendar.DAY_OF_MONTH));
}
```

## 4。结论

在本文中，我们展示了如何在 Java 中获得一个没有时间的`Date`。我们首先展示了在 Java 8 之前以及使用 Java 8 时如何做到这一点。

正如我们在文章中实现的例子中看到的，使用 **Java 8 应该总是首选**，因为它有一个特定的表示来处理没有时间的日期。

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20220626075339/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-date-operations-1)