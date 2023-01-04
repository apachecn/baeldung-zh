# java.util.Date vs java.sql.Date

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-util-date-sql-date>

## 1。概述

在本教程中，我们将比较两个日期类:`java.util.Date`和`java.sql.Date`。

一旦我们完成了比较，就应该清楚使用哪一个以及为什么。

## 2。`java.util.Date`

**[`java.util.Date`](https://web.archive.org/web/20221205132232/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Date.html)类表示特定的时刻，从 1970 年 1 月 1 日 00:00:00 GMT(大纪元时间)**开始，精度为毫秒。该类用于保持协调世界时(UTC)。

我们可以用两种方法初始化它。

通过调用构造函数:

```
Date date = new Date();
```

这将创建一个新的`date`对象，时间设置为当前时间，精确到毫秒。

或者通过从该时期开始经过若干毫秒:

```
long timestamp = 1532516399000; // 25 July 2018 10:59:59 UTC
Date date = new Date(timestamp);
```

让我们注意一下，Java 8 之前出现的其他构造函数现在已经被弃用了。

然而， **`Date`有很多问题，总体来说不再推荐使用它**。

它是可变的。一旦我们初始化了它，我们就可以改变它的内部值。例如，我们可以调用`setTime`方法:

```
date.setTime(0); // 01 January 1970 00:00:00
```

要了解不可变对象的更多优点，请查看本文:[Java 中的不可变对象](/web/20221205132232/https://www.baeldung.com/java-immutable-object)。

它也不能很好地处理所有的日期。从技术上讲，它应该反映协调世界时(UTC)。但是，这取决于主机环境的操作系统。

大多数现代操作系统使用 1 天= 24 小时 x 60 米 x 60 秒= 86400 秒，我们可以看到，这并没有将“闰秒”考虑在内。

**随着 Java 8 的推出， [`java.time`](https://web.archive.org/web/20221205132232/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/package-summary.html) 应该会用到**包。在 Java 8 之前，有一个替代方案可用—[`Joda Time`](https://web.archive.org/web/20221205132232/http://www.joda.org/joda-time/)。

## `**3\. java.sql.Date**`

**的 [`java.sql.Date`](https://web.archive.org/web/20221205132232/https://docs.oracle.com/en/java/javase/11/docs/api/java.sql/java/sql/Date.html) 扩展了`java.util.Date`类。**

它的主要用途是表示 SQL DATE，它保存年、月和日。不保留时间数据。

事实上，日期存储为自 1970 年 1 月 1 日 00:00:00 GMT 以来的毫秒数，时间部分被标准化，即设置为零。

基本上，它是一个处理 SQL 特定需求的`java.util.Date`包装器。`java.sql.Date`应该只在处理数据库时使用。

然而，由于`java.sql.Date`不保存时区信息，我们的本地环境和数据库服务器之间的时区转换依赖于 JDBC 驱动程序的实现。这又增加了一层复杂性。

最后让我们注意一下，为了支持其他 SQL 数据类型:SQL TIME 和 SQL TIMESTAMP，还有另外两个`java.sql`类可用: [`Time`](https://web.archive.org/web/20221205132232/https://docs.oracle.com/en/java/javase/11/docs/api/java.sql/java/sql/Time.html) 和 [`Timestamp`](https://web.archive.org/web/20221205132232/https://docs.oracle.com/en/java/javase/11/docs/api/java.sql/java/sql/Timestamp.html) 。

后者，即使从`java.util.Date`扩展，也支持纳秒。

## 4。结论

类`java.util.Date`存储自纪元以来的日期时间值，单位为毫秒。`java.sql.Date`只存储日期值，通常在 JDBC 使用。

处理日期是很棘手的。我们需要记住一些特殊情况:闰秒，不同的时区等等。与 JDBC 打交道时，我们可以谨慎使用`java.sql.Date`。

如果我们要使用`java.util.Date,`，我们需要记住它的缺点。如果使用 Java 8，那么最好不要使用`java.util.Date`。