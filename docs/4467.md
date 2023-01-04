# 在 Java 中遍历一系列日期

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-iterate-date-range>

## 1。概述

在这个快速教程中，我们将研究在 Java 7、Java 8 和 Java 9 中使用开始和结束日期迭代一系列日期的几种方法。

## 2。Java 7

从 Java 7 开始，**我们将使用类`java.util.Date`保存日期值，使用`java.util.Calendar`从一个日期递增到下一个日期。**

让我们看一个使用简单的`while`循环以及`java.util.Date`和`java.util.Calendar`类的例子:

```
void iterateBetweenDatesJava7(Date start, Date end) {
    Date current = start;

    while (current.before(end)) {
        processDate(current);

        Calendar calendar = Calendar.getInstance();
        calendar.setTime(current);
        calendar.add(Calendar.DATE, 1);
        current = calendar.getTime();
    }
} 
```

## 3.Java 8

从 Java 8 开始，**我们可以使用新的 Java 8 日期 API** 。这给了我们自处理的、不可变的、流畅的、线程安全的对象。它还**允许我们编写更简洁的代码，而不需要像`java.util.Calendar`那样的助手类**来增加日期。

让我们使用一个简单的`for`循环、`LocalDate`类和方法`plusDays(1)`在日期范围内向前移动:

```
void iterateBetweenDatesJava8(LocalDate start, LocalDate end) {
    for (LocalDate date = start; date.isBefore(end); date = date.plusDays(1)) {
        processDate(date);
    }
}
```

这里值得注意的是**虽然流 API 从 Java 8 开始可用，但是直到 Java 9** 才可以使用日期 API 和流 API 在两个日期之间进行迭代。

关于 Java 8 日期 API 的更详细的解释，请查看本文。

## 4.Java 9+版本

**Java 9 引入了方法`datesUntil,`，让我们使用流 API** 从开始日期迭代到结束日期。

让我们更新示例代码，以利用这一特性:

```
void iterateBetweenDatesJava9(LocalDate start, LocalDate end) {
    start.datesUntil(end).forEach(this::processDate);
}
```

## 5。结论

正如我们在这篇快速文章中看到的，在 Java 中迭代一个日期范围是一项简单的任务。在使用 Java 8 和更高版本时尤其如此，我们可以使用 Date API 更容易地处理日期。

注意，在 Java 7 和更早的版本中，建议同时处理日期和时间，即使我们只使用日期。

然而，在 Java 8 和更高版本中，我们可以根据自己的需要，从日期 API(如`LocalDate,` `LocalDateTime,` 和其他选项)中选择合适的类。

当然，从 Java 9 开始，我们可以结合使用 Stream API 和 Date API 来迭代日期流。

和往常一样，代码片段可以在 GitHub 上找到[。](https://web.archive.org/web/20221206093536/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-9)