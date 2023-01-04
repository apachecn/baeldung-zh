# 在 Java 8 中向 LocalDate 添加日期时跳过周末

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-localdate-add-days-skip-weekends>

## 1.概观

在本教程中，我们将简要地看一下在 Java 8 中向`LocalDate` 实例添加日期时**跳过周末的算法。**

我们还将通过算法从`LocalDate`对象中减去**天，同时跳过周末。**

## 2.添加天数

在这个方法中，我们不断地向`LocalDate`对象添加一天，直到我们添加了所需的天数。在添加日期时，**我们检查新`LocalDate`实例的日期是周六还是周日**。

如果检查返回`true`，那么我们不会增加计数器的天数，直到那个时间点。然而，如果当天是工作日，那么我们增加计数器。

这样，我们不断增加天数，直到计数器等于应该增加的天数:

```java
public static LocalDate addDaysSkippingWeekends(LocalDate date, int days) {
    LocalDate result = date;
    int addedDays = 0;
    while (addedDays < days) {
        result = result.plusDays(1);
        if (!(result.getDayOfWeek() == DayOfWeek.SATURDAY || result.getDayOfWeek() == DayOfWeek.SUNDAY)) {
            ++addedDays;
        }
    }
    return result;
}
```

在上面的代码中，我们使用了`LocalDate`对象的`plusDays()`方法来给`result`对象添加天数。只有当那天是工作日时，我们才增加变量`addedDays `。当变量`addedDays`等于`days`变量时，我们停止向`result` `LocalDate`对象添加一天。

## 3.减去天数

类似地，我们可以使用`minusDays()`方法从`LocalDate`对象中减去天数，直到我们减去所需的天数。

为了实现这一点，**我们将为减去的天数保留一个计数器，该计数器仅在得出的日期是工作日时递增**:

```java
public static LocalDate subtractDaysSkippingWeekends(LocalDate date, int days) {
    LocalDate result = date;
    int subtractedDays = 0;
    while (subtractedDays < days) {
        result = result.minusDays(1);
        if (!(result.getDayOfWeek() == DayOfWeek.SATURDAY || result.getDayOfWeek() == DayOfWeek.SUNDAY)) {
            ++subtractedDays;
        }
    }
    return result;
}
```

从上面的实现中，我们可以看到，只有当`result LocalDate` 对象是工作日时，`subtractedDays `才会递增。使用 while 循环，我们减去天数，直到`subtractedDays`等于`days`变量。

## 4.结论

在这篇简短的文章中，我们**研究了跳过周末的`LocalDate`对象**的加减天数的算法。此外，我们还研究了它们在 Java 中的实现。

和往常一样，GitHub 上的[提供了工作示例的完整源代码。](https://web.archive.org/web/20220525123158/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-date-operations-2)