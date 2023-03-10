# Joda-Time 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/joda-time>

## 1.介绍

在 Java 8 发布之前，Joda-Time 是使用最广泛的日期和时间处理库。它的目的是为处理日期和时间提供一个直观的 API，并解决 Java 日期/时间 API 中存在的设计问题。

**随着 Java 8 版本的发布，在 JDK 核心中引入了该库中实现的核心概念。**新的日期和时间 API 在`java.time`包( [JSR-310](https://web.archive.org/web/20221126215424/https://jcp.org/en/jsr/detail?id=310) )中找到。这些特性的概述可以在这篇[文章](/web/20221126215424/https://www.baeldung.com/java-8-date-time-intro)中找到。

Java 8 发布后，作者认为项目已经基本完成，建议尽可能使用 Java 8 API。

## 2.为什么使用 Joda-Time？

在 Java 8 之前，日期/时间 API 存在多个设计问题。

问题之一是，`Date`和`SimpleDateFormatter`类不是线程安全的。为了解决这个问题， **Joda-Time 使用不可变的类来处理日期和时间。**

`Date`类并不代表一个实际的日期，而是指定一个瞬间，精度为毫秒。`Date`中的年份从 1900 年开始，而大多数日期操作通常使用从 1970 年 1 月 1 日开始的纪元时间。

此外，`Date`的日、月和年偏移是违反直觉的。日从 0 开始，而月从 1 开始。要访问它们中的任何一个，我们必须使用`Calendar`类。 **Joda-Time 提供了一个干净流畅的 API 来处理日期和时间。**

Joda-Time 还提供了对八种日历系统的**支持，而 Java 只提供两种:公历–`java.util.GregorianCalendar`和日文–`java.util.JapaneseImperialCalendar`。**

## 3.设置

为了包含 Joda-Time 库的功能，我们需要从 [Maven Central](https://web.archive.org/web/20221126215424/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22joda-time%22%20AND%20a%3A%22joda-time%22) 添加以下依赖项:

```java
<dependency>
    <groupId>joda-time</groupId>
    <artifactId>joda-time</artifactId>
    <version>2.10</version>
</dependency>
```

## 4.库概述

Joda-Time 使用 **`org.joda.time`** 包中的类对**的日期和时间概念**进行建模。

在这些类别中，最常用的是:

*   `LocalDate`–表示没有时间的日期
*   `LocalTime`–表示不带时区的时间
*   `LocalDateTime`–表示没有时区的日期和时间
*   `Instant`–表示 Java 纪元 1970-01-01T00:00:00Z 的精确时间点，以毫秒为单位
*   `Duration`–代表两个时间点之间的持续时间，单位为毫秒
*   `Period`–类似于`Duration`，但允许访问日期和时间对象的单个组件，如年、月、日等。
*   `Interval`–代表两个瞬间之间的时间间隔

其他重要的特性是**日期解析器和格式化器**。这些都可以在 **`org.joda.time.format`** 包中找到。

**日历系统和时区**具体的类可以在**`org.joda.time.chrono``org.joda.time.tz`**包中找到。

让我们看一些例子，在这些例子中，我们使用 Joda-Time 的关键特性来处理日期和时间。

## 5.表示日期和时间

### 5.1.当前日期和时间

**当前日期，没有时间信息，**可以使用`now()`方法从**的`LocalDate`类**中获取:

```java
LocalDate currentDate = LocalDate.now();
```

当我们只需要当前时间，而不需要日期信息时，我们可以使用`LocalTime`类:

```java
LocalTime currentTime = LocalTime.now();
```

为了在不考虑时区的情况下获得当前日期和时间的**表示，我们可以使用`LocalDateTime`** :

```java
LocalDateTime currentDateAndTime = LocalDateTime.now();
```

现在，使用`currentDateAndTime`，我们可以将其转换为其他类型的对象建模日期和时间。

我们可以通过使用方法`toDateTime()`获得一个`DateTime`对象(它考虑了时区)。当不需要时间时，我们可以用方法`toLocalDate()`将它转换成一个`LocalDate`，当我们只需要时间时，我们可以用`toLocalTime()`获得一个`LocalTime`对象:

```java
DateTime dateTime = currentDateAndTime.toDateTime();
LocalDate localDate = currentDateAndTime.toLocalDate();
LocalTime localTime = currentDateAndTime.toLocalTime();
```

**以上所有方法都有一个重载方法，该方法接受一个`DateTimeZone`对象**来帮助我们表示指定时区的日期或时间:

```java
LocalDate currentDate = LocalDate.now(DateTimeZone.forID("America/Chicago"));
```

另外，Joda-Time 提供了与 Java 日期和时间 API 的良好集成。构造函数接受一个`java.util.Date`对象，我们也可以使用`toDate()`方法返回一个`java.util.Date`对象:

```java
LocalDateTime currentDateTimeFromJavaDate = new LocalDateTime(new Date());
Date currentJavaDate = currentDateTimeFromJavaDate.toDate();
```

### 5.2.自定义日期和时间

为了表示自定义的日期和时间，Joda-Time 为我们提供了几个构造函数。我们可以指定以下对象:

*   一个`Instant`
*   一个 Java `Date`对象
*   使用 ISO 格式的日期和时间的`String`表示
*   日期和时间的部分:年、月、日、小时、分钟、秒、毫秒

```java
Date oneMinuteAgoDate = new Date(System.currentTimeMillis() - (60 * 1000));
Instant oneMinutesAgoInstant = new Instant(oneMinuteAgoDate);

DateTime customDateTimeFromInstant = new DateTime(oneMinutesAgoInstant);
DateTime customDateTimeFromJavaDate = new DateTime(oneMinuteAgoDate);
DateTime customDateTimeFromString = new DateTime("2018-05-05T10:11:12.123");
DateTime customDateTimeFromParts = new DateTime(2018, 5, 5, 10, 11, 12, 123); 
```

我们可以定义自定义日期和时间的另一种方法是解析 ISO 格式的日期和时间的给定`String`表示:

```java
DateTime parsedDateTime = DateTime.parse("2018-05-05T10:11:12.123");
```

我们还可以通过定义自定义的`DateTimeFormatter`来解析日期和时间的自定义表示:

```java
DateTimeFormatter dateTimeFormatter
  = DateTimeFormat.forPattern("MM/dd/yyyy HH:mm:ss");
DateTime parsedDateTimeUsingFormatter
  = DateTime.parse("05/05/2018 10:11:12", dateTimeFormatter);
```

## 6.使用日期和时间

### 6.1.使用`Instant`

一个`Instant`表示从 1970-01-01T00:00:00Z 到给定时刻的毫秒数。例如，可以使用默认的构造函数或方法`now()`获得当前时刻:

```java
Instant instant = new Instant();
Instant.now();
```

为了创建一个自定义时刻的`Instant`，我们可以使用其中一个构造函数或者使用方法`ofEpochMilli()`和`ofEpochSecond()`:

```java
Instant instantFromEpochMilli
  = Instant.ofEpochMilli(milliesFromEpochTime);
Instant instantFromEpocSeconds
  = Instant.ofEpochSecond(secondsFromEpochTime);
```

构造函数接受一个代表 ISO 格式的日期和时间的`String`，一个代表 1970-01-01T00:00:00Z 之间的毫秒数的 Java `Date`或`long`值:

```java
Instant instantFromString
  = new Instant("2018-05-05T10:11:12");
Instant instantFromDate
  = new Instant(oneMinuteAgoDate);
Instant instantFromTimestamp
  = new Instant(System.currentTimeMillis() - (60 * 1000));
```

当日期和时间被表示为一个`String`时，我们可以选择使用我们想要的格式来解析`String`:

```java
Instant parsedInstant
  = Instant.parse("05/05/2018 10:11:12", dateTimeFormatter);
```

现在我们知道了`Instant`代表什么，以及如何创建一个，让我们看看如何使用它。

为了与`Instant`对象进行比较，我们可以使用`compareTo()`,因为它实现了`Comparable`接口，但是我们也可以使用`ReadableInstant`接口中提供的 Joda-Time API 方法，其中`Instant`也实现了:

```java
assertTrue(instantNow.compareTo(oneMinuteAgoInstant) > 0);
assertTrue(instantNow.isAfter(oneMinuteAgoInstant));
assertTrue(oneMinuteAgoInstant.isBefore(instantNow));
assertTrue(oneMinuteAgoInstant.isBeforeNow());
assertFalse(oneMinuteAgoInstant.isEqual(instantNow));
```

另一个有用的特性是 **`Instant`可以转换成一个`DateTime`对象或者一个 Java `Date`** 事件:

```java
DateTime dateTimeFromInstant = instant.toDateTime();
Date javaDateFromInstant = instant.toDate();
```

当我们需要访问日期和时间的一部分时，比如年、小时等等，我们可以使用`get()`方法并指定一个`DateTimeField`:

```java
int year = instant.get(DateTimeFieldType.year());
int month = instant.get(DateTimeFieldType.monthOfYear());
int day = instant.get(DateTimeFieldType.dayOfMonth());
int hour = instant.get(DateTimeFieldType.hourOfDay());
```

既然我们已经介绍了`Instant`类，那么让我们看看如何使用`Duration`、`Period`和`Interval`的一些例子。

### 6.2.使用`Duration`、`Period`和`Interval`

A `Duration`代表两个时间点之间的毫秒时间，或者在这种情况下，它可以是两个`Instants`。**当我们需要向另一个`Instant`添加或从另一个**减去特定的时间量时，我们会使用这个方法，而不考虑时间顺序和时区:

```java
long currentTimestamp = System.currentTimeMillis();
long oneHourAgo = currentTimestamp - 24*60*1000;
Duration duration = new Duration(oneHourAgo, currentTimestamp);
Instant.now().plus(duration);
```

此外，我们可以确定持续时间代表多少天、小时、分钟、秒或毫秒:

```java
long durationInDays = duration.getStandardDays();
long durationInHours = duration.getStandardHours();
long durationInMinutes = duration.getStandardMinutes();
long durationInSeconds = duration.getStandardSeconds();
long durationInMilli = duration.getMillis();
```

`Period`和`Duration`的主要区别在于 **`Period`是根据其日期和时间成分(年、月、小时等)来定义的。)并且不代表精确的毫秒数**。使用`Period`时，日期和时间计算**将考虑时区和夏令时**。

例如，将 1 个月的`Period`添加到 2 月 1 日将导致日期表示为 3 月 1 日。通过使用`Period`，该库将考虑闰年。

如果我们使用一个`Duration` we，结果将是不正确的，因为`Duration`代表一个固定的时间量，不考虑年表或时区:

```java
Period period = new Period().withMonths(1);
LocalDateTime datePlusPeriod = localDateTime.plus(period);
```

顾名思义，`Interval`表示由两个`Instant`对象表示的两个固定时间点之间的日期和时间间隔:

```java
Interval interval = new Interval(oneMinuteAgoInstant, instantNow);
```

当我们需要检查两个间隔是否重叠或计算它们之间的间隔时，该类非常有用。当不重叠时，`overlap()`方法将返回重叠的`Interval`或`null`:

```java
Instant startInterval1 = new Instant("2018-05-05T09:00:00.000");
Instant endInterval1 = new Instant("2018-05-05T11:00:00.000");
Interval interval1 = new Interval(startInterval1, endInterval1);

Instant startInterval2 = new Instant("2018-05-05T10:00:00.000");
Instant endInterval2 = new Instant("2018-05-05T11:00:00.000");
Interval interval2 = new Interval(startInterval2, endInterval2);

Interval overlappingInterval = interval1.overlap(interval2);
```

可以使用`gap()`方法计算间隔之间的差异，当我们想要知道一个间隔的结束是否等于另一个间隔的开始时，我们可以使用`abuts()`方法:

```java
assertTrue(interval1.abuts(new Interval(
  new Instant("2018-05-05T11:00:00.000"),
  new Instant("2018-05-05T13:00:00.000"))));
```

### 6.3.日期和时间操作

一些最常见的操作是加、减和转换日期和时间。该库为每个类`LocalDate`、`LocalTime`、`LocalDateTime`和`DateTime`提供了特定的方法。值得注意的是，这些类是不可变的，因此每次方法调用都会创建一个该类型的新对象。

让我们将`LocalDateTime`作为当前时刻，并尝试改变它的值:

```java
LocalDateTime currentLocalDateTime = LocalDateTime.now();
```

为了给`currentLocalDateTime`增加额外的一天，我们使用了`plusDays()`方法:

```java
LocalDateTime nextDayDateTime = currentLocalDateTime.plusDays(1);
```

我们也可以使用`plus()`方法给我们的`currentLocalDateTime:`添加一个`Period`或`Duration`

```java
Period oneMonth = new Period().withMonths(1);
LocalDateTime nextMonthDateTime = currentLocalDateTime.plus(oneMonth);
```

对于其他日期和时间组件，方法是类似的，例如，`plusYears()`用于添加额外的年份，plusSeconds()用于添加更多的秒，等等。

要从我们的`currentLocalDateTime`中减去一天，我们可以使用`minusDays()`方法:

```java
LocalDateTime previousDayLocalDateTime
  = currentLocalDateTime.minusDays(1);
```

此外，做日期和时间的计算，我们也可以，设置日期或时间的个别部分。例如，使用`withHourOfDay()`方法可以将小时设置为 10。以前缀`“with”`开头的其他方法可以用来设置日期或时间的组成部分:

```java
LocalDateTime currentDateAtHour10 = currentLocalDateTime
  .withHourOfDay(0)
  .withMinuteOfHour(0)
  .withSecondOfMinute(0)
  .withMillisOfSecond(0);
```

另一个重要的方面是我们可以从一个日期和时间类类型转换到另一个。为此，我们可以使用库提供的特定方法:

*   `toDateTime()`–将`LocalDateTime`转换为`DateTime`对象
*   `toLocalDate()`–将`LocalDateTime`转换为`LocalDate`对象
*   `toLocalTime() – converts LocalDateTime to a LocalTime object`
*   `toDate()`–将`LocalDateTime`转换为 Java `Date`对象

## 7.使用时区

Joda-Time 使我们能够轻松地适应不同的时区，并在它们之间进行转换。我们有一个抽象类`DateTimeZone`,用来表示一个时区的所有方面。

**Joda-Time 使用的默认时区是从`user.timezone` Java 系统属性中选择的。**库 API 让我们为每个类或计算分别指定应该使用哪个时区。例如，我们可以创建一个本地日期时间对象

当我们知道我们将在整个应用程序中使用特定时区时，我们可以设置默认时区:

```java
DateTimeZone.setDefault(DateTimeZone.UTC);
```

从现在开始，所有日期和时间操作，如果没有另外指定，将使用 UTC 时区表示。

要查看所有可用的时区，我们可以使用`getAvailableIDs():`方法

```java
DateTimeZone.getAvailableIDs()
```

当我们需要表示特定时区的日期或时间时，我们可以使用任何一个类`LocalTime`、`LocalDate`、`LocalDateTime`、`DateTime`，并在构造函数中指定`DateTimeZone`对象:

```java
DateTime dateTimeInChicago
  = new DateTime(DateTimeZone.forID("America/Chicago"));
DateTime dateTimeInBucharest
  = new DateTime(DateTimeZone.forID("Europe/Bucharest"));
LocalDateTime localDateTimeInChicago
  = new LocalDateTime(DateTimeZone.forID("America/Chicago"));
```

此外，当在这些类之间转换时，我们可以指定所需的时区。方法`toDateTime()`接受一个`DateTimeZone`对象，而`toDate()`接受 java.util.TimeZone 对象:

```java
DateTime convertedDateTime
  = localDateTimeInChicago.toDateTime(DateTimeZone.forID("Europe/Bucharest"));
Date convertedDate
  = localDateTimeInChicago.toDate(TimeZone.getTimeZone("Europe/Bucharest"));
```

## 8.结论

Joda-Time 是一个非常棒的库，它的主要目标是修复 JDK 中关于日期和时间操作的问题。它很快成为用于日期和时间处理的`de facto`库，最近 Java 8 中引入了它的主要概念。

**需要注意的是，作者认为它`“to be a largely finished project”`并建议迁移现有代码以使用 Java 8 实现。**

这篇文章的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221126215424/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-date-operations-1)