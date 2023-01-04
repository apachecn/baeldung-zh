# 用 Java 从日期中提取年、月和日

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-year-month-day>

## 1.概观

**在这个简短的教程中，我们将看看如何在 Java 中从给定的`Date `中提取`year`、`month`和`day `。**

我们将看看如何使用遗留的`java.util.Date` 类以及 Java 8 的新日期时间库来提取这些值。

在 Java 8 中，一个全新的日期和时间库被引入，原因有很多。除了其他优点，**新的库为提取`Year`、`Month`、`Day`等操作提供了更好的 API。来自给定的`Date`。**

如果你正在寻找一篇关于新的日期时间库的更详细的文章，请看这里的。

## 2.使用 Java 7

对于给定的`java.util.Date `提取单个字段，如`Year`、`Month`、*、*等。我们需要做的第一步是将它转换成`Calendar`实例:

```java
Date date = // the date instance
Calendar calendar = Calendar.getInstance();
calendar.setTime(date);
```

一旦我们有了一个`Calendar `实例，我们就可以直接调用它的`get`方法，并提供我们想要提取的特定字段。

我们可以使用`Calendar `中的常量来提取特定的字段。

### 2.1.获取年份

为了提取`year,`,我们可以通过将`Calendar.YEAR` 作为参数传递来调用`get `:

```java
calendar.get(Calendar.YEAR);
```

### 2.2.获取月份

类似地，为了提取`month,`,我们可以通过将`Calendar.MONTH `作为参数来调用`get `:

```java
calendar.get(Calendar.MONTH);
```

请注意，`Calendar `中的月份是零索引的；对于一月份，此方法将返回 0。

### 2.3.得到一天

最后，为了提取`day,`,我们通过将`Calendar.DAY_OF_MONTH `作为参数传递来调用`get `:

```java
calendar.get(Calendar.DAY_OF_MONTH);
```

## 3.使用 Java 8

**新的`java.time`包包含了许多可以用来表示`Date`的类。**

除了`Date`之外，每个类存储的附加信息也不同。

基本的`LocalDate `只包含日期信息，而`LocalDateTime`包含日期和时间信息。

类似地，更高级的类如`OffsetDateTime`和`ZonedDateTime`分别包含关于从`UTC`偏移的附加信息和关于`time-zone`的信息。

在任何情况下，所有这些类都支持提取年、月和日期信息的直接方法。

让我们探索这些方法，从`a LocalDate `实例名`localDate`中提取信息。

### 3.1.获取年份

提取`Year, LocalDate`只是提供了`a getYear `的方法:

```java
localDate.getYear();
```

### 3.2.获取月份

类似地，为了提取`Month, `,我们使用`getMonthValue ` API:

```java
localDate.getMonthValue();
```

与`Calendar`不同，`LocalDate`中的月份从 1 开始索引；对于一月份，它将返回 1。

### 3.3.得到一天

最后，为了提取`Day,`,我们有`getDayOfMonth`方法:

```java
localDate.getDayOfMonth();
```

## 4.结论

在这个快速教程中，我们探索了如何在 Java 中从`Date`中提取`Year`、`Month`和`Day`的整数值。

我们已经展示了如何使用旧的`Date`和`Calendar`类以及 Java8 的新日期时间库提取这些值。

本教程中使用的代码片段的完整源代码[可从 Github](https://web.archive.org/web/20220707143818/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-date-operations-1) 获得。