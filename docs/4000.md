# 迁移到新的 Java 8 日期时间 API

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/migrating-to-java-8-date-time-api>

## 1.概观

在本教程中，您将学习如何重构代码，以便利用 Java 8 中引入的新日期时间 API。

## 2.新 API 一览

在 Java 中处理日期曾经很困难。JDK 提供的旧数据库只包括三个类:`java.util.Date, java.util.Calendar` 和`java.util.Timezone`。

这些只适合最基本的任务。对于任何稍微复杂的东西，开发人员要么使用第三方库，要么编写大量定制代码。

Java 8 引入了一个全新的日期时间 API ( `java.util.time.*`)，它大致基于流行的 Java 库 JodaTime。这个新的 API 极大地简化了日期和时间处理，并修复了旧的日期库的许多缺点。

### 1.1。API 清晰度

新 API 的第一个优势是**清晰**——API 非常清晰、简洁且易于理解。它没有在旧库中发现的许多不一致，例如字段编号(在日历月中是从零开始的，但是一周中的日子是从一开始的)。

### 1.2。API 灵活性

另一个优势是灵活性-**处理多种时间表现**。旧的日期库只包含一个时间表示类— `java.util.Date`,尽管它的名字是这样的，但它实际上是一个时间戳。它只存储自 Unix 纪元以来经过的毫秒数。

新的 API 有许多不同的时间表示，每一种都适用于不同的用例:

*   `Instant`–代表一个时间点(时间戳)
*   `LocalDate`–代表日期(年、月、日)
*   `LocalDateTime`–与`LocalDate`相同，但包括纳秒精度的时间
*   `OffsetDateTime`–与`LocalDateTime`相同，但有时区偏移
*   `LocalTime`–纳秒精度的时间，无日期信息
*   `ZonedDateTime`–与`OffsetDateTime`相同，但包括一个时区 ID
*   `OffsetLocalTime`–与`LocalTime`相同，但有时区偏移
*   `MonthDay`–月和日，无年份和时间
*   `YearMonth`–月份和年份，没有日期和时间
*   `Duration`–以秒、分、小时表示的时间。具有纳秒精度
*   `Period`–以日、月和年表示的时间量

### 1.3。不变性和线程安全

另一个优点是 Java 8 日期时间 API 中的所有时间表示都是不可变的，因此是线程安全的。

所有的变异方法都返回一个新的副本，而不是修改原始对象的状态。

像`java.util.Date` 这样的旧类不是线程安全的，可能会引入非常微妙的并发错误。

### 1.4。方法链接

所有的变异方法可以链接在一起，允许在一行代码中实现复杂的转换。

```
ZonedDateTime nextFriday = LocalDateTime.now()
  .plusHours(1)
  .with(TemporalAdjusters.next(DayOfWeek.FRIDAY))
  .atZone(ZoneId.of("PST")); 
```

## 2。示例

下面的例子将演示如何使用新旧 API 执行常见任务。

**获取当前时间**

```
// Old
Date now = new Date();

// New
ZonedDateTime now = ZonedDateTime.now(); 
```

**代表特定时间**

```
// Old
Date birthDay = new GregorianCalendar(1990, Calendar.DECEMBER, 15).getTime();

// New
LocalDate birthDay = LocalDate.of(1990, Month.DECEMBER, 15); 
```

**提取特定字段**

```
// Old
int month = new GregorianCalendar().get(Calendar.MONTH);

// New
Month month = LocalDateTime.now().getMonth(); 
```

**加减时间**

```
// Old
GregorianCalendar calendar = new GregorianCalendar();
calendar.add(Calendar.HOUR_OF_DAY, -5);
Date fiveHoursBefore = calendar.getTime();

// New
LocalDateTime fiveHoursBefore = LocalDateTime.now().minusHours(5); 
```

**更改特定字段**

```
// Old
GregorianCalendar calendar = new GregorianCalendar();
calendar.set(Calendar.MONTH, Calendar.JUNE);
Date inJune = calendar.getTime();

// New
LocalDateTime inJune = LocalDateTime.now().withMonth(Month.JUNE.getValue()); 
```

**截断**

截断会重置所有小于指定字段的时间字段。在下面的例子中，分钟和下面的一切都将被设置为零

```
// Old
Calendar now = Calendar.getInstance();
now.set(Calendar.MINUTE, 0);
now.set(Calendar.SECOND, 0);
now.set(Calendar.MILLISECOND, 0);
Date truncated = now.getTime();

// New
LocalTime truncated = LocalTime.now().truncatedTo(ChronoUnit.HOURS); 
```

**时区转换**

```
// Old
GregorianCalendar calendar = new GregorianCalendar();
calendar.setTimeZone(TimeZone.getTimeZone("CET"));
Date centralEastern = calendar.getTime();

// New
ZonedDateTime centralEastern = LocalDateTime.now().atZone(ZoneId.of("CET")); 
```

**获取两个时间点之间的时间跨度**

```
// Old
GregorianCalendar calendar = new GregorianCalendar();
Date now = new Date();
calendar.add(Calendar.HOUR, 1);
Date hourLater = calendar.getTime();
long elapsed = hourLater.getTime() - now.getTime();

// New
LocalDateTime now = LocalDateTime.now();
LocalDateTime hourLater = LocalDateTime.now().plusHours(1);
Duration span = Duration.between(now, hourLater); 
```

**时间格式化和解析**

DateTimeFormatter 替代了旧的 SimpleDateFormat，它是线程安全的，并提供了附加功能。

```
// Old
SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
Date now = new Date();
String formattedDate = dateFormat.format(now);
Date parsedDate = dateFormat.parse(formattedDate);

// New
LocalDate now = LocalDate.now();
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd");
String formattedDate = now.format(formatter);
LocalDate parsedDate = LocalDate.parse(formattedDate, formatter); 
```

**一个月的天数**

```
// Old
Calendar calendar = new GregorianCalendar(1990, Calendar.FEBRUARY, 20);
int daysInMonth = calendar.getActualMaximum(Calendar.DAY_OF_MONTH);

// New
int daysInMonth = YearMonth.of(1990, 2).lengthOfMonth();
```

## 3。与遗留代码交互

在许多情况下，用户可能需要确保与依赖旧数据库的第三方库的互操作性。

在 Java 8 中，旧的日期库类被扩展了一些方法，这些方法把它们转换成新的日期 API 中相应的对象。新的类提供了相似的功能。

```
Instant instantFromCalendar = GregorianCalendar.getInstance().toInstant();
ZonedDateTime zonedDateTimeFromCalendar = new GregorianCalendar().toZonedDateTime();
Date dateFromInstant = Date.from(Instant.now());
GregorianCalendar calendarFromZonedDateTime = GregorianCalendar.from(ZonedDateTime.now());
Instant instantFromDate = new Date().toInstant();
ZoneId zoneIdFromTimeZone = TimeZone.getTimeZone("PST").toZoneId(); 
```

## 4。结论

在本文中，我们探索了 Java 8 中新的日期时间 API。我们看了一下它与不推荐的 API 相比的优势，并用多个例子指出了不同之处。

请注意，我们仅仅触及了新的日期时间 API 功能的皮毛。请务必通读官方文档，了解新 API 提供的所有工具。

代码示例可以在 [GitHub 项目](https://web.archive.org/web/20221116062642/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-8-datetime)中找到。