# 将 java.util.Date 转换为 java.sql.Date

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-convert-util-date-to-sql>

## 1.介绍

在这个简短的教程中，我们将探索几种将`java.util.Date`转换为`java.sql.Date`的策略。

首先，我们将看看标准转换，然后，我们将检查一些被认为是最佳实践的备选方案。

## 2.`java.util.Date`对`java.sql.Date`

这两个日期类都用于特定的场景，是不同 Java 标准包的一部分:

*   `[java.util](https://web.archive.org/web/20221208143830/https://docs.oracle.com/javase/8/docs/api/java/util/package-summary.html) `包是 JDK 的一部分，包含各种实用程序类以及日期和时间工具。
*   [`java.sql`](https://web.archive.org/web/20221208143830/https://docs.oracle.com/en/java/javase/12/docs/api/java.sql/java/sql/package-summary.html) 包是 JDBC API 的一部分，从 Java 7 开始，默认包含在 JDK 中。

`[java.util.Date](https://web.archive.org/web/20221208143830/https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/Date.html)` 代表特定的时间瞬间，精度为毫秒:

```java
java.util.Date date = new java.util.Date(); 
System.out.println(date);
// Wed Mar 27 08:22:02 IST 2015
```

`[java.sql.Date](https://web.archive.org/web/20221208143830/https://docs.oracle.com/javase/7/docs/api/java/sql/Date.html)` 是一个以毫秒为单位的包装器，允许 JDBC 驱动程序将其识别为 SQL 日期值。这个类的值只不过是从 [Unix 纪元](/web/20221208143830/https://www.baeldung.com/java-date-unix-timestamp)开始以毫秒计算的特定日期的年、月和日。任何比一天更细的时间信息都将被截断:

```java
long millis=System.currentTimeMillis(); 
java.sql.Date date = new java.sql.Date(millis); 
System.out.println(date);
// 2015-03-30
```

## 3.为什么需要转换

虽然`java.util.Date`的用法更普遍，但是`java.sql.Date`用于支持 Java 应用程序与数据库的通信。因此，在这些情况下需要转换为`java.sql.Date`。

显式的[引用转换](/web/20221208143830/https://www.baeldung.com/java-type-casting)也不会工作，因为我们正在处理一个完全不同的类层次:**没有向下转换或向上转换可用。**如果我们试图将其中一个日期转换为另一个日期，我们将收到一个`[ClassCastException](/web/20221208143830/https://www.baeldung.com/java-classcastexception#:~:text=ClassCastException%20is%20an%20unchecked%20exception,how%20we%20can%20avoid%20them.)`:

```java
java.sql.Date date = (java.sql.Date) new java.util.Date() // not allowed
```

## 4.如何转换为`java.sql.Date`

从`java.util.Date`到`java.sql.Date`有几种转换策略，我们将在下面探讨。

### 4.1.标准转换

正如我们在上面看到的，`java.util.Date`包含时间信息，而`java.sql.Date`不包含。因此，我们**通过使用` java.sql.Date`** 的构造器方法实现了有损转换，该方法从 Unix 纪元开始接受以毫秒表示的输入时间:

```java
java.util.Date utilDate = new java.util.Date();
java.sql.Date sqlDate = new java.sql.Date(utilDate.getTime());
```

事实上，由于时区不同，丢失表示值**的时间部分可能会导致报告不同的日期**:

```java
SimpleDateFormat isoFormat = new SimpleDateFormat("yyyy-MM-dd'T'HH:mm:ss");
isoFormat.setTimeZone(TimeZone.getTimeZone("America/Los_Angeles"));

java.util.Date date = isoFormat.parse("2010-05-23T22:01:02");
TimeZone.setDefault(TimeZone.getTimeZone("America/Los_Angeles"));
java.sql.Date sqlDate = new java.sql.Date(date.getTime());
System.out.println(sqlDate);
// This will print 2010-05-23

TimeZone.setDefault(TimeZone.getTimeZone("Rome"));
sqlDate = new java.sql.Date(date.getTime());
System.out.println(sqlDate);
// This will print 2010-05-24
```

出于这个原因，我们可能需要考虑一种转换方法，我们将在接下来的小节中研究这种方法。

### 4.2.使用 `java.sql.Timestamp`代替`java.sql.Date`

要考虑的第一个选择是使用`java.sql.Timestamp`类而不是`java.sql.Date`。这个类还包含关于时间的信息:

```java
java.sql.Timestamp timestamp = new java.sql.Timestamp(date.getTime());
System.out.println(date); //Mon May 24 07:01:02 CEST 2010
System.out.println(timestamp); //2010-05-24 07:01:02.0
```

当然，如果我们查询一个具有`DATE`类型的数据库列，这个解决方案可能不是正确的。

### 4.3.使用来自`java.time`包的类

第二个也是最好的选择是将两个类都转换成`[java.time](https://web.archive.org/web/20221208143830/https://docs.oracle.com/javase/8/docs/api/java/time/package-summary.html)`包中提供的新类。**此转换的唯一先决条件是使用 JDBC 4.2(或更高版本)**。JDBC 4.2 于 2014 年 3 月与 [Java SE 8](/web/20221208143830/https://www.baeldung.com/java-8-new-features) 一同发布。

从 Java 8 开始，Java 早期版本中提供的日期-时间类的使用已经被弃用，取而代之的是新的`java.time`包中提供的。这些增强类可以更好地满足所有日期/时间需求，包括通过 [JDBC 驱动](/web/20221208143830/https://www.baeldung.com/java-jdbc)与数据库通信。

如果我们采用这种策略， **`java.util.Date`应该转换成`java.time.Instant`** :

```java
Date date = new java.util.Date();
Instant instant = date.toInstant().atZone(ZoneId.of("Rome");
```

而 **`java.sql.Date`应该转换成`java.time.LocalDate`** :

```java
java.sql.Date sqlDate = new java.sql.Date(timeInMillis);
java.time.LocalDate localDate = sqlDate.toLocalDate();
```

`java.time.Instant`类可用于映射 SQL 日期时间列，而`java.time.LocalDate`可用于映射 SQL 日期列。

例如，现在让我们用时区信息生成一个`java.util.Date`:

```java
SimpleDateFormat isoFormat = new SimpleDateFormat("yyyy-MM-dd'T'HH:mm:ss");
isoFormat.setTimeZone(TimeZone.getTimeZone("America/Los_Angeles"));
Date date = isoFormat.parse("2010-05-23T22:01:02"); 
```

接下来，让我们从`java.util.Date`生成一个`LocalDate`:

```java
TimeZone.setDefault(TimeZone.getTimeZone("America/Los_Angeles"));
java.time.LocalDate localDate = date.toInstant().atZone(ZoneId.of("America/Los_Angeles")).toLocalDate();
Asserts.assertEqual("2010-05-23", localDate.toString());
```

如果我们随后尝试切换默认时区，`LocalDate`将保持相同的值:

```java
TimeZone.setDefault(TimeZone.getTimeZone("Rome"));
localDate = date.toInstant().atZone(ZoneId.of("America/Los_Angeles")).toLocalDate();
Asserts.assertEqual("2010-05-23", localDate.toString()) 
```

这正如预期的那样工作，这要感谢我们在转换期间指定时区的显式引用。

## 5.结论

在本教程中。我们已经看到了如何从标准的`java.util Date`转换到`java.sql`包中提供的。除了标准转换，我们还研究了两种替代方案。第一个使用`Timestamp`，第二个依赖于更新的`java.time`类。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20221208143830/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-date-operations-3)