# Java 字符串转换

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-string-conversions>

## 1。概述

在这篇简短的文章中，我们将探索一些简单的将`String`对象转换成 Java 支持的不同数据类型的方法。

## 2。将`String`转换为`int`或`Integer`或

如果我们需要将一个`String`转换成原语`int`或`Integer`包装类型，我们可以使用`parseInt()`或`valueOf()`API 来获得相应的`int`或`Integer`返回值:

```
@Test
public void whenConvertedToInt_thenCorrect() {
    String beforeConvStr = "1";
    int afterConvInt = 1;

    assertEquals(Integer.parseInt(beforeConvStr), afterConvInt);
}

@Test
public void whenConvertedToInteger_thenCorrect() {
    String beforeConvStr = "12";
    Integer afterConvInteger = 12;

    assertEquals(Integer.valueOf(beforeConvStr).equals(afterConvInteger), true);
}
```

## 3。将`String`转换为`long`或`Long`或

如果我们需要将一个`String`转换成原语`long`或`Long`包装类型，我们可以分别使用`parseLong()`或`valueOf()`:

```
@Test
public void whenConvertedTolong_thenCorrect() {
    String beforeConvStr = "12345";
    long afterConvLongPrimitive = 12345;

    assertEquals(Long.parseLong(beforeConvStr), afterConvLongPrimitive);
}

@Test
public void whenConvertedToLong_thenCorrect() {
    String beforeConvStr = "14567";
    Long afterConvLong = 14567l;

    assertEquals(Long.valueOf(beforeConvStr).equals(afterConvLong), true);
}
```

## 4。将`String`转换为`double`或`Double`或

如果我们需要将一个`String`转换成原语`double`或`Double`包装类型，我们可以分别使用`parseDouble()`或`valueOf()`:

```
@Test
public void whenConvertedTodouble_thenCorrect() {
    String beforeConvStr = "1.4";
    double afterConvDoublePrimitive = 1.4;

    assertEquals(Double.parseDouble(beforeConvStr), afterConvDoublePrimitive, 0.0);
}

@Test
public void whenConvertedToDouble_thenCorrect() {
    String beforeConvStr = "145.67";
    double afterConvDouble = 145.67d;

    assertEquals(Double.valueOf(beforeConvStr).equals(afterConvDouble), true);
}
```

## 5。将`String`转换为`ByteArray`

为了将`String`转换成一个字节数组，`getBytes()`使用平台的默认字符集将`String`编码成一个字节序列，并将结果存储到一个新的字节数组中。

当传递的`String`不能使用默认字符集编码时，`getBytes()`的行为是未指定的。根据 java [文档](https://web.archive.org/web/20220625235933/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html)，当需要对编码过程进行更多控制时，应该使用[Java . nio . charset . charset encoder](https://web.archive.org/web/20220625235933/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/charset/CharsetEncoder.html)类:

```
@Test
public void whenConvertedToByteArr_thenCorrect() {
    String beforeConvStr = "abc";
    byte[] afterConvByteArr = new byte[] { 'a', 'b', 'c' };

    assertEquals(Arrays.equals(beforeConvStr.getBytes(), afterConvByteArr), true);
}
```

## 6。将`String`转换为`CharArray`

为了将一个`String`转换成一个`CharArray`实例，我们可以简单地使用`toCharArray()`:

```
@Test
public void whenConvertedToCharArr_thenCorrect() {
    String beforeConvStr = "hello";
    char[] afterConvCharArr = { 'h', 'e', 'l', 'l', 'o' };

    assertEquals(Arrays.equals(beforeConvStr.toCharArray(), afterConvCharArr), true);
}
```

## 7。将`String`转换为`boolean`或`Boolean`或

要将一个`String`实例转换成原始的`boolean`或`Boolean`包装类型，我们可以分别使用`parseBoolean()`或`valueOf()`API:

```
@Test
public void whenConvertedToboolean_thenCorrect() {
    String beforeConvStr = "true";
    boolean afterConvBooleanPrimitive = true;

    assertEquals(Boolean.parseBoolean(beforeConvStr), afterConvBooleanPrimitive);
}

@Test
public void whenConvertedToBoolean_thenCorrect() {
    String beforeConvStr = "true";
    Boolean afterConvBoolean = true;

    assertEquals(Boolean.valueOf(beforeConvStr), afterConvBoolean);
}
```

## 8。将`String`转换为`Date`或`LocalDateTime`或

Java 6 提供了用于表示日期的`java.util.Date`数据类型。Java 8 为`Date`和`Time`引入了新的 API，以解决旧的`java.util.Date`和`java.util.Calendar`的缺点。

你可以阅读[这篇](/web/20220625235933/https://www.baeldung.com/java-8-date-time-intro)文章了解更多详情。

### 8.1。将`String`转换为`java.util.Date`

为了将`String`对象转换成`Date`对象，我们需要首先通过传递描述日期和时间格式的模式来构造一个`SimpleDateFormat`对象。

例如，模式的可能值可以是“MM-dd-yyyy”或“yyyy-MM-dd”。接下来，我们需要调用传递了`String`的`parse`方法。

作为参数传递的`String`应该与模式的格式相同。否则，运行时将抛出一个`ParseException`:

```
@Test
public void whenConvertedToDate_thenCorrect() throws ParseException {
    String beforeConvStr = "15/10/2013";
    int afterConvCalendarDay = 15;
    int afterConvCalendarMonth = 9;
    int afterConvCalendarYear = 2013;
    SimpleDateFormat formatter = new SimpleDateFormat("dd/M/yyyy");
    Date afterConvDate = formatter.parse(beforeConvStr);
    Calendar calendar = new GregorianCalendar();
    calendar.setTime(afterConvDate);

    assertEquals(calendar.get(Calendar.DAY_OF_MONTH), afterConvCalendarDay);
    assertEquals(calendar.get(Calendar.MONTH), afterConvCalendarMonth);
    assertEquals(calendar.get(Calendar.YEAR), afterConvCalendarYear);
}
```

### 8.2。将`String`转换为`java.time.LocalDateTime`

`LocalDateTime`是一个不可变的日期时间对象，表示时间，通常被视为年-月-日-小时-分-秒。

为了将字符串对象转换成`LocalDateTime`对象，我们可以简单地使用`parse` API:

```
@Test
public void whenConvertedToLocalDateTime_thenCorrect() {
    String str = "2007-12-03T10:15:30";
    int afterConvCalendarDay = 03;
    Month afterConvCalendarMonth = Month.DECEMBER;
    int afterConvCalendarYear = 2007;
    LocalDateTime afterConvDate 
      = new UseLocalDateTime().getLocalDateTimeUsingParseMethod(str);

    assertEquals(afterConvDate.getDayOfMonth(), afterConvCalendarDay);
    assertEquals(afterConvDate.getMonth(), afterConvCalendarMonth);
    assertEquals(afterConvDate.getYear(), afterConvCalendarYear);
}
```

根据[Java . TIME . format . datetime formatter . iso _ LOCAL _ DATE _ TIME](https://web.archive.org/web/20220625235933/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/format/DateTimeFormatter.html#ISO_LOCAL_DATE_TIME)，`String`必须代表有效时间。否则，运行时会抛出一个`ParseException`。

例如“`2011-12-03`”表示有效的字符串格式，其中 4 位数表示年份，2 位数表示年份的月份，2 位数表示月份的日期 。

## 9。结论

在这篇快速教程中，我们介绍了将 S `tring`对象转换成 java 支持的不同数据类型的不同实用方法。

GitHub 上的[提供了本文的完整源代码和所有代码片段。](https://web.archive.org/web/20220625235933/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-conversions-2)