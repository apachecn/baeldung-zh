# Java 中的临时调节器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-temporal-adjuster>

## 1。概述

在本教程中，我们将快速浏览一下`TemporalAdjuster`，并在一些实际场景中使用它。

Java 8 引入了一个新的处理日期和时间的库——`java.time` 和`TemporalAdjuster` 是其中的一部分。如果你想阅读更多关于 `java.time,` 的内容，请查看[这篇介绍性文章。](/web/20220626075605/https://www.baeldung.com/java-8-date-time-intro)

简单来说，`TemporalAdjuster`就是调整一个`Temporal`对象的策略。在进入`TemporalAdjuster`的用法之前，我们先来看看`Temporal`界面本身。

## 2。`Temporal`

根据我们将要使用的实现，`Temporal`定义了日期、时间或两者组合的表示。

有许多`Temporal`接口的实现，包括:

*   `LocalDate`–表示没有时区的日期
*   `LocalDateTime`–表示没有时区的日期和时间
*   `HijrahDate`–代表回历系统中的日期
*   `MinguoDate`–民国历法中的一个日期
*   `ThaiBuddhistDate`–代表泰国佛教日历系统中的一个日期

## 3。`TemporalAdjuster`

这个新库中包含的接口之一是`TemporalAdjuster`。

`TemporalAdjuster`是一个函数接口，在*temporal adjusts*类中有许多预定义的实现。该接口有一个名为 *adjustInto()* 的抽象方法，通过向其传递一个 *Temporal* 对象，可以在其任何实现中调用该方法。

*TemporalAdjuster* 允许我们执行复杂的日期操作。**例如**，我们可以获得下一个星期天的日期，当月的最后一天或者下一年的第一天。当然，我们可以使用旧的`java.util.Calendar`来实现。

然而，新的 API 使用其预定义的实现抽象出底层逻辑。欲了解更多信息，请访问 [Javadoc](https://web.archive.org/web/20220626075605/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/temporal/TemporalAdjuster.html) 。

## 4。预定义`TemporalAdjusters`

类`TemporalAdjusters`有许多预定义的静态方法，这些方法返回一个`TemporalAdjuster`对象，以许多不同的方式调整`Temporal`对象，不管它们可能是`Temporal`的什么实现。

以下是这些方法的简短列表及其快速定义:

*   `dayOfWeekInMonth()`–一周中第几天的调整器。例如三月的第二个星期二
*   `firstDayOfMonth()`–当月第一天的日期调整
*   `firstDayOfNextMonth()`–调整下个月第一天的日期
*   `firstDayOfNextYear()`–调整下一年第一天的日期
*   `firstDayOfYear()`–当前年度第一天的日期的调整器
*   `lastDayOfMonth()`–当月最后一天的日期调整
*   `nextOrSame()`–如果今天与要求的星期几相匹配，则调整下一个特定星期几或同一天的日期

正如我们所看到的，这些方法的名字是不言自明的。欲了解更多`TemporalAdjusters`，请访问 [Javadoc](https://web.archive.org/web/20220626075605/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/temporal/TemporalAdjusters.html) 。

**让我们从一个简单的例子**开始——我们可以使用`LocalDate.now()`从系统时钟中获取当前日期，而不是像例子中那样使用特定的日期。

但是，对于本教程，我们将使用一个固定的日期，这样当预期的结果改变时，测试就不会失败。让我们看看如何使用`TemporalAdjusters`类来获取 2017-07-08 之后的星期天的日期:

```java
@Test
public void whenAdjust_thenNextSunday() {
    LocalDate localDate = LocalDate.of(2017, 07, 8);
    LocalDate nextSunday = localDate.with(TemporalAdjusters.next(DayOfWeek.SUNDAY));

    String expected = "2017-07-09";

    assertEquals(expected, nextSunday.toString());
}
```

下面是我们获取当月最后一天的方法:

```java
LocalDate lastDayOfMonth = localDate.with(TemporalAdjusters.lastDayOfMonth());
```

## 5。定义定制`TemporalAdjuster`实现

我们还可以为`TemporalAdjuster`定义我们的定制实现。做这件事有两种不同的方法。

### 5.1。使用λ表达式

让我们看看如何使用`Temporal.with()`方法获得 2017-07-08 之后 14 天的日期:

```java
@Test
public void whenAdjust_thenFourteenDaysAfterDate() {
    LocalDate localDate = LocalDate.of(2017, 07, 8);
    TemporalAdjuster temporalAdjuster = t -> t.plus(Period.ofDays(14));
    LocalDate result = localDate.with(temporalAdjuster);

    String fourteenDaysAfterDate = "2017-07-22";

    assertEquals(fourteenDaysAfterDate, result.toString());
}
```

在本例中，使用 lambda 表达式，我们将`temporalAdjuster`对象设置为向保存日期(2017-07-08)的`localDate`对象添加 14 天。

让我们看看如何通过使用 lambda 表达式定义我们自己的`TemporalAdjuster`实现来获得 2017-07-08 之后的工作日的日期。但是，这一次，通过使用`ofDateAdjuster()`静态工厂方法:

```java
static TemporalAdjuster NEXT_WORKING_DAY = TemporalAdjusters.ofDateAdjuster(date -> {
    DayOfWeek dayOfWeek = date.getDayOfWeek();
    int daysToAdd;
    if (dayOfWeek == DayOfWeek.FRIDAY)
        daysToAdd = 3;
    else if (dayOfWeek == DayOfWeek.SATURDAY)
        daysToAdd = 2;
    else
        daysToAdd = 1;
    return today.plusDays(daysToAdd);
});
```

测试我们的代码:

```java
@Test
public void whenAdjust_thenNextWorkingDay() {
    LocalDate localDate = LocalDate.of(2017, 07, 8);
    TemporalAdjuster temporalAdjuster = NEXT_WORKING_DAY;
    LocalDate result = localDate.with(temporalAdjuster);

    assertEquals("2017-07-10", date.toString());
}
```

### 5.2。通过实现`TemporalAdjuster`接口

让我们看看如何通过实现`TemporalAdjuster`接口来编写一个自定义`TemporalAdjuster`来获取 2017-07-08 之后的工作日:

```java
public class CustomTemporalAdjuster implements TemporalAdjuster {

    @Override
    public Temporal adjustInto(Temporal temporal) {
        DayOfWeek dayOfWeek 
          = DayOfWeek.of(temporal.get(ChronoField.DAY_OF_WEEK));

        int daysToAdd;
        if (dayOfWeek == DayOfWeek.FRIDAY)
            daysToAdd = 3;
        else if (dayOfWeek == DayOfWeek.SATURDAY)
            daysToAdd = 2;
        else
            daysToAdd = 1;
        return temporal.plus(daysToAdd, ChronoUnit.DAYS);
    }
}
```

现在，让我们运行我们的测试:

```java
@Test
public void whenAdjustAndImplementInterface_thenNextWorkingDay() {
    LocalDate localDate = LocalDate.of(2017, 07, 8);
    CustomTemporalAdjuster temporalAdjuster = new CustomTemporalAdjuster();
    LocalDate nextWorkingDay = localDate.with(temporalAdjuster);

    assertEquals("2017-07-10", nextWorkingDay.toString());
}
```

## 6。结论

在本教程中，我们已经展示了什么是`TemporalAdjuster`，如何使用预定义的`TemporalAdjusters,`，以及如何以两种不同的方式实现我们的自定义`TemporalAdjuster`实现。

本教程的完整实现可以在 GitHub 上找到[。](https://web.archive.org/web/20220626075605/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-8-datetime)