# 将 java.util.Date 转换为字符串

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-util-date-to-string>

## 1。概述

在本教程中，我们将展示如何在 Java 中将`Date`对象**转换为`String`对象。为此，我们将使用旧的`java.util.Date`类型以及 Java 8 中引入的新的`Date/Time` API。**

如果你想学习如何做相反的转换，即从`String`到`Date`类型，你可以在这里查看[这个教程。](/web/20220625220918/https://www.baeldung.com/java-string-to-date)

关于新`Date/Time` API 的更多细节，请参见[本相关教程](/web/20220625220918/https://www.baeldung.com/java-8-date-time-intro)。

## 2.将`java.util.Date`转换为`String`

尽管如果我们正在使用 Java 8，我们不应该使用`java.util.Date`，但有时我们别无选择(例如，我们从不受我们控制的库中接收`Date`对象)。

在这种情况下，我们有几种方法可以将`java.util.Date`转换成`String`。

### 2.1.准备`Date`对象

让我们首先声明日期的预期`String`表示，并定义期望的日期格式模式:

```java
private static final String EXPECTED_STRING_DATE = "Aug 1, 2018 12:00 PM";
private static final String DATE_FORMAT = "MMM d, yyyy HH:mm a";
```

现在我们需要我们想要转换的实际的`Date`对象。我们将使用一个`Calendar`实例来创建它:

```java
TimeZone.setDefault(TimeZone.getTimeZone("CET"));
Calendar calendar = Calendar.getInstance();
calendar.set(2018, Calendar.AUGUST, 1, 12, 0);
Date date = calendar.getTime();
```

我们已经将默认的`TimeZone`设置为`CET`以防止以后使用新 API 时出现问题。我们应该注意到**`Date`本身没有任何时区，但是它的`toString()`使用当前默认的时区**。

我们将在下面的所有例子中使用这个`Date`实例。

### 2.2.使用`SimpleDateFormat`类

在这个例子中，我们将使用`[SimpleDateFormat](https://web.archive.org/web/20220625220918/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/text/SimpleDateFormat.html)` 类的`format()`方法。让我们使用日期格式创建一个实例:

```java
DateFormat formatter = new SimpleDateFormat(DATE_FORMAT);
```

在此之后，我们可以格式化我们的日期，并将其与预期的输出进行比较:

```java
String formattedDate = formatter.format(date);

assertEquals(EXPECTED_STRING_DATE, formattedDate);
```

### 2.3.使用抽象的`DateFormat`类

正如我们已经看到的，`SimpleDateFormat`是抽象`DateFormat`类的子类。这个类提供了日期和时间格式化的各种方法。

我们将使用它来实现与上面相同的输出:

```java
String formattedDate = DateFormat
  .getDateTimeInstance(DateFormat.MEDIUM, DateFormat.SHORT)
  .format(date);
```

使用这种方法，我们传递样式模式——`MEDIUM`表示日期，而`SHORT`表示时间。

## 3.使用`Formatter`类

另一个简单的方法是使用`Formatter` 类来获得与前面例子中相同的`String`。

虽然这可能不是可读性最好的解决方案，**但这是一个线程安全的一行程序，非常有用，尤其是在多线程环境中**(我们应该记住`SimpleDateFormat`不是线程安全的):

```java
String formattedDate = String.format("%1$tb %1$te, %1$tY %1$tI:%1$tM %1$Tp", date);
```

我们使用`1$`来表示我们将只传递一个用于每个标志的参数。关于标志的详细解释可以在`Formatter`类的[日期/时间转换部分](https://web.archive.org/web/20220625220918/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Formatter.html#dt)中找到。

## 4.使用 Java 8 `Date/Time API`进行转换

Java 8 的`Date/Time` API 比`java.util.Date`和`java.util.Calendar `类强大得多，我们应该尽可能地使用它。让我们看看如何使用它将我们现有的`Date`对象转换成`String`。

这一次，我们将使用第 2.1 节中声明的`[DateTimeFormatter](https://web.archive.org/web/20220625220918/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/format/DateTimeFormatter.html)` 类及其`format()`方法，以及相同的日期模式:

```java
DateTimeFormatter formatter = DateTimeFormatter.ofPattern(DATE_FORMAT);
```

为了使用新的 API，我们需要**将我们的`Date`对象转换成一个`Instant`对象:**

```java
Instant instant = date.toInstant();
```

由于我们期望的`String`既有日期部分又有时间部分，**我们还需要将`Instant`对象转换成`LocalDateTime` :**

```java
LocalDateTime ldt = instant
  .atZone(ZoneId.of("CET"))
  .toLocalDateTime();
```

最后，我们可以很容易地得到格式化的`String`:

```java
String formattedDate = ldt.format(formatter);
```

## 5.结论

在本文中，我们举例说明了**将`java.util.Date`对象转换为`String`** 的几种方法。我们首先展示了如何使用旧的`java.util.Date`和`java.util.Calendar `类以及相应的日期格式化类来实现这一点。

然后我们使用了`Formatter`类，最后是 Java 8 日期/时间 API。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220625220918/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-conversions)