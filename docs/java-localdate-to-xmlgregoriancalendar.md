# 在 LocalDate 和 XMLGregorianCalendar 之间转换

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-localdate-to-xmlgregoriancalendar>

## 1.概观

在这个快速教程中，我们将讨论`LocalDate`和`XMLGregorianCalendar` ，并提供两种类型之间转换的例子。

## 2.`XMLGregorianCalendar`

XML 模式标准为用 XML 格式指定日期定义了明确的规则。为了使用这种格式，Java 1.5 中引入的 Java 类`[XMLGregorianCalendar](/web/20220523235403/https://www.baeldung.com/java-gregorian-calendar),` 是 W3C XML Schema 1.0 日期/时间数据类型的代表。

## 3.`LocalDate`

一个 [`LocalDate`](/web/20220523235403/https://www.baeldung.com/java-8-date-time-intro) 实例表示 ISO-8601 日历系统中没有时区的日期。因此，`LocalDate`适合存储生日，但不适合存储任何与时间相关的内容。Java 在 1.8 版本中引入了`LocalDate`。

## 4.从`LocalDate`到`XMLGregorianCalendar`

首先，我们来看看如何从`LocalDate`转换到`XMLGregorianCalendar`。为了生成一个新的`XMLGregorianCalendar`实例，我们使用了`javax.xml.datatype`包中的一个`DataTypeFactory`。

因此，让我们创建一个`LocalDate`的实例，并将其转换为`XMLGregorianCalendar`:

```java
LocalDate localDate = LocalDate.of(2019, 4, 25);

XMLGregorianCalendar xmlGregorianCalendar = 
  DatatypeFactory.newInstance().newXMLGregorianCalendar(localDate.toString());

assertThat(xmlGregorianCalendar.getYear()).isEqualTo(localDate.getYear());
assertThat(xmlGregorianCalendar.getMonth()).isEqualTo(localDate.getMonthValue());
assertThat(xmlGregorianCalendar.getDay()).isEqualTo(localDate.getDayOfMonth());
assertThat(xmlGregorianCalendar.getTimezone()).isEqualTo(DatatypeConstants.FIELD_UNDEFINED); 
```

如前所述，`XMLGregorianCalendar`实例有可能包含时区信息。然而，`LocalDate`没有任何关于时间的信息。

因此，当我们执行转换时，**时区值将保持为`FIELD_UNDEFINED`** 。

## 5.从`XMLGregorianCalendar`到`LocalDate`

同样，我们现在将看看如何反过来执行转换。事实证明，从一个`XMLGregorianCalendar`转换到`LocalDate`要容易得多。

同样，由于`LocalDate`没有关于时间的信息，一个`LocalDate`实例只能包含`XMLGregorianCalendar`信息的一个子集。

让我们创建一个`XMLGregorianCalendar`的实例并执行转换:

```java
XMLGregorianCalendar xmlGregorianCalendar = 
  DatatypeFactory.newInstance().newXMLGregorianCalendar("2019-04-25");

LocalDate localDate = LocalDate.of(
  xmlGregorianCalendar.getYear(), 
  xmlGregorianCalendar.getMonth(), 
  xmlGregorianCalendar.getDay());

assertThat(localDate.getYear()).isEqualTo(xmlGregorianCalendar.getYear());
assertThat(localDate.getMonthValue()).isEqualTo(xmlGregorianCalendar.getMonth());
assertThat(localDate.getDayOfMonth()).isEqualTo(xmlGregorianCalendar.getDay()); 
```

## 6.结论

在这个快速教程中，我们已经介绍了`LocalDate`实例和`XMLGregorianCalendar`实例之间的转换，反之亦然。

和往常一样，GitHub 上的示例代码[可用。](https://web.archive.org/web/20220523235403/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-datetime-conversion)