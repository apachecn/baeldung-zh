# 检查两个 Java 日期是否在同一天

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-check-two-dates-on-same-day>

## 1.概观

在这个快速教程中，**我们将学习几种不同的方法来检查两个`java.util.Date`物体是否有同一天的**。

我们将首先考虑使用核心 Java——即 Java 8 特性——的解决方案，然后再看几个 Java 8 之前的替代方案。

最后，我们还将看看一些外部库— **Apache Commons Lang、Joda-Time 和 Date4J** 。

## 2.核心 Java

类别 **`Date`代表特定的时间瞬间，精度为毫秒**。为了找出两个`Date`对象是否包含同一天，**我们需要检查年-月-日对于两个对象是否相同，并丢弃时间方面**。

### 2.1.使用`LocalDate`

有了 Java 8 的新[日期时间 API，我们就可以使用`LocalDate`对象了。这是一个不可变的对象，表示没有时间的日期。](/web/20220525125755/https://www.baeldung.com/java-8-date-time-intro)

让我们看看如何使用这个类来检查两个`Date` 对象是否是同一天:

```
public static boolean isSameDay(Date date1, Date date2) {
    LocalDate localDate1 = date1.toInstant()
      .atZone(ZoneId.systemDefault())
      .toLocalDate();
    LocalDate localDate2 = date2.toInstant()
      .atZone(ZoneId.systemDefault())
      .toLocalDate();
    return localDate1.isEqual(localDate2);
}
```

在这个例子中，**我们已经使用默认时区**将两个`Date`对象转换为`LocalDate`。一旦被转换，我们只需要使用`isEqual`方法来**检查`LocalDate`对象是否相等。**

因此，使用这种方法，我们将能够确定两个`Date`对象是否包含同一天。

### 2.2.使用`Instant`

在上面的例子中，当将`Date`对象转换为`LocalDate`对象时，我们使用`Instant`作为中间对象。Java 8 中引入的 **`Instant`，代表特定的时间点**。

我们可以直接用**将`Instant`对象截成天数单位**，这将时间字段值设置为零，然后我们可以对它们进行比较:

```
public static boolean isSameDayUsingInstant(Date date1, Date date2) {
    Instant instant1 = date1.toInstant()
      .truncatedTo(ChronoUnit.DAYS);
    Instant instant2 = date2.toInstant()
      .truncatedTo(ChronoUnit.DAYS);
    return instant1.equals(instant2);
}
```

### 2.3.使用`SimpleDateFormat`

从 Java 的早期版本开始，我们已经能够使用 [`SimpleDateFormat`](/web/20220525125755/https://www.baeldung.com/java-simple-date-format) 类在`Date`和`String`对象表示之间进行转换。这个类支持使用多种模式进行转换。**在我们的例子中，我们将使用模式“yyyyMMdd”**。

使用这个，我们将格式化`Date,`转换成一个`String`对象，然后使用标准的`equals`方法比较它们:

```
public static boolean isSameDay(Date date1, Date date2) {
    SimpleDateFormat fmt = new SimpleDateFormat("yyyyMMdd");
    return fmt.format(date1).equals(fmt.format(date2));
}
```

### 2.4.使用`Calendar`

`Calendar`类提供了获取特定时刻不同日期时间单位的值的方法。

首先，我们需要创建一个`Calendar`实例，并使用每个提供的日期设置`Calendar`对象的时间。然后我们可以分别查询和**比较**的年-月-日属性，以确定`Date`对象是否有同一天:

```
public static boolean isSameDay(Date date1, Date date2) {
    Calendar calendar1 = Calendar.getInstance();
    calendar1.setTime(date1);
    Calendar calendar2 = Calendar.getInstance();
    calendar2.setTime(date2);
    return calendar1.get(Calendar.YEAR) == calendar2.get(Calendar.YEAR)
      && calendar1.get(Calendar.MONTH) == calendar2.get(Calendar.MONTH)
      && calendar1.get(Calendar.DAY_OF_MONTH) == calendar2.get(Calendar.DAY_OF_MONTH);
}
```

## 3.外部库

既然我们已经很好地理解了如何使用 core Java 提供的新旧 API 来比较`Date`对象，那么让我们来看看一些外部库。

### 3.1 .Apache common lang〔t0〕

**`DateUtils`类提供了许多有用的工具，使得使用遗留`Calendar`和`Date`对象**变得更加容易。

从 [Maven Central](https://web.archive.org/web/20220525125755/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.commons%22%20AND%20a%3A%22commons-lang3%22) 可以获得 [Apache Commons Lang](https://web.archive.org/web/20220525125755/https://commons.apache.org/proper/commons-lang/) 工件:

```
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

那么我们可以简单地使用来自`DateUtils`的方法`isSameDay`:

```
DateUtils.isSameDay(date1, date2);
```

### 3.2.joda-时间图书馆

核心 Java `Date`和`Time`库的替代品是 [Joda-Time](/web/20220525125755/https://www.baeldung.com/joda-time) 。**这个广泛使用的[库](https://web.archive.org/web/20220525125755/http://www.joda.org/joda-time/)在处理日期和时间**时是一个很好的替代品。

神器可以在 [Maven Central](https://web.archive.org/web/20220525125755/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22joda-time%22%20AND%20a%3A%22joda-time%22) 上找到:

```
<dependency>
    <groupId>joda-time</groupId>
    <artifactId>joda-time</artifactId>
    <version>2.10</version>
</dependency>
```

在本库中， **`org.joda.time.LocalDate`代表没有时间**的日期。因此，我们可以从`java.util.Date`对象构建`LocalDate`对象，然后比较它们:

```
public static boolean isSameDay(Date date1, Date date2) {
    org.joda.time.LocalDate localDate1 = new org.joda.time.LocalDate(date1);
    org.joda.time.LocalDate localDate2 = new org.joda.time.LocalDate(date2);
    return localDate1.equals(localDate2);
}
```

### 3.3.Date4J 库

Date4j 还提供了一个我们可以使用的简单明了的实现。

同样，它也可以从 [Maven Central](https://web.archive.org/web/20220525125755/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.darwinsys%22%20AND%20a%3A%22hirondelle-date4j%22) 获得:

```
<dependency>
    <groupId>com.darwinsys</groupId>
    <artifactId>hirondelle-date4j</artifactId>
    <version>1.5.1</version>
</dependency>
```

使用这个库，我们需要从一个`java.util.Date`对象中**构造`DateTime`对象。然后我们可以简单地**使用`isSameDayAs`方法**:**

```
public static boolean isSameDay(Date date1, Date date2) {
    DateTime dateObject1 = DateTime.forInstant(date1.getTime(), TimeZone.getDefault());
    DateTime dateObject2 = DateTime.forInstant(date2.getTime(), TimeZone.getDefault());
    return dateObject1.isSameDayAs(dateObject2);
}
```

## 4.结论

在这个快速教程中，我们探索了几种检查两个`java.util.Date`对象是否包含同一天的方法。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20220525125755/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-date-operations-2)