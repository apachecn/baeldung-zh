# 如何在 Java 中通过传递特定日期来确定星期几？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-get-day-of-week>

## 1.概观

在这个简短的教程中，我们将看到如何从 Java 日期中提取一个数字和文本形式的星期几。

## 2.问题

业务逻辑通常需要星期几。为什么？首先，工作日和周末的工作时间和服务水平不同。因此，将日期作为一个数字对于很多系统来说是必要的。但是我们也可能需要将这一天作为文本来显示。

那么，我们如何在 Java 中从日期中提取星期几呢？

## 3.用`java.util.Date`解决

`java.util.Date `从 Java 1.0 开始就是 Java 日期类。从 Java 版本 7 或更低版本开始的代码可能使用这个类。

### 3.1.以数字表示的星期几

首先，**我们使用`java.util.Calendar`提取日期作为数字**:

```java
public static int getDayNumberOld(Date date) {
    Calendar cal = Calendar.getInstance();
    cal.setTime(date);
    return cal.get(Calendar.DAY_OF_WEEK);
}
```

产生的**数字范围从 1(星期日)到 7(星期六)**。`Calendar`为此定义常数:`Calendar.SUNDAY`–`Calendar.SATURDAY`。

### 3.2.文本形式的星期几

现在我们**将日期提取为文本**。我们传入一个`Locale`来确定语言:

```java
public static String getDayStringOld(Date date, Locale locale) {
    DateFormat formatter = new SimpleDateFormat("EEEE", locale);
    return formatter.format(date);
}
```

这个**以您的语言**返回全天，例如英语的“星期一”或德语的“Montag”。

## 4.用`java.time.LocalDate`解决

[Java 8 革新了日期和时间处理](/web/20221208143830/https://www.baeldung.com/java-8-date-time-intro)，并引入了日期的`java.time.LocalDate`。所以，**只在 Java 版本 8 或更高版本上运行的 Java 项目应该使用这个类！**

### 4.1.以数字表示的星期几

**提取日期作为数字是微不足道的**现在:

```java
public static int getDayNumberNew(LocalDate date) {
    DayOfWeek day = date.getDayOfWeek();
    return day.getValue();
}
```

得到的数字仍然在 1 到 7 之间。但是这次，**周一是 1，周日是 7** ！一周中的**天有自己的`enum` — `DayOfWeek`** 。不出所料，`enum`值为`MONDAY`–`SUNDAY`。

### 4.2.文本形式的星期几

现在，我们再次将日期提取为文本。我们还传入一个`Locale`:

```java
public static String getDayStringNew(LocalDate date, Locale locale) {
    DayOfWeek day = date.getDayOfWeek();
    return day.getDisplayName(TextStyle.FULL, locale);
}
```

与`java.util.Date`一样，这将以所选语言返回全天。

## 5.结论

在本文中，我们从 Java 日期中提取了星期几。我们看到了如何使用`java.util.Date`和`java.time.LocalDate`返回数字和文本。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20221208143830/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-date-operations-2)