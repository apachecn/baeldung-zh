# 获取 Java 8 之前的当前日期

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-get-the-current-date-legacy>

## 1.介绍

在遗留系统中，当[新的日期和时间 API](/web/20220626074323/https://www.baeldung.com/java-8-date-time-intro) 和强烈推荐的 [Joda-Time 库](/web/20220626074323/https://www.baeldung.com/joda-time)都不可用时，我们可能需要处理日期。

在这个简短的教程中，我们将看看几种方法，看看如何在 Java 8 之前的系统中获取当前日期。

## 2.系统时间

当我们只需要一个代表当前日期和时间的数值时，我们可以使用系统时间。为了获得从`January 1, 1970 00:00:00 GMT`开始经过的毫秒数，我们可以使用返回一个`long`的`currentTimeMillis`方法:

```java
long elapsedMilliseconds = System.currentTimeMillis();
```

**当我们想要[更精确地测量经过的时间](/web/20220626074323/https://www.baeldung.com/java-measure-elapsed-time)时，我们可以使用`nanoTime`方法。**这将返回从一个固定但任意的时刻过去的纳秒值。

这个任意的时间对于 JVM 内部的所有调用都是相同的，所以返回的值只对计算多个调用`nanoTime`之间经过的纳秒数之差有用:

```java
long elapsedNanosecondsStart = System.nanoTime();
long elapsedNanoseconds = System.nanoTime() - elapsedNanosecondsStart;
```

## 3.`java.util`套餐

使用来自`java.util`包的类，我们可以表示一个时刻，通常是从`January 1, 1970 00:00:00 GMT`开始经过的毫秒数。

### 3.1.`java.util.Date`

我们可以用一个`Date`对象来表示一个特定的日期和时间。**这包含毫秒的精度和时区信息**。

虽然有许多可用的构造函数，但创建表示本地时区当前日期的`Date`对象的最简单方法是使用基本的构造函数:

```java
Date currentUtilDate = new Date();
```

现在让我们为特定的日期和时间创建一个`Date`对象。我们可以使用前面提到的构造函数，简单地传递毫秒值。

或者，我们可以使用`SimpleDateFormat`类来[将一个`String`](/web/20220626074323/https://www.baeldung.com/java-string-to-date) 值转换成一个实际的`Date`对象:

```java
SimpleDateFormat dateFormatter = new SimpleDateFormat("dd-MM-yyyy HH:mm:ss");
Date customUtilDate = dateFormatter.parse("30-01-2020 10:11:12");
```

我们可以使用多种日期模式来满足我们的需求。

### 3.2.`java.util.Calendar`

一个`Calendar`对象可以做一个`Date`所做的事情，并且它对于日期算术计算来说是**更好，因为它也可以接受一个 [`Locale`](/web/20220626074323/https://www.baeldung.com/java-8-localization)** 。我们可以将`Locale`指定为地理、政治或文化区域。

要获得当前日期，在没有指定`TimeZone`或`Locale`的情况下，我们可以使用`getInstance`方法:

```java
Calendar currentUtilCalendar = Calendar.getInstance();
```

对于从`Calendar`到`Date`的转换，我们可以简单地使用`getTime`方法:

```java
Date currentDate = Calendar.getInstance().getTime();
```

有趣的是， [`GregorianCalendar`](/web/20220626074323/https://www.baeldung.com/java-gregorian-calendar) 类是世界上使用最多的日历的实现。

## 4.`java.sql`套餐

接下来，我们将探索代表等效 SQL 对象的`java.util.Date`类的三个扩展。

### 4.1.`java.sql.Date`

使用`java.sql.Date`、**对象，我们无法访问时区信息，精度在日级别被截断**。为了表示今天，我们可以使用接受毫秒的`long`表示的构造函数:

```java
Date currentSqlDate = new Date(System.currentTimeMillis());
```

和以前一样，对于一个特定的日期，我们可以先用`SimpleDateFormat`类转换成`java.util.Date`，然后用`getTime`方法得到毫秒。然后，我们可以将这个值传递给`java.sql.Date`构造函数。

当`Date`的`String`表示与`**yyyy-[m]m-[d]d**`模式匹配时，我们可以简单地使用`valueOf`方法:

```java
Date customSqlDate = Date.valueOf("2020-01-30");
```

### 4.2.`java.sql.Time`

`java.sql.Time`对象为**提供了对小时、分钟和秒信息的访问**——同样，没有对时区的访问。让我们用毫秒来表示当前的`Time`:

```java
Time currentSqlTime = new Time(System.currentTimeMillis());
```

要使用`valueOf`方法指定时间，我们可以传入一个与`**hh:mm:ss**` 模式匹配的值:

```java
Time customSqlTime = Time.valueOf("10:11:12");
```

### 4.3.`java.sql.Timestamp`

在最后一节中，我们将使用`Timestamp` 类组合 SQL `Date`和`Time`信息。这使得我们可以将**精度降低到纳秒**。

让我们通过再次将当前毫秒数的`long`值传递给构造函数来创建一个`Timestamp`对象:

```java
Timestamp currentSqlTimestamp = new Timestamp(System.currentTimeMillis());
```

最后，让我们使用`valueOf`方法和所需的`**yyyy-[m]m-[d]d hh:mm:ss[.f…]**`模式创建一个新的定制`Timestamp`:

```java
Timestamp customSqlTimestamp = Timestamp.valueOf("2020-1-30 10:11:12.123456789");
```

## 5.结论

在这个简短的教程中，我们看到了如何在不使用 Java 8 或任何外部库的情况下获取当前日期和给定时刻的日期。

和往常一样，这篇文章的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220626074323/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-date-operations-2)