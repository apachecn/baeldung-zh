# 在 Java 中将即时格式转换为字符串

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-instant-to-string>

## 1.概观

在本教程中，我们将阐明如何在 Java 中将一个瞬间格式化为一个`String`。

首先，我们将从 Java 中 instant 的一些背景知识开始。然后，我们将展示如何使用核心 Java 和第三方库(如 Joda-Time)来回答我们的核心问题。

## 2.使用核心 Java 即时格式化

根据 [Java 文档](https://web.archive.org/web/20220613105332/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/Instant.html)，instant 是从 Java 纪元 1970-01-01T00:00:00Z 测量的时间戳。

Java 8 附带了一个名为`Instant`的方便的类来表示时间轴上的一个特定的瞬时点。通常，我们可以使用这个类来记录应用程序中的事件时间戳。

现在我们知道了 Java 中的 instant 是什么，让我们看看如何将它转换成一个`String`对象。

### 2.1.使用`DateTimeFormatter`类

一般来说，我们需要一个格式化程序来格式化一个`Instant`对象。幸运的是，Java 8 引入了 [`DateTimeFormatter`](/web/20220613105332/https://www.baeldung.com/java-datetimeformatter) 类来统一格式化日期和时间。

基本上，`DateTimeFormatter`提供了完成工作的`format()`方法。

简单来说， **`DateTimeFormatter`需要一个时区来格式化一个瞬间**。如果没有它，它将无法将 instant 转换为人类可读的日期/时间字段。

例如，假设我们想要使用`dd.MM.yyyy`格式显示我们的`Instant`实例:

```
public class FormatInstantUnitTest {

    private static final String PATTERN_FORMAT = "dd.MM.yyyy";

    @Test
    public void givenInstant_whenUsingDateTimeFormatter_thenFormat() {
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern(PATTERN_FORMAT)
            .withZone(ZoneId.systemDefault());

        Instant instant = Instant.parse("2022-02-15T18:35:24.00Z");
        String formattedInstant = formatter.format(instant);

        assertThat(formattedInstant).isEqualTo("15.02.2022");
    }
    ...
}
```

如上图，我们可以使用`withZone()`方法来指定时区。

请记住，**未指定时区将导致`[UnsupportedTemporalTypeException](https://web.archive.org/web/20220613105332/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/temporal/UnsupportedTemporalTypeException.html)`** :

```
@Test(expected = UnsupportedTemporalTypeException.class)
public void givenInstant_whenNotSpecifyingTimeZone_thenThrowException() {
    DateTimeFormatter formatter = DateTimeFormatter.ofPattern(PATTERN_FORMAT);

    Instant instant = Instant.now();
    formatter.format(instant);
}
```

### 2.2.使用`toString()`方法

另一个解决方案是使用`toString()`方法来获取`Instant`对象的字符串表示。

让我们用一个测试用例来举例说明`toString()`方法的使用:

```
@Test
public void givenInstant_whenUsingToString_thenFormat() {
    Instant instant = Instant.ofEpochMilli(1641828224000L);
    String formattedInstant = instant.toString();

    assertThat(formattedInstant).isEqualTo("2022-01-10T15:23:44Z");
}
```

这种方法的局限性在于**我们不能使用自定义的、人性化的格式来显示瞬间**。

## 3.joda-时间图书馆

或者，我们可以使用 [Joda-Time API](/web/20220613105332/https://www.baeldung.com/joda-time) 来实现相同的目标。这个库提供了一组现成的类和接口，用于在 Java 中操作日期和时间。

在这些类中，我们找到了 [`DateTimeFormat`](https://web.archive.org/web/20220613105332/https://www.joda.org/joda-time/apidocs/org/joda/time/format/DateTimeFormatter.html) 类。顾名思义，**这个类可以用来格式化或解析日期/时间数据到一个字符串**。

那么，我们来举例说明如何使用`DateTimeFormatter` 将一个瞬间转换成一个字符串:

```
@Test
public void givenInstant_whenUsingJodaTime_thenFormat() {
    org.joda.time.Instant instant = new org.joda.time.Instant("2022-03-20T10:11:12");

    String formattedInstant = DateTimeFormat.forPattern(PATTERN_FORMAT)
        .print(instant);

    assertThat(formattedInstant).isEqualTo("20.03.2022");
}
```

正如我们所见，`DateTimeFormatter`提供了`forPattern()`来指定格式化模式，并提供了`print()`来格式化`Instant`对象。

## 4.结论

总而言之，在本文中，我们深入讨论了如何在 Java 中将 instant 格式化为字符串。

在这个过程中，我们探索了几种使用核心 Java 方法实现这一点的方法。然后，我们解释了如何使用 Joda-Time 库完成同样的事情。

和往常一样，本文中使用的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220613105332/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-datetime-string)