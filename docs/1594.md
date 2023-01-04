# 在 Java 中将字符串转换为日期

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-string-to-date>

## 1.概观

在本教程中，**我们将探索几种将`String` 对象转换成`Date`对象**的方法。我们将从 Java 8 中引入的新的`Date Time`API`java.time`开始，然后再看同样用于表示日期的旧的`java.util.Date`数据类型。

最后，我们将看看一些使用 Joda-Time 和 Apache Commons Lang `DateUtils`类进行转换的外部库。

## 延伸阅读:

## [将 java.util.Date 转换成字符串](/web/20221023102738/https://www.baeldung.com/java-util-date-to-string)

Learn several methods for converting Date objects to String objects in Java.[Read more](/web/20221023102738/https://www.baeldung.com/java-util-date-to-string) →

## [在 Java 中检查一个字符串是否是有效的日期](/web/20221023102738/https://www.baeldung.com/java-string-valid-date)

Have a look at different ways to check if a String is a valid date in Java[Read more](/web/20221023102738/https://www.baeldung.com/java-string-valid-date) →

## [在字符串和时间戳之间转换](/web/20221023102738/https://www.baeldung.com/java-string-to-timestamp)

Learn how to convert between String and Timestamp with a little help from LocalDateTime and Java 8.[Read more](/web/20221023102738/https://www.baeldung.com/java-string-to-timestamp) →

## 2.将`String`转换为`LocalDate`或`LocalDateTime`

[`LocalDate`](/web/20221023102738/https://www.baeldung.com/java-8-date-time-intro) 和`LocalDateTime`是不可变的日期时间对象，分别表示日期、日期和时间。默认情况下，Java 日期是 ISO-8601 格式，所以如果我们有任何表示这种格式的日期和时间的字符串，那么**我们可以直接使用这些类的`parse()`API**。

**有兴趣做 Java 开发实习生？看看 [Jooble](https://web.archive.org/web/20221023102738/https://jooble.org/jobs-java-developer-internship) ！**

### 2.1.使用`Parse` API

日期时间 API 提供了解析包含日期和时间信息的`String`的`parse()`方法。 **要将字符串对象转换为`LocalDate and LocalDateTime`对象，`String`必须根据 [ISO_LOCAL_DATE](https://web.archive.org/web/20221023102738/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/format/DateTimeFormatter.html#ISO_LOCAL_DATE) 或 [ISO_LOCAL_DATE_TIME](https://web.archive.org/web/20221023102738/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/format/DateTimeFormatter.html#ISO_LOCAL_DATE_TIME) 表示一个有效的日期或时间**。

否则，运行时会抛出一个`DateTimeParseException`。

在我们的第一个例子中，让我们将一个`String`转换成一个`java.time`。`LocalDate`:

```
LocalDate date = LocalDate.parse("2018-05-05");
```

可以使用与上述类似的方法将`String`转换为`java.time`。`LocalDateTime`:

```
LocalDateTime dateTime = LocalDateTime.parse("2018-05-05T11:50:55");
```

值得注意的是，`LocalDate`和`LocalDateTime`对象都是时区不可知的。但是，**当我们需要处理时区特定的日期和时间时，我们可以使用`ZonedDateTime`** `parse`方法直接得到一个时区特定的日期时间:

```
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss z");
ZonedDateTime zonedDateTime = ZonedDateTime.parse("2015-05-05 10:15:30 Europe/Paris", formatter);
```

现在让我们看看如何用自定义格式转换字符串。

### 2.2.使用带有自定义格式化程序的`Parse` API

将带有自定义日期格式的`String`转换成`Date`对象是 Java 中的一个普遍操作。

为此，**我们将使用`DateTimeFormatter`类，它提供了许多预定义的格式化程序，**，并允许我们定义一个格式化程序。

让我们从使用`DateTimeFormatter`的一个预定义格式化程序的例子开始:

```
String dateInString = "19590709";
LocalDate date = LocalDate.parse(dateInString, DateTimeFormatter.BASIC_ISO_DATE);
```

在下一个例子中，让我们创建一个应用“EEE，MMM d yyyy”格式的格式化程序此格式指定三个字符作为一周的完整日名称，一个数字表示一个月中的某一天，三个字符表示月，四个数字表示年。

该格式化程序识别字符串，如“`Fri,  3 Jan 2003″ or “Wed, 23 Mar 1994`”:

```
String dateInString = "Mon, 05 May 1980";
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("EEE, d MMM yyyy", Locale.ENGLISH);
LocalDate dateTime = LocalDate.parse(dateInString, formatter);
```

### 2.3.常见的日期和时间模式

让我们来看看一些常见的日期和时间模式:

*   `y –`年份(1996 年；96)
*   一年中的月份(七月；七月；07)
*   `d`–一个月中的第几天(1-31)
*   `E`–一周中的日名(星期五、星期天)
*   `a`**–**上午/下午标记(上午，下午)
*   `H`–一天中的小时(0-23)
*   `h`–上午/下午的小时数(1-12)
*   `m`–小时中的分钟(0-60)
*   `s`–以分钟为单位的秒(0-60)

关于我们可以用来指定解析模式的符号的完整列表，请点击[此处](https://web.archive.org/web/20221023102738/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/format/DateTimeFormatter.html#patterns)。

如果我们需要将`java.time` 日期转换成旧的`java.util.Date` 对象，请阅读[这篇](/web/20221023102738/https://www.baeldung.com/java-date-to-localdate-and-localdatetime)文章了解更多细节。

## 3.将`String`转换为`java.util.Date`

**在 Java 8 之前，Java 日期和时间机制是由旧的 APIs】、`java.util.Calendar`和 `java.util.TimeZone`、**提供的，我们有时仍然需要使用它们。

让我们看看如何将一个字符串转换成一个`java.util.Date`对象:

```
SimpleDateFormat formatter = new SimpleDateFormat("dd-MMM-yyyy", Locale.ENGLISH);

String dateInString = "7-Jun-2013";
Date date = formatter.parse(dateInString);
```

在上面的例子中，**我们首先需要通过传递描述日期和时间格式的模式来构造一个`SimpleDateFormat`对象**。

接下来我们需要调用传递日期`String`的`parse()`方法。如果传递的`String`参数与模式的格式不同，那么将抛出一个`ParseException`。

### 3.1.将时区信息添加到`java.util.Date`

**需要注意的是，`java.util.Date`没有时区**的概念，只代表自 Unix 纪元时间 1970-01-01T00:00:00Z 以来经过的秒数。

但是，当我们直接打印`Date`对象时，打印出来的总是 Java 默认的系统时区。

在最后一个示例中，我们将了解如何在添加时区信息的同时设置日期格式:

```
SimpleDateFormat formatter = new SimpleDateFormat("dd-M-yyyy hh:mm:ss a", Locale.ENGLISH);
formatter.setTimeZone(TimeZone.getTimeZone("America/New_York"));

String dateInString = "22-01-2015 10:15:55 AM"; 
Date date = formatter.parse(dateInString);
String formattedDateString = formatter.format(date);
```

我们还可以通过编程方式更改 JVM 时区，但不建议这样做:

```
TimeZone.setDefault(TimeZone.getTimeZone("GMT"));
```

## 4.外部库

既然我们已经很好地理解了如何使用 core Java 提供的新旧 API 将`String`对象转换为`Date`对象，那么让我们来看看一些外部库。

### 4.1.joda-时间图书馆

核心 Java `Date`和`Time`库的替代品是 [Joda-Time](https://web.archive.org/web/20221023102738/http://www.joda.org/joda-time/) 。尽管作者现在建议用户迁移到`java.time` (JSR-310)，如果这是不可能的，那么 **Joda-Time 库为处理日期和时间**提供了一个极好的选择。这个库提供了 Java 8 `Date Time`项目中支持的几乎所有功能。

神器可以在 [Maven Central](https://web.archive.org/web/20221023102738/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22joda-time%22) 上找到:

```
<dependency>
    <groupId>joda-time</groupId>
    <artifactId>joda-time</artifactId>
    <version>2.10</version>
</dependency>
```

这里有一个使用标准`DateTime`的简单例子:

```
DateTimeFormatter formatter = DateTimeFormat.forPattern("dd/MM/yyyy HH:mm:ss");

String dateInString = "07/06/2013 10:11:59";
DateTime dateTime = DateTime.parse(dateInString, formatter);
```

让我们看一个显式设置时区的例子:

```
DateTimeFormatter formatter = DateTimeFormat.forPattern("dd/MM/yyyy HH:mm:ss");

String dateInString = "07/06/2013 10:11:59";
DateTime dateTime = DateTime.parse(dateInString, formatter);
DateTime dateTimeWithZone = dateTime.withZone(DateTimeZone.forID("Asia/Kolkata"));
```

### 4.2 .Apache commons lang–dateutils

`DateUtils`类提供了许多**有用的工具，使得处理遗留`Calendar`和`Date`对象**变得更加容易。

commons-lang3 工件可从 [Maven Central](https://web.archive.org/web/20221023102738/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22commons-lang3%22) 获得:

```
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

让我们使用日期模式的`Array`将日期`String`转换成`java.util.Date` :

```
String dateInString = "07/06-2013";
Date date = DateUtils.parseDate(dateInString, 
  new String[] { "yyyy-MM-dd HH:mm:ss", "dd/MM-yyyy" });
```

## 5.结论

在本文中，我们展示了几种将字符串转换成不同类型的`Date`对象的方法(有时间和没有时间)，既有普通 Java 也有使用外部库的方法。

这篇文章的完整源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221023102738/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-datetime-string)