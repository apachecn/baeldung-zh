# Java 中的增量日期

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-increment-date>

## 1。概述

在本教程中，我们将研究使用 Java 将日期增加一天的方法。在 Java 8 之前，标准的 Java 日期和时间库对用户不太友好。因此，Joda-Time 成为 Java 8 之前 Java 事实上的标准日期和时间库。

也有其他的类和库可以用来完成这个任务，比如`java.util.Calendar` 和 Apache Commons。

Java 8 包含了一个更好的日期和时间 API 来解决旧库的缺点。

因此，我们正在研究**如何使用 Java 8、Joda-Time API、Java 的 `Calendar` 类和 Apache Commons 库**将日期递增一天。

## 2。Maven 依赖关系

以下依赖关系应包含在`pom.xml`文件中:

```java
<dependencies>
    <dependency>
        <groupId>joda-time</groupId>
        <artifactId>joda-time</artifactId>
        <version>2.10</version>
    </dependency>
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-lang3</artifactId>
        <version>3.12.0</version>
    </dependency>
</dependencies>
```

你可以在 [Maven Central](https://web.archive.org/web/20220617075809/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22joda-time%22%20AND%20a%3A%22joda-time%22) 上找到最新版本的 Joda-Time，以及最新版本的 [Apache Commons Lang](https://web.archive.org/web/20220617075809/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.commons%22%20AND%20a%3A%22commons-lang3%22) 。

## 3。使用`java.time`

`java.time.LocalDate `类是不可变的日期时间表示，通常被视为年-月-日。

`LocalDate `有许多处理日期的方法，让我们看看如何用它来完成同样的任务:

```java
public static String addOneDay(String date) {
    return LocalDate
      .parse(date)
      .plusDays(1)
      .toString();
}
```

在这个例子中，我们使用`java.time.LocalDate `类及其`plusDays() `方法将日期增加一天。

现在，让我们验证这个方法是否如预期的那样工作:

```java
@Test
public void givenDate_whenUsingJava8_thenAddOneDay() 
  throws Exception {

    String incrementedDate = addOneDay("2018-07-03");
    assertEquals("2018-07-04", incrementedDate);
}
```

## 4。使用`java.util.Calendar`

另一种方法是使用`java.util.Calendar `和它的`add()`方法来增加日期。

我们将它与`java.text.SimpleDateFormat `一起用于日期格式化:

```java
public static String addOneDayCalendar(String date) 
  throws ParseException {

    SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
    Calendar c = Calendar.getInstance();
    c.setTime(sdf.parse(date));
    c.add(Calendar.DATE, 1);
    return sdf.format(c.getTime());
}
```

是否确保使用预期的日期格式。通过`add() `方法增加日期。

同样，让我们确保这种方法按预期工作:

```java
@Test
public void givenDate_whenUsingCalendar_thenAddOneDay() 
  throws Exception {

    String incrementedDate = addOneDayCalendar("2018-07-03");
    assertEquals("2018-07-04", incrementedDate);
}
```

## 5。使用 Joda-Time

`org.joda.time.DateTime `类有许多帮助正确处理日期和时间的方法。

让我们看看如何使用它将日期增加一天:

```java
public static String addOneDayJodaTime(String date) {
    DateTime dateTime = new DateTime(date);
    return dateTime
      .plusDays(1)
      .toString("yyyy-MM-dd");
}
```

这里，我们使用`org.joda.time.DateTime `类及其`plusDays() `方法将日期增加一天。

我们可以验证上面的代码与下面的单元测试一起工作:

```java
@Test
public void givenDate_whenUsingJodaTime_thenAddOneDay() throws Exception {
    String incrementedDate = addOneDayJodaTime("2018-07-03");
    assertEquals("2018-07-04", incrementedDate);
}
```

## 6。使用 Apache Commons

另一个常用于日期操作的库是 Apache Commons。这是一套围绕着使用`java.util.Calendar`和`java.util.Date`对象的实用程序。

对于我们的任务，我们可以使用`org.apache.commons.lang3.time.DateUtils `类及其`addDays() `方法(注意`SimpleDateFormat`再次用于日期格式化):

```java
public static String addOneDayApacheCommons(String date) 
  throws ParseException {

    SimpleDateFormat sdf
      = new SimpleDateFormat("yyyy-MM-dd");
    Date incrementedDate = DateUtils
      .addDays(sdf.parse(date), 1);
    return sdf.format(incrementedDate);
}
```

像往常一样，我们将通过单元测试来验证结果:

```java
@Test
public void givenDate_whenUsingApacheCommons_thenAddOneDay()
  throws Exception {

    String incrementedDate = addOneDayApacheCommons(
      "2018-07-03");
    assertEquals("2018-07-04", incrementedDate);
}
```

## 7。结论

在这篇简短的文章中，我们研究了处理将日期递增一天的简单任务的各种方法。我们已经展示了如何使用 Java 的核心 API 以及一些流行的第三方库来实现这一点。

本文中使用的代码示例可以在 GitHub 上找到[。](https://web.archive.org/web/20220617075809/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-date-operations-1)