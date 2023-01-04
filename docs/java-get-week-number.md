# 从任意日期获取周数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-get-week-number>

## 1.介绍

在本文中，我们将研究几个可以在 Java 中使用的选项，以获取给定日期的周数。我们将从使用 Java 8 之前的类的遗留代码的几个选项开始。之后，我们将看看 Java 8 中引入的`java.time`包中更新的[日期时间 API](/web/20221004034605/https://www.baeldung.com/java-8-date-time-intro) 。

## 2.Java 8 之前

在 Java 8 之前，日期和时间的计算主要是使用`Date`和`Calendar`类来完成的。通常我们会创建一个`Calendar`，然后可以通过使用不同的常量从中提取我们需要的信息。

### 2.1.使用`Calendar`字段获取周数

让我们看看我们的第一个例子:

```java
Calendar calendar = Calendar.getInstance(locale); 
calendar.set(year, month, day); 
int weekOfYear = calendar.get(Calendar.WEEK_OF_YEAR);
```

我们简单地为给定的`Locale`创建一个`Calendar`实例，并设置年、月和日，最后，我们从`calendar`对象获得`WEEK_OF_YEAR`字段。这将返回当前年度内的周数。

现在，让我们看看如何从我们的一个单元测试中调用这个方法:

```java
@Test
public void givenDateUsingFieldsAndLocaleItaly_whenGetWeekNumber_thenWeekIsReturnedCorrectly() {
    Calendar calendar = Calendar.getInstance(Locale.ITALY);
    calendar.set(2020, 10, 22);

    assertEquals(47, calendar.get(Calendar.WEEK_OF_YEAR));
}
```

我们在采用这种方法时需要小心，因为`Calendar`类中的月份字段是从零开始的。**这意味着如果我们想要指定十二月，那么我们需要使用数字 11，这经常会导致混淆。**

### 2.2.使用`Locale`设置获取周数

在倒数第二个例子中，我们将看看对我们的`Calendar`应用一些额外的设置会有什么效果:

```java
Calendar calendar = Calendar.getInstance();
calendar.setFirstDayOfWeek(firstDayOfWeek);
calendar.setMinimalDaysInFirstWeek(minimalDaysInFirstWeek);
calendar.set(year, month, day);

int weekOfYear = calendar.get(Calendar.WEEK_OF_YEAR); 
```

`Calendar`类定义了两个方法:

*   `setFirstDayOfWeek`
*   `setMinimalDaysInFirstWeek`

这些方法对我们如何计算周数有影响。**通常，当创建`Calendar`时，这两个值都取自`Locale`。**但是也可以手动设置一周的第一天以及一年中第一周的最短天数。

### 2.3.`Locale` 差异

区域设置在如何计算周数方面起着重要作用:

```java
@Test
public void givenDateUsingFieldsAndLocaleCanada_whenGetWeekNumber_thenWeekIsReturnedCorrectly() {
    Calendar calendar = Calendar.getInstance(Locale.CANADA);
    calendar.set(2020, 10, 22);

    assertEquals(48, calendar.get(Calendar.WEEK_OF_YEAR));
} 
```

在这个单元测试中，我们只改变了`Calendar`的区域设置，使用`Locale.CANADA` 而不是`Locale.ITALY`，现在返回的周数是`48`而不是`47`。

两个结果都是正确的。**如前所述，这是因为每个`Locale`对`setFirstDayOfWeek`和`setMinimalDaysInFirstWeek`方法**有不同的设置。

## 3.Java 8 `Date Time` API

Java 8 为 [`Date`和`Time`](/web/20221004034605/https://www.baeldung.com/java-8-date-time-intro) 引入了新的 API，以解决旧的`java.util.Date`和`java.util.Calendar`的缺点。

在这一节中，我们将看看使用这个更新的 API 从日期中获取周数的一些选项。

### 3.1.使用数值获得周数

同样，正如我们之前在`Calendar`中看到的，我们也可以将年、月和日的值直接传递给`LocalDate`:

```java
LocalDate date = LocalDate.of(year, month, day);
int weekOfYear = date.get(WeekFields.of(locale).weekOfYear()); 
```

与 Java 8 之前的示例相比，我们的优势在于我们没有月字段从零开始的问题。

### 3.2.使用`Chronofield`获取周数

在最后一个例子中，我们将看到如何使用 **`ChronoField`枚举，它实现了`TemporalField`接口**:

```java
LocalDate date = LocalDate.of(year, month, day);
int weekOfYear = date.get(ChronoField.ALIGNED_WEEK_OF_YEAR); 
```

这个例子类似于我们之前看到的使用`Calendar.WEEK_OF_YEAR` `int`常量，但是使用了`ChronoField.ALIGNED_WEEK_OF_YEAR`。

## 4.结论

在这个快速教程中，我们展示了几种使用普通 Java 从日期中获取星期数的方法。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20221004034605/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-date-operations-2)