# 如何用 Java 计算“时间前”

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-calculate-time-ago>

## 1.概观

计算相对时间和两个时间点之间的持续时间是软件系统中的常见用例。例如，我们可能希望向用户显示自从在社交媒体平台上发布新图片这样的事件以来已经过去了多长时间。这种“时间之前”文本的例子是“5 分钟之前”、“1 年之前”等。

虽然单词的语义和选择完全依赖于上下文，但总体思想是相同的。

在本教程中，我们将探索用 Java 计算时间的几种解决方案。**由于[在 Java 8](/web/20221128041338/https://www.baeldung.com/java-8-date-time-intro) 中引入了新的日期和时间 API，我们将分别讨论版本 7 和版本 8 的解决方案。**

## 2.Java 版本 7

Java 7 中有几个与时间相关的类。而且由于 Java 7 日期 API 的缺点，也有几个第三方的时间和日期库。

首先，我们用纯 Java 7 来计算“时间前”。

### 2.1.纯 Java 7

我们定义了一个`enum`来保存不同的时间粒度，并将它们转换为毫秒:

```java
public enum TimeGranularity {
    SECONDS {
        public long toMillis() {
            return TimeUnit.SECONDS.toMillis(1);
        }
    }, MINUTES {
        public long toMillis() {
            return TimeUnit.MINUTES.toMillis(1);
        }
    }, HOURS {
        public long toMillis() {
            return TimeUnit.HOURS.toMillis(1);
        }
    }, DAYS {
        public long toMillis() {
            return TimeUnit.DAYS.toMillis(1);
        }
    }, WEEKS {
        public long toMillis() {
            return TimeUnit.DAYS.toMillis(7);
        }
    }, MONTHS {
        public long toMillis() {
            return TimeUnit.DAYS.toMillis(30);
        }
    }, YEARS {
        public long toMillis() {
            return TimeUnit.DAYS.toMillis(365);
        }
    }, DECADES {
        public long toMillis() {
            return TimeUnit.DAYS.toMillis(365 * 10);
        }
    };

    public abstract long toMillis();
}
```

我们使用了`java.util.concurrent.TimeUnit` `enum`，这是一个强大的时间转换工具。使用`TimeUnit`枚举，我们为`TimeGranularity` `enum`的每个值覆盖了`toMillis()`抽象方法，这样它就返回了每个值对应的毫秒数。例如，对于“decade”，它返回 3650 天的毫秒数。

由于定义了`TimeGranularity` enum，我们可以定义两个方法。第一个函数接受一个`java.util.Date`对象和一个`TimeGranularity`实例，并返回一个“时间之前”的字符串:

```java
static String calculateTimeAgoByTimeGranularity(Date pastTime, TimeGranularity granularity) {
    long timeDifferenceInMillis = getCurrentTime() - pastTime.getTime();
    return timeDifferenceInMillis / granularity.toMillis() + " " + 
      granularity.name().toLowerCase() + " ago";
}
```

该方法将当前时间和给定时间的差值除以`TimeGranularity`值，单位为毫秒。因此，我们可以粗略地计算在指定的时间粒度中从给定时间开始已经过去的时间量。

为了获得当前时间，我们使用了`getCurrentTime()`方法。对于测试，我们返回一个固定的时间点，避免从本地机器读取时间。实际上，这个方法会使用`System.currentTimeMillis()` 或`LocalDateTime.now().`返回当前时间的真实值

让我们测试一下这个方法:

```java
Assert.assertEquals("5 hours ago", 
  TimeAgoCalculator.calculateTimeAgoByTimeGranularity(
    new Date(getCurrentTime() - (5 * 60 * 60 * 1000)), TimeGranularity.HOURS));
```

此外，我们还可以编写一个方法，自动检测最大合适的时间粒度，并返回一个更加人性化的输出:

```java
static String calculateHumanFriendlyTimeAgo(Date pastTime) {
    long timeDifferenceInMillis = getCurrentTime() - pastTime.getTime();
    if (timeDifferenceInMillis / TimeGranularity.DECADES.toMillis() > 0) {
        return "several decades ago";
    } else if (timeDifferenceInMillis / TimeGranularity.YEARS.toMillis() > 0) {
        return "several years ago";
    } else if (timeDifferenceInMillis / TimeGranularity.MONTHS.toMillis() > 0) {
        return "several months ago";
    } else if (timeDifferenceInMillis / TimeGranularity.WEEKS.toMillis() > 0) {
        return "several weeks ago";
    } else if (timeDifferenceInMillis / TimeGranularity.DAYS.toMillis() > 0) {
        return "several days ago";
    } else if (timeDifferenceInMillis / TimeGranularity.HOURS.toMillis() > 0) {
        return "several hours ago";
    } else if (timeDifferenceInMillis / TimeGranularity.MINUTES.toMillis() > 0) {
        return "several minutes ago";
    } else {
        return "moments ago";
    }
}
```

现在，让我们来看一个测试，看看一个使用示例:

```java
Assert.assertEquals("several hours ago", 
  TimeAgoCalculator.calculateHumanFriendlyTimeAgo(new Date(getCurrentTime() - (5 * 60 * 60 * 1000))));
```

根据上下文，我们可以使用不同的词，如“几个”、“几个”、“许多”，甚至是确切的值。

### 2.2.joda-时间图书馆

在 Java 8 发布之前， [Joda-Time](/web/20221128041338/https://www.baeldung.com/joda-time) 是 Java 中各种时间和日期相关操作的事实上的标准。我们可以使用 Joda-Time 库的三个类来计算“时间之前”:

*   `org.joda.time.Period`获取`org.joda.time.DateTime`的两个对象，并计算这两个时间点之间的差异
*   `org.joda.time.format.PeriodFormatter`定义了打印`Period`对象的格式
*   `org.joda.time.format.PeriodFormatuilder`这是一个创建自定义`PeriodFormatter`的生成器类

我们可以使用这三个类轻松获得现在和过去某个时间之间的准确时间:

```java
static String calculateExactTimeAgoWithJodaTime(Date pastTime) {
    Period period = new Period(new DateTime(pastTime.getTime()), new DateTime(getCurrentTime()));
    PeriodFormatter formatter = new PeriodFormatterBuilder().appendYears()
      .appendSuffix(" year ", " years ")
      .appendSeparator("and ")
      .appendMonths()
      .appendSuffix(" month ", " months ")
      .appendSeparator("and ")
      .appendWeeks()
      .appendSuffix(" week ", " weeks ")
      .appendSeparator("and ")
      .appendDays()
      .appendSuffix(" day ", " days ")
      .appendSeparator("and ")
      .appendHours()
      .appendSuffix(" hour ", " hours ")
      .appendSeparator("and ")
      .appendMinutes()
      .appendSuffix(" minute ", " minutes ")
      .appendSeparator("and ")
      .appendSeconds()
      .appendSuffix(" second", " seconds")
      .toFormatter();
    return formatter.print(period);
}
```

让我们来看一个用法示例:

```java
Assert.assertEquals("5 hours and 1 minute and 1 second", 
  TimeAgoCalculator.calculateExactTimeAgoWithJodaTime(new Date(getCurrentTime() - (5 * 60 * 60 * 1000 + 1 * 60 * 1000 + 1 * 1000)))); 
```

也有可能生成更人性化的输出:

```java
static String calculateHumanFriendlyTimeAgoWithJodaTime(Date pastTime) {
    Period period = new Period(new DateTime(pastTime.getTime()), new DateTime(getCurrentTime()));
    if (period.getYears() != 0) {
        return "several years ago";
    } else if (period.getMonths() != 0) {
        return "several months ago";
    } else if (period.getWeeks() != 0) {
        return "several weeks ago";
    } else if (period.getDays() != 0) {
        return "several days ago";
    } else if (period.getHours() != 0) {
        return "several hours ago";
    } else if (period.getMinutes() != 0) {
        return "several minutes ago";
    } else {
        return "moments ago";
    }
}
```

我们可以运行一个测试，看看这个方法返回一个更友好的“以前的时间”字符串:

```java
Assert.assertEquals("several hours ago", 
  TimeAgoCalculator.calculateHumanFriendlyTimeAgoWithJodaTime(new Date(getCurrentTime() - (5 * 60 * 60 * 1000)))); 
```

同样，我们可以使用不同的术语，如“一个”、“几个”或“几个”，这取决于用例。

### 2.3.Joda-Time `TimeZone`

使用 Joda-Time 库在计算“时间之前”时添加时区非常简单:

```java
String calculateZonedTimeAgoWithJodaTime(Date pastTime, TimeZone zone) {
    DateTimeZone dateTimeZone = DateTimeZone.forID(zone.getID());
    Period period = new Period(new DateTime(pastTime.getTime(), dateTimeZone), new DateTime(getCurrentTimeByTimeZone(zone)));
    return PeriodFormat.getDefault().print(period);
}
```

方法返回指定时区的当前时间值。对于测试，这个方法返回一个固定的时间点，但是在实践中，应该使用`Calendar.getInstance(zone).getTimeInMillis()` 或`LocalDateTime.now(zone).`返回当前时间的真实值

## 3.Java 8

[Java 8 引入了一个新的改进的日期和时间 API](/web/20221128041338/https://www.baeldung.com/java-8-date-time-intro) ，它采用了 Joda-Time 库中的许多想法。**我们可以用原生的`java.time.Duration`和`java.time.Period`类来计算“时间之前”:**

```java
static String calculateTimeAgoWithPeriodAndDuration(LocalDateTime pastTime, ZoneId zone) {
    Period period = Period.between(pastTime.toLocalDate(), getCurrentTimeByTimeZone(zone).toLocalDate());
    Duration duration = Duration.between(pastTime, getCurrentTimeByTimeZone(zone));
    if (period.getYears() != 0) {
        return "several years ago";
    } else if (period.getMonths() != 0) {
        return "several months ago";
    } else if (period.getDays() != 0) {
        return "several days ago";
    } else if (duration.toHours() != 0) {
        return "several hours ago";
    } else if (duration.toMinutes() != 0) {
        return "several minutes ago";
    } else if (duration.getSeconds() != 0) {
        return "several seconds ago";
    } else {
        return "moments ago";
    }
}
```

上面的代码片段支持时区，并且只使用原生 Java 8 API。

## 4.美丽时光图书馆

PrettyTime 是一个功能强大的库，专门提供“以前”的功能，支持 i18n】。此外，它高度可定制，易于使用，可以与 Java 版本 7 和 8 一起使用。

首先，让我们将它的[依赖项](https://web.archive.org/web/20221128041338/https://mvnrepository.com/artifact/org.ocpsoft.prettytime/prettytime/3.2.7.Final)添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>org.ocpsoft.prettytime</groupId>
    <artifactId>prettytime</artifactId>
    <version>3.2.7.Final</version>
</dependency>
```

现在，以一种人类友好的格式获取“以前的时间”是相当容易的:

```java
String calculateTimeAgoWithPrettyTime(Date pastTime) {
    PrettyTime prettyTime = new PrettyTime();
    return prettyTime.format(pastTime);
}
```

## 5.Time4J 库

最后，Time4J 是另一个在 Java 中操作时间和日期数据的优秀库。它有一个`PrettyTime`类，可以用来计算以前的时间。

让我们添加它的[依赖关系](https://web.archive.org/web/20221128041338/https://mvnrepository.com/artifact/net.time4j):

```java
<dependency>
    <groupId>net.time4j</groupId>
    <artifactId>time4j-base</artifactId>
    <version>5.9</version>
</dependency>
<dependency>
    <groupId>net.time4j</groupId>
    <artifactId>time4j-sqlxml</artifactId>
    <version>5.8</version>
</dependency>
```

添加这个依赖项后，计算以前的时间非常简单:

```java
String calculateTimeAgoWithTime4J(Date pastTime, ZoneId zone, Locale locale) {
    return PrettyTime.of(locale).printRelative(pastTime.toInstant(), zone);
}
```

和 PrettyTime 库一样，Time4J 也支持 i18n 开箱即用。

## 6.结论

在本文中，我们讨论了在 Java 中计算时间的不同方法。

纯 Java 和第三方库都有解决方案。由于 Java 8 中引入了新的日期和时间 API，纯 Java 解决方案对于 8 之前和之后的版本是不同的。

与往常一样，GitHub 上的[提供了示例的源代码。](https://web.archive.org/web/20221128041338/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-date-operations-2)