# 用 Java 计算年龄

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-get-age>

## 1。概述

在这个快速教程中，我们将看到如何使用 Java 8、Java 7 和 Joda-Time 库计算年龄。

在所有情况下，我们将把出生日期和当前日期作为输入，并返回以年为单位计算的年龄。

## 2。使用 Java 8

Java 8 引入了新的日期时间 API ,用于处理日期和时间，主要基于 Joda-Time 库。

在 Java 8 中，我们可以使用`java.time.LocalDate`来表示我们的出生日期和当前日期，然后使用`Period`来计算它们在年份上的差异:

```
public int calculateAge(
  LocalDate birthDate,
  LocalDate currentDate) {
    // validate inputs ...
    return Period.between(birthDate, currentDate).getYears();
}
```

**[`LocalDate`](https://web.archive.org/web/20220813055522/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/LocalDate.html) 在这里很有用，因为**只表示日期，而 Java 的`Date`类则表示日期和时间。`LocalDate.now()`可以给我们当前的日期。

而 [`Period`](https://web.archive.org/web/20220813055522/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/Period.html) 在我们需要考虑年、月、日的时间段时很有帮助。

**如果我们想得到一个更精确的年龄，比如说以秒为单位，那么我们会想分别查看一下** **`LocalDateTime`和`Duration`(也许会返回一个`long `)。**

## 3。使用 Joda-Time

如果 Java 8 不是一个选项，**我们仍然可以从 [Joda-Time](https://web.archive.org/web/20220813055522/http://www.joda.org/joda-time/)** 得到同样的结果，这是 Java 8 出现之前的日期-时间操作的事实上的标准。

我们需要将 [Joda 时间依赖性](https://web.archive.org/web/20220813055522/https://search.maven.org/classic/#artifactdetails%7Cjoda-time%7Cjoda-time%7C2.10%7Cjar)添加到 pom 中:

```
<dependency>
  <groupId>joda-time</groupId>
  <artifactId>joda-time</artifactId>
  <version>2.10</version>
</dependency>
```

然后我们可以编写一个类似的方法来计算年龄，这次使用 [`LocalDate`](https://web.archive.org/web/20220813055522/http://www.joda.org/joda-time/apidocs/index.html) 和[`Years`](https://web.archive.org/web/20220813055522/http://joda-time.sourceforge.net/apidocs/org/joda/time/Years.html)from[Joda-Time](/web/20220813055522/https://www.baeldung.com/joda-time):

```
public int calculateAgeWithJodaTime(
  org.joda.time.LocalDate birthDate,
  org.joda.time.LocalDate currentDate) {
    // validate inputs ...
    Years age = Years.yearsBetween(birthDate, currentDate);
    return age.getYears();   
}
```

## 4。使用 Java 7

在 Java 7 中没有专用的 API，我们只能自己开发，所以有很多方法。

举个例子，我们可以使用`[java.util.Date](https://web.archive.org/web/20220813055522/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Date.html)`:

```
public int calculateAgeWithJava7(
  Date birthDate, 
  Date currentDate) {            
    // validate inputs ...                                                                               
    DateFormat formatter = new SimpleDateFormat("yyyyMMdd");                           
    int d1 = Integer.parseInt(formatter.format(birthDate));                            
    int d2 = Integer.parseInt(formatter.format(currentDate));                          
    int age = (d2 - d1) / 10000;                                                       
    return age;                                                                        
}
```

在这里，我们将给定的`birthDate`和`currentDate`对象转换成整数，并找出它们之间的差异，并且**只要我们在 8000 年后**还没有使用 Java 7，这种方法应该可以工作到那时。

## 5。结论

在本文中，我们展示了如何使用 Java 8、Java 7 和 Joda-Time 库轻松计算年龄。

要了解更多关于 Java 8 的数据时间支持，请查看我们的 Java 8 日期时间介绍。

和往常一样，这些片段的完整代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220813055522/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-date-operations-1)