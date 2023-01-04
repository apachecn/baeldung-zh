# Java 8 日期/时间 API 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-8-date-time-intro>

## 1.概观

Java 8 为`Date`和`Time`引入了新的 API，以解决旧的`java.util.Date`和`java.util.Calendar`的缺点。

在本教程中，让我们从现有的`Date`和`Calendar`API 中的问题开始，讨论新的 Java 8 `Date`和`Time`API 如何解决这些问题。

我们还将看看新 Java 8 项目的一些核心类，它们是`java.time`包的一部分，比如`LocalDate`、`LocalTime`、 `LocalDateTime`、 `ZonedDateTime`、 `Period`、 `Duration` 以及它们支持的 API。

## 延伸阅读:

## [在 Spring 中使用日期参数](/web/20230102142017/https://www.baeldung.com/spring-date-parameters)

Learn how to work with Date parameters in Spring MVC[Read more](/web/20230102142017/https://www.baeldung.com/spring-date-parameters) →

## [在 Java 中检查一个字符串是否是有效的日期](/web/20230102142017/https://www.baeldung.com/java-string-valid-date)

Have a look at different ways to check if a String is a valid date in Java[Read more](/web/20230102142017/https://www.baeldung.com/java-string-valid-date) →

## 2.现有`Date`/`Time` API 的问题

*   **线程安全**—`Date`和`Calendar`类不是线程安全的，让开发人员处理难以调试的并发问题，并编写额外的代码来处理线程安全。相反，Java 8 中引入的新的`Date`和`Time`API 是不可变的，并且是线程安全的，因此消除了开发人员的并发性问题。
*   **API 设计和易于理解**—`Date`和`Calendar`API 设计不佳，执行日常操作的方法不充分。新的`Date` / `Time` API 是以 ISO 为中心的，在日期、时间、持续时间和周期方面遵循一致的领域模型。有各种各样的实用方法支持最常见的操作。
*   **`ZonedDate`和`Time`**–开发人员必须编写额外的逻辑来处理旧 API 的时区逻辑，而使用新 API，可以通过`Local`和`ZonedDate`/`Time`API 来处理时区。

## 3.使用`LocalDate`、`LocalTime`和`LocalDateTime`

最常用的类有`LocalDate`、`LocalTime`和`LocalDateTime`。顾名思义，它们代表观察者上下文中的本地日期/时间。

当不需要在上下文中明确指定时区时，我们主要使用这些类。作为本节的一部分，我们将介绍最常用的 API。

### 3.1。与`LocalDate`一起工作

`LocalDate`代表**一个不带时间的 ISO 格式的日期(yyyy-MM-dd)。**我们可以用它来存储生日和发薪日之类的日期。

可以从系统时钟创建当前日期的实例:

```
LocalDate localDate = LocalDate.now();
```

并且我们可以通过使用`of`方法或者`parse`方法得到代表特定日、月、年的 `LocalDate`。

例如，这些代码片段代表 2015 年 2 月 20 日的`LocalDate`:

```
LocalDate.of(2015, 02, 20);

LocalDate.parse("2015-02-20");
```

`LocalDate`提供了各种实用方法来获取各种信息。让我们快速浏览一下这些 API 方法。

以下代码片段获取当前本地日期并添加一天:

```
LocalDate tomorrow = LocalDate.now().plusDays(1);
```

此示例获取当前日期并减去一个月。注意它是如何接受一个`enum`作为时间单位的:

```
LocalDate previousMonthSameDay = LocalDate.now().minus(1, ChronoUnit.MONTHS);
```

在下面的两个代码示例中，我们解析日期“2016-06-12”并分别获取星期几和月几。注意返回值——第一个是表示`DayOfWeek`的对象，而第二个是表示月份顺序值的`int`:

```
DayOfWeek sunday = LocalDate.parse("2016-06-12").getDayOfWeek();

int twelve = LocalDate.parse("2016-06-12").getDayOfMonth();
```

我们可以测试日期是否在闰年，例如当前日期:

```
boolean leapYear = LocalDate.now().isLeapYear();
```

此外，可以确定一个日期与另一个日期的关系发生在另一个日期之前或之后:

```
boolean notBefore = LocalDate.parse("2016-06-12")
  .isBefore(LocalDate.parse("2016-06-11"));

boolean isAfter = LocalDate.parse("2016-06-12")
  .isAfter(LocalDate.parse("2016-06-11"));
```

最后，可以从给定的日期获得日期边界。

在下面的两个例子中，我们分别得到代表给定日期一天(2016-06-12T00:00)开始的`LocalDateTime`和代表一个月(2016-06-01)开始的`LocalDate`:

```
LocalDateTime beginningOfDay = LocalDate.parse("2016-06-12").atStartOfDay();
LocalDate firstDayOfMonth = LocalDate.parse("2016-06-12")
  .with(TemporalAdjusters.firstDayOfMonth());
```

现在让我们来看看我们是如何使用当地时间的。

### 3.2。与`LocalTime`一起工作

`LocalTime`表示没有日期的**时间。**

与`LocalDate`类似，我们可以从系统时钟或者通过使用`parse`和`of`方法创建一个`LocalTime`的实例。

我们现在将快速浏览一些常用的 API。

可以从系统时钟中创建电流`LocalTime`的实例:

```
LocalTime now = LocalTime.now();
```

我们可以通过解析一个字符串表示来创建一个代表上午 6:30 的`LocalTime` :

```
LocalTime sixThirty = LocalTime.parse("06:30");
```

工厂方法`of`也可以用来创建一个`LocalTime`。这段代码使用工厂方法创建代表早上 6:30 的`LocalTime`:

```
LocalTime sixThirty = LocalTime.of(6, 30);
```

让我们通过解析一个字符串并使用“plus”API 给它添加一个小时来创建一个`LocalTime`。结果将是代表上午 7:30 的`LocalTime`;

```
LocalTime sevenThirty = LocalTime.parse("06:30").plus(1, ChronoUnit.HOURS);
```

可以使用各种 getter 方法来获取特定的时间单位，如小时、分钟和秒:

```
int six = LocalTime.parse("06:30").getHour();
```

我们还可以检查某个特定时间是在另一个特定时间之前还是之后。此代码示例比较两个结果为真的`LocalTime`:

```
boolean isbefore = LocalTime.parse("06:30").isBefore(LocalTime.parse("07:30"));
```

最后通过`LocalTime` 类中的常数可以得到一天的最大、最小、中午时间。这在执行数据库查询以查找给定时间范围内的记录时非常有用。

例如，下面的代码表示 23:59:59.99:

```
LocalTime maxTime = LocalTime.MAX
```

现在让我们深入了解一下`LocalDateTime`。

### 3.3。与`LocalDateTime`一起工作

`LocalDateTime`用来表示**日期和时间的组合。**当我们需要日期和时间的组合时，这是最常用的类。

该类提供了各种 API。在这里，我们将看看一些最常用的。

类似于`LocalDate`和`LocalTime`，可以从系统时钟中获得`LocalDateTime` 的实例:

```
LocalDateTime.now();
```

下面的代码示例解释了如何使用工厂的“of”和“parse”方法创建实例。结果将是一个代表 2015 年 2 月 20 日，早上 6:30 的`LocalDateTime`实例；

```
LocalDateTime.of(2015, Month.FEBRUARY, 20, 06, 30);
```

```
LocalDateTime.parse("2015-02-20T06:30:00");
```

有一些实用程序 API 支持特定时间单位的加减，如日、月、年和分钟。

下面的代码演示了“加”和“减”的方法。这些 API 的行为与它们在`LocalDate`和`LocalTime`中的对应部分完全一样:

```
localDateTime.plusDays(1);
```

```
localDateTime.minusHours(2);
```

Getter 方法也可用于提取类似于 date 和 time 类的特定单位。给定上面的`LocalDateTime`实例，该代码示例将返回月份二月:

```
localDateTime.getMonth();
```

## 4.使用`ZonedDateTime` API

当我们需要处理特定时区的日期和时间时，Java 8 提供了 `ZonedDateTime` 。`ZoneId`是用于表示不同区域的标识符。大约有 40 个不同的时区，`ZoneId`代表它们如下。

在这里，我们为巴黎创造了一个`Zone`:

```
ZoneId zoneId = ZoneId.of("Europe/Paris"); 
```

我们可以获得一组所有区域 id:

```
Set<String> allZoneIds = ZoneId.getAvailableZoneIds();
```

`LocalDateTime`可以转换为特定的区域:

```
ZonedDateTime zonedDateTime = ZonedDateTime.of(localDateTime, zoneId);
```

`ZonedDateTime`提供了`parse` 方法来获取特定时区的日期时间:

```
ZonedDateTime.parse("2015-05-03T10:15:30+01:00[Europe/Paris]");
```

另一种处理时区的方法是使用`OffsetDateTime`。`OffsetDateTime`是带有偏移量的日期时间的不可变表示。此类存储所有日期和时间字段，精度为纳秒，以及与 UTC/格林威治的偏移量。

可以使用`ZoneOffset`创建 `OffSetDateTime` 实例。在这里，我们创建一个代表 2015 年 2 月 20 日上午 6:30 的`LocalDateTime`:

```
LocalDateTime localDateTime = LocalDateTime.of(2015, Month.FEBRUARY, 20, 06, 30);
```

然后我们通过创建一个`ZoneOffset`并为`localDateTime`实例设置来增加两个小时的时间:

```
ZoneOffset offset = ZoneOffset.of("+02:00");

OffsetDateTime offSetByTwo = OffsetDateTime
  .of(localDateTime, offset);
```

我们现在有一个 2015-02-20 06:30 +02:00 的`localDateTime`。

现在让我们继续讨论如何使用`Period`和`Duration`类修改日期和时间值。

## 5.使用`Period`和`Duration`

`Period` 类用年、月和日来表示时间量，而`Duration`类用秒和纳秒来表示时间量。

### 5.1。与`Period`一起工作

`Period`类广泛用于修改给定日期的值或获取两个日期之间的差值:

```
LocalDate initialDate = LocalDate.parse("2007-05-10");
```

我们可以通过使用`Period`来操纵`Date`:

```
LocalDate finalDate = initialDate.plus(Period.ofDays(5));
```

`Period`类有各种 getter 方法，比如`getYears`、 `getMonths` 和 `getDays` 来从`Period` 对象中获取值。

例如，当我们试图获得天数`:`的差值时，这将返回一个值为 5 的`int`

```
int five = Period.between(initialDate, finalDate).getDays();
```

我们可以使用`ChronoUnit.between`获得特定单位的两个日期之间的` Period`,例如日、月或年:

```
long five = ChronoUnit.DAYS.between(initialDate, finalDate);
```

此代码示例返回五天。

让我们继续看一看`Duration`类。

### 5.2。与`Duration`一起工作

与`Period,` 类似，`Duration class` 用于处理`Time.`

让我们创建一个上午 6:30 的`LocalTime`,然后添加一个 30 秒的持续时间来创建一个上午 6:30:30 的`LocalTime`;

```
LocalTime initialTime = LocalTime.of(6, 30, 0);

LocalTime finalTime = initialTime.plus(Duration.ofSeconds(30));
```

我们可以将两个瞬间之间的 `Duration`作为一个`Duration`或者一个特定的单位。

首先，我们使用`Duration`类的`between()`方法找到`finalTime`和`initialTime`之间的时差，并返回以秒为单位的时差:

```
long thirty = Duration.between(initialTime, finalTime).getSeconds();
```

在第二个例子中，我们使用`ChronoUnit`类的`between()`方法来执行相同的操作:

```
long thirty = ChronoUnit.SECONDS.between(initialTime, finalTime);
```

现在我们来看看如何将现有的`Date` 和`Calendar` 转换成新的`Date` / `Time`。

## 6.与`Date`和`Calendar`的兼容性

Java 8 增加了`toInstant()`方法，有助于将现有的`Date`和`Calendar`实例转换为新的日期和时间 API:

```
LocalDateTime.ofInstant(date.toInstant(), ZoneId.systemDefault());
LocalDateTime.ofInstant(calendar.toInstant(), ZoneId.systemDefault());
```

`LocalDateTime`可以从纪元秒构建。以下代码的结果将是一个代表 2016-06-13T11:34:50 的`LocalDateTime`:

```
LocalDateTime.ofEpochSecond(1465817690, 0, ZoneOffset.UTC);
```

现在让我们继续进行`Date`和`Time`格式化。

## 7.`Date`和`Time`格式

Java 8 为`Date`和`Time` :的简单格式化提供了 API

```
LocalDateTime localDateTime = LocalDateTime.of(2015, Month.JANUARY, 25, 6, 30);
```

此代码传递一个 ISO 日期格式来格式化本地日期，结果为 2015-01-25:

```
String localDateString = localDateTime.format(DateTimeFormatter.ISO_DATE);
```

`DateTimeFormatter`提供了各种标准的格式化选项。

自定义模式也可以提供给 format 方法，它在这里返回一个`LocalDate`作为 2015/01/25:

```
localDateTime.format(DateTimeFormatter.ofPattern("yyyy/MM/dd"));
```

我们可以将格式样式作为格式选项的一部分作为`SHORT`、`LONG`或`MEDIUM`来传递。

例如，这将给出表示 2015 年 1 月 25 日 06:30:00:

```
localDateTime
  .format(DateTimeFormatter.ofLocalizedDateTime(FormatStyle.MEDIUM)
  .withLocale(Locale.UK));
```

让我们来看看 Java 8 Core`Date`/`Time`API 的替代方案。

## 8.反向移植和替代选项

### 8.1。利用三个十工程

对于正在从 Java 7 或 Java 6 迁移到 Java 8 并希望使用日期和时间 API 的组织来说， [ThreeTen](https://web.archive.org/web/20230102142017/http://www.threeten.org/) 项目提供了反向移植功能。

开发人员可以使用这个项目中可用的类来实现与新的 Java 8 `Date`和`Time`API 相同的功能。一旦他们迁移到 Java 8，包就可以交换。

ThreeTen 项目的工件可以在 [Maven 中央存储库](https://web.archive.org/web/20230102142017/https://mvnrepository.com/artifact/org.threeten/threetenbp)中找到:

```
<dependency>
    <groupId>org.threeten</groupId>
    <artifactId>threetenbp</artifactId>
    <version>1.3.1</version>
</dependency>
```

### 8.2。joda-时间库

Java 8 `Date`和`Time`库的另一个替代品是 [Joda-Time](https://web.archive.org/web/20230102142017/http://www.joda.org/joda-time/) 库。事实上，Java 8 `Date` / `Time` API 已经由 Joda-Time library 的作者(Stephen Colebourne)和 Oracle 共同主导。这个库提供了 Java 8 `Date` / `Time`项目支持的几乎所有功能。

通过在我们的项目中包含下面的 pom 依赖项，可以在 [Maven Central](https://web.archive.org/web/20230102142017/https://mvnrepository.com/artifact/joda-time/joda-time) 中找到工件:

```
<dependency>
    <groupId>joda-time</groupId>
    <artifactId>joda-time</artifactId>
    <version>2.9.4</version>
</dependency> 
```

## 9。结论

Java 8 提供了一组丰富的 API，具有一致的 API 设计，便于开发。

上述文章的代码示例可以在 [Java 8 Date/Time](https://web.archive.org/web/20230102142017/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-8-datetime) git 存储库中找到。