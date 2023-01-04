# 如何获取两个日期之间的所有日期？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-between-dates>

## 1。概述

Java 8 中引入的新时间 API 使得不使用外部库处理日期和时间成为可能。

在这个简短的教程中，我们将了解在不同版本的 Java 中，获取两个日期之间的所有日期是如何变得越来越容易的。

## 2。使用 Java 7

在 Java 7 中，一种计算方法是使用`Calendar`实例。

首先，我们将得到没有时间的开始和结束日期。然后，我们将遍历这些，并使用`add`方法和`Calendar.Date`字段在每次迭代中添加一天，直到到达结束日期。

下面是演示它的代码——使用`Calendar`实例:

```java
public static List getDatesBetweenUsingJava7(Date startDate, Date endDate) {
  List datesInRange = new ArrayList<>();
  Calendar calendar = getCalendarWithoutTime(startDate);
  Calendar endCalendar = getCalendarWithoutTime(endDate);

  while (calendar.before(endCalendar)) {
    Date result = calendar.getTime();
    datesInRange.add(result);
    calendar.add(Calendar.DATE, 1);
  }

  return datesInRange;
}

private static Calendar getCalendarWithoutTime(Date date) {
  Calendar calendar = new GregorianCalendar();
  calendar.setTime(date);
  calendar.set(Calendar.HOUR, 0);
  calendar.set(Calendar.HOUR_OF_DAY, 0);
  calendar.set(Calendar.MINUTE, 0);
  calendar.set(Calendar.SECOND, 0);
  calendar.set(Calendar.MILLISECOND, 0);
  return calendar;
}
```

## 3。使用 Java 8

在 Java 8 中，我们现在可以创建一个连续无限的`Stream`日期，并且只提取相关的部分。不幸的是，**当一个谓词匹配到**时，没有办法终止一个无限的`Stream` ——这就是为什么我们需要计算这两天之间的天数，然后简单地计算`limit()`和`Stream:`

```java
public static List<LocalDate> getDatesBetweenUsingJava8(
  LocalDate startDate, LocalDate endDate) { 

    long numOfDaysBetween = ChronoUnit.DAYS.between(startDate, endDate); 
    return IntStream.iterate(0, i -> i + 1)
      .limit(numOfDaysBetween)
      .mapToObj(i -> startDate.plusDays(i))
      .collect(Collectors.toList()); 
} 
```

注意，首先，我们可以使用与`ChronoUnit`枚举的`DAYS`常数相关的`between`函数来获得两个日期之间的天数差。

然后我们创建一个整数`Stream`,表示从开始日期算起的天数。下一步，我们使用`plusDays()` API 将整数转换成`LocalDate`对象。

最后，我们将所有内容收集到一个 list 实例中。

## 4。使用 Java 9

最后，Java 9 提供了专门的计算方法:

```java
public static List<LocalDate> getDatesBetweenUsingJava9(
  LocalDate startDate, LocalDate endDate) {

    return startDate.datesUntil(endDate)
      .collect(Collectors.toList());
}
```

我们可以使用一个`LocalDate`类的专用`datesUntil`方法，通过一次方法调用获得两个日期之间的日期。`datesUntill`返回从其方法被调用的日期对象开始到作为方法参数给定的日期的有序日期`Stream`。

## 5。结论

在这篇简短的文章中，我们研究了如何使用不同版本的 Java 获取两个日期之间的所有日期。

我们讨论了在 Java 8 版本中引入的 Time API 如何使在日期文字上运行操作变得更加容易，而在 Java 9 中，只需调用`datesUntil.`就可以做到这一点

和往常一样，代码片段可以在 GitHub 上找到[。](https://web.archive.org/web/20220703150043/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-date-operations-1)