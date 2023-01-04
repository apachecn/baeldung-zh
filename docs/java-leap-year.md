# 在 Java 中查找闰年

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-leap-year>

## 1.概观

在本教程中，我们将用 Java 演示几种确定给定年份是否是闰年的方法。

闰年是能被 4 和 400 整除且没有余数的一年。因此，能被 100 整除但不能被 400 整除的年份不符合条件，尽管它们能被 4 整除。

## 2.**使用 Java-8 之前版本的日历 API**

从 Java 1.1 开始， [`GregorianCalendar`](/web/20220627093726/https://www.baeldung.com/java-gregorian-calendar) 类允许我们检查一年是否是闰年:

```java
public boolean isLeapYear(int year);
```

正如我们所料，如果给定的年份是闰年，这个方法返回`true`,对于非闰年，返回`false`。

**公元前(公元前)年需要作为负值**传递，计算为 1—`year`。例如，公元前 3 年表示为-2，因为 1–3 =-2。

## 3.**使用 Java 8+日期/时间 API**

Java 8 引入了`java`。`time`用一个好得多的[日期时间 API](/web/20220627093726/https://www.baeldung.com/java-8-date-time-intro) 打包。

`java`中的 [`Year`](https://web.archive.org/web/20220627093726/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/Year.html) 类。`time`包有一个`static`方法来检查给定的年份是否是闰年:

```java
public static boolean isLeap(long year);
```

它还有一个实例方法来做同样的事情:

```java
public boolean isLeap();
```

## 4.**使用 Joda-Time API**

[Joda-Time API](/web/20220627093726/https://www.baeldung.com/joda-time) 是 Java 项目中用于日期和时间实用程序的最常用的第三方库之一。从 Java 8 开始，**这个库就处于可维护状态**，正如 [Joda-Time GitHub 源代码库](https://web.archive.org/web/20220627093726/https://github.com/JodaOrg/joda-time#joda-time)中提到的。

没有预定义的 API 方法来查找 Joda-Time 中的闰年。然而，我们可以使用它们的`[LocalDate](https://web.archive.org/web/20220627093726/https://www.joda.org/joda-time/apidocs/org/joda/time/LocalDate.html)`和`[Days](https://web.archive.org/web/20220627093726/https://www.joda.org/joda-time/apidocs/org/joda/time/Days.html)`类来检查闰年:

```java
LocalDate localDate = new LocalDate(2020, 1, 31);
int numberOfDays = Days.daysBetween(localDate, localDate.plusYears(1)).getDays();

boolean isLeapYear = (numberOfDays > 365) ? true : false;
```

## 5.结论

在本教程中，我们已经了解了什么是闰年，查找闰年的逻辑，以及我们可以用来检查闰年的几个 Java APIs。

和往常一样，代码片段可以在 GitHub 上找到。