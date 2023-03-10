# 休眠–映射日期和时间

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-date-time>

## 1。简介

在本文中，我们将展示如何在 Hibernate 中映射时态列值，包括来自`java.sql`、 `java.util`和`java.time`包的类。

## 2。项目设置

为了演示时态类型的映射，我们需要 H2 数据库和最新版本的`hibernate-core`库:

```java
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>5.4.12.Final</version>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>1.4.194</version>
</dependency>
```

对于当前版本的`hibernate-core`库，请访问 [Maven Central](https://web.archive.org/web/20220524025527/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.hibernate%22%20AND%20a%3A%22hibernate-core%22) 库。

## 3。时区设置

在处理日期时，为 JDBC 司机设定一个特定的时区是个好主意。这样，我们的应用程序将独立于系统的当前时区。

对于我们的示例，我们将在每个会话的基础上设置它:

```java
session = HibernateUtil.getSessionFactory().withOptions()
  .jdbcTimeZone(TimeZone.getTimeZone("UTC"))
  .openSession();
```

另一种方法是在 Hibernate 属性文件中设置用于构建会话工厂的`hibernate.jdbc.time_zone`属性。这样，我们可以为整个应用程序指定一次时区。

## 4。映射`java.sql`类型

`java.sql`包包含与 SQL 标准定义的类型一致的 JDBC 类型:

*   `Date`对应的是`DATE` SQL 类型，只有日期没有时间
*   `Time`对应于`TIME` SQL 类型，它是以小时、分钟和秒指定的一天中的时间
*   `Timestamp`包含精度高达纳秒的日期和时间信息，对应于`TIMESTAMP` SQL 类型

因为这些类型与 SQL 一致，所以它们的映射相对简单。我们可以使用`@Basic`或`@Column`注释:

```java
@Entity
public class TemporalValues {

    @Basic
    private java.sql.Date sqlDate;

    @Basic
    private java.sql.Time sqlTime;

    @Basic
    private java.sql.Timestamp sqlTimestamp;

}
```

我们可以这样设置相应的值:

```java
temporalValues.setSqlDate(java.sql.Date.valueOf("2017-11-15"));
temporalValues.setSqlTime(java.sql.Time.valueOf("15:30:14"));
temporalValues.setSqlTimestamp(
  java.sql.Timestamp.valueOf("2017-11-15 15:30:14.332"));
```

注意，为实体字段选择`java.sql`类型并不总是一个好的选择。这些类是 JDBC 特有的，包含许多不推荐使用的功能。

## 5。映射`java.util.Date`类型

**类型`java.util.Date`包含日期和时间信息，精确到毫秒。**但是它不直接与任何 SQL 类型相关。

这就是为什么我们需要另一个注释来指定所需的 SQL 类型:

```java
@Basic
@Temporal(TemporalType.DATE)
private java.util.Date utilDate;

@Basic
@Temporal(TemporalType.TIME)
private java.util.Date utilTime;

@Basic
@Temporal(TemporalType.TIMESTAMP)
private java.util.Date utilTimestamp;
```

`@Temporal`注释有一个类型为`TemporalType.` 的参数值，它可以是`DATE`、`TIME`或`TIMESTAMP`，这取决于我们想要用于映射的底层 SQL 类型。

我们可以这样设置相应的字段:

```java
temporalValues.setUtilDate(
  new SimpleDateFormat("yyyy-MM-dd").parse("2017-11-15"));
temporalValues.setUtilTime(
  new SimpleDateFormat("HH:mm:ss").parse("15:30:14"));
temporalValues.setUtilTimestamp(
  new SimpleDateFormat("yyyy-MM-dd HH:mm:ss.SSS")
    .parse("2017-11-15 15:30:14.332"));
```

正如我们所见，**`java.util.Date`类型(毫秒精度)不够精确，无法处理时间戳值(纳秒精度)。**

因此，当我们从数据库中检索实体时，我们会毫不意外地在该字段中找到一个`java.sql.Timestamp`实例，即使我们最初持久化了一个`java.util.Date`:

```java
temporalValues = session.get(TemporalValues.class, 
  temporalValues.getId());
assertThat(temporalValues.getUtilTimestamp())
  .isEqualTo(java.sql.Timestamp.valueOf("2017-11-15 15:30:14.332"));
```

这对我们的代码来说应该没问题，因为`Timestamp`扩展了`Date`。

## 6。映射`java.util.Calendar`类型

与`java.util.Date`一样，`java.util.Calendar`类型可能被映射到不同的 SQL 类型，所以我们必须用`@Temporal`来指定它们。

唯一的区别是 Hibernate 不支持从`Calendar`到 `TIME`的映射:

```java
@Basic
@Temporal(TemporalType.DATE)
private java.util.Calendar calendarDate;

@Basic
@Temporal(TemporalType.TIMESTAMP)
private java.util.Calendar calendarTimestamp;
```

下面是我们如何设置该字段的值:

```java
Calendar calendarDate = Calendar.getInstance(
  TimeZone.getTimeZone("UTC"));
calendarDate.set(Calendar.YEAR, 2017);
calendarDate.set(Calendar.MONTH, 10);
calendarDate.set(Calendar.DAY_OF_MONTH, 15);
temporalValues.setCalendarDate(calendarDate);
```

## 7。映射`java.time`类型

**从 Java 8 开始，新的 Java 日期和时间 API 可用于处理时态值**。这个 API 修复了`java.util.Date`和`java.util.Calendar`类的许多问题。

来自 `java.time`包的类型被直接映射到相应的 SQL 类型。所以不需要显式指定`@Temporal`注释:

*   `LocalDate`映射到`DATE`
*   `LocalTime`和`OffsetTime`被映射到`TIME`
*   `Instant`、`LocalDateTime`、`OffsetDateTime`、`ZonedDateTime`映射到`TIMESTAMP`

这意味着我们只能用`@Basic`(或`@Column`)注释来标记这些字段，就像这样:

```java
@Basic
private java.time.LocalDate localDate;

@Basic
private java.time.LocalTime localTime;

@Basic
private java.time.OffsetTime offsetTime;

@Basic
private java.time.Instant instant;

@Basic
private java.time.LocalDateTime localDateTime;

@Basic
private java.time.OffsetDateTime offsetDateTime;

@Basic
private java.time.ZonedDateTime zonedDateTime;
```

`java.time`包中的每个时态类都有一个静态的`parse()`方法来使用适当的格式解析提供的`String`值。下面是我们如何设置实体字段的值:

```java
temporalValues.setLocalDate(LocalDate.parse("2017-11-15"));

temporalValues.setLocalTime(LocalTime.parse("15:30:18"));
temporalValues.setOffsetTime(OffsetTime.parse("08:22:12+01:00"));

temporalValues.setInstant(Instant.parse("2017-11-15T08:22:12Z"));
temporalValues.setLocalDateTime(
  LocalDateTime.parse("2017-11-15T08:22:12"));
temporalValues.setOffsetDateTime(
  OffsetDateTime.parse("2017-11-15T08:22:12+01:00"));
temporalValues.setZonedDateTime(
  ZonedDateTime.parse("2017-11-15T08:22:12+01:00[Europe/Paris]"));
```

## 8。结论

在本文中，我们展示了如何在 Hibernate 中映射不同类型的时态值。

这篇文章的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220524025527/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate-mapping)