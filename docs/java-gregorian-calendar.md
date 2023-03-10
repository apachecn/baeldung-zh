# java.util.GregorianCalendar 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-gregorian-calendar>

## 1。简介

在本教程中，我们将快速浏览一下`GregorianCalendar`类。

## 2。`GregorianCalendar`

`GregorianCalendar`是抽象类`java.util.Calendar`的具体实现。毫不奇怪，公历是世界上使用最广泛的民用 历法 。

### 2.1。获取实例

有两种方法可以获得`GregorianCalendar:` `Calendar.getInstance()`的实例，并使用其中一个构造函数。

**使用静态工厂方法`Calendar.getInstance() `并不是一种推荐的方法，因为它会返回一个受默认语言环境影响的实例。**

它可能会为泰语返回一个`BuddhistCalendar`,为日本返回一个`JapaneseImperialCalendar`。不知道被返回的实例的类型可能会导致`ClassCastException` `:`

```java
@Test(expected = ClassCastException.class)
public void test_Class_Cast_Exception() {
    TimeZone tz = TimeZone.getTimeZone("GMT+9:00");
    Locale loc = new Locale("ja", "JP", "JP");
    Calendar calendar = Calendar.getInstance(loc);
    GregorianCalendar gc = (GregorianCalendar) calendar;
}
```

**使用七个重载的构造函数之一，我们可以初始化`Calendar`对象，或者使用默认的日期和时间，这取决于我们操作系统的语言环境，或者我们可以指定日期、时间、语言环境和时区的组合。**

让我们来理解不同的构造函数，通过它们可以实例化一个`GregorianCalendar`对象。

默认构造函数将使用操作系统的时区和区域设置中的当前日期和时间来初始化日历:

```java
new GregorianCalendar();
```

我们可以用默认的区域设置指定默认时区的`year, month, dayOfMonth, hourOfDay, minute`和秒:

```java
new GregorianCalendar(2018, 6, 27, 16, 16, 47);
```

注意，我们不必指定`hourOfDay, minute`和`second`，因为还有其他没有这些参数的构造函数。

我们可以将时区作为一个参数来传递，以便在这个时区中创建一个带有默认区域设置的日历:

```java
new GregorianCalendar(TimeZone.getTimeZone("GMT+5:30"));
```

我们可以将地区作为一个参数传递，以在该地区创建一个带有默认时区的日历:

```java
new GregorianCalendar(new Locale("en", "IN"));
```

最后，我们可以将时区和地区作为参数传递:

```java
new GregorianCalendar(TimeZone.getTimeZone("GMT+5:30"), new Locale("en", "IN"));
```

### 2.2。Java 8 的新方法

在 Java 8 中，`GregorianCalendar.`引入了新的方法

**`from()`方法从 ZonedDateTime 对象中获取一个默认地区的`GregorianCalendar`实例。**

使用`getCalendarType()`我们可以得到日历实例的类型。可用的日历类型有“格里高利”、“佛教”和“日本”。

例如，我们可以利用这一点来确保在继续我们的应用程序逻辑之前，我们有一个特定类型的日历:

```java
@Test
public void test_Calendar_Return_Type_Valid() {
    Calendar calendar = Calendar.getInstance();
    assert ("gregory".equals(calendar.getCalendarType()));
}
```

**调用`toZonedDateTime()`我们可以将日历对象转换成一个`ZonedDateTime`对象**，它代表时间线上与这个`GregorianCalendar.`相同的点

### 2.3。修改日期

可以使用`add()`、`roll()`和`set()`方法修改日历字段。

**`add()`方法允许我们根据日历的内部规则集以指定单位**向日历添加时间:

```java
@Test
public void test_whenAddOneDay_thenMonthIsChanged() {
    int finalDay1 = 1;
    int finalMonthJul = 6; 
    GregorianCalendar calendarExpected = new GregorianCalendar(2018, 5, 30);
    calendarExpected.add(Calendar.DATE, 1);
    System.out.println(calendarExpected.getTime());

    assertEquals(calendarExpected.get(Calendar.DATE), finalDay1);
    assertEquals(calendarExpected.get(Calendar.MONTH), finalMonthJul);
}
```

我们还可以使用`add()`方法从日历对象中减去时间:

```java
@Test
public void test_whenSubtractOneDay_thenMonthIsChanged() {
    int finalDay31 = 31;
    int finalMonthMay = 4; 
    GregorianCalendar calendarExpected = new GregorianCalendar(2018, 5, 1);
    calendarExpected.add(Calendar.DATE, -1);

    assertEquals(calendarExpected.get(Calendar.DATE), finalDay31);
    assertEquals(calendarExpected.get(Calendar.MONTH), finalMonthMay);
}
```

执行`add()`方法会强制立即重新计算日历的毫秒数和所有字段。

注意，使用`add()`也可能改变更高的日历字段(在本例中为月)。

`roll()`方法将一个有符号的数量添加到指定的日历字段中，而不改变更大的字段。较大的字段代表较大的时间单位。例如，`DAY_OF_MONTH`比`HOUR.`大

让我们看一个如何累计月份的示例。

在这种情况下，`YEAR`是一个较大的字段，不会递增:

```java
@Test
public void test_whenRollUpOneMonth_thenYearIsUnchanged() {
    int rolledUpMonthJuly = 7, orginalYear2018 = 2018;
    GregorianCalendar calendarExpected = new GregorianCalendar(2018, 6, 28);
    calendarExpected.roll(Calendar.MONTH, 1);

    assertEquals(calendarExpected.get(Calendar.MONTH), rolledUpMonthJuly);
    assertEquals(calendarExpected.get(Calendar.YEAR), orginalYear2018);
}
```

同样，我们可以向下滚动几个月:

```java
@Test
public void test_whenRollDownOneMonth_thenYearIsUnchanged() {
    int rolledDownMonthJune = 5, orginalYear2018 = 2018;
    GregorianCalendar calendarExpected = new GregorianCalendar(2018, 6, 28);
    calendarExpected.roll(Calendar.MONTH, -1);

    assertEquals(calendarExpected.get(Calendar.MONTH), rolledDownMonthJune);
    assertEquals(calendarExpected.get(Calendar.YEAR), orginalYear2018);
}
```

**我们可以使用`set()`方法直接将日历字段设置为指定值。**直到下一次调用`get()`、`getTime()`、`add()`或`roll()`时，才会重新计算日历的时间值(单位为毫秒)。

因此，多次调用`set()`不会触发不必要的计算。

让我们看一个将月份字段设置为 3(即四月)的示例:

```java
@Test
public void test_setMonth() {
    GregorianCalendarExample calendarDemo = new GregorianCalendarExample();
    GregorianCalendar calendarActual = new GregorianCalendar(2018, 6, 28);
    GregorianCalendar calendarExpected = new GregorianCalendar(2018, 6, 28);
    calendarExpected.set(Calendar.MONTH, 3);
    Date expectedDate = calendarExpected.getTime();

    assertEquals(expectedDate, calendarDemo.setMonth(calendarActual, 3));
}
```

### 2.4。与`XMLGregorianCalendar`一起工作

JAXB 允许将 Java 类映射到 XML 表示。`javax.xml.datatype.XMLGregorianCalendar`类型可以帮助映射基本的 XSD 模式类型，比如`xsd:date`、`xsd:time`和`xsd:dateTime`。

让我们看一个从`GregorianCalendar`类型转换成`XMLGregorianCalendar`类型的例子:

```java
@Test
public void test_toXMLGregorianCalendar() throws Exception {
    GregorianCalendarExample calendarDemo = new GregorianCalendarExample();
    DatatypeFactory datatypeFactory = DatatypeFactory.newInstance();
    GregorianCalendar calendarActual = new GregorianCalendar(2018, 6, 28);
    GregorianCalendar calendarExpected = new GregorianCalendar(2018, 6, 28);
    XMLGregorianCalendar expectedXMLGregorianCalendar = datatypeFactory
      .newXMLGregorianCalendar(calendarExpected);

    assertEquals(
      expectedXMLGregorianCalendar, 
      alendarDemo.toXMLGregorianCalendar(calendarActual));
}
```

一旦 calendar 对象被转换成 XML 格式，就可以在任何需要序列化日期的用例中使用，比如消息传递或 web 服务调用。

让我们看一个如何从`XMLGregorianCalendar`类型转换回`GregorianCalendar`类型的例子:

```java
@Test
public void test_toDate() throws DatatypeConfigurationException {
    GregorianCalendar calendarActual = new GregorianCalendar(2018, 6, 28);
    DatatypeFactory datatypeFactory = DatatypeFactory.newInstance();
    XMLGregorianCalendar expectedXMLGregorianCalendar = datatypeFactory
      .newXMLGregorianCalendar(calendarActual);
    expectedXMLGregorianCalendar.toGregorianCalendar().getTime();
    assertEquals(
      calendarActual.getTime(), 
      expectedXMLGregorianCalendar.toGregorianCalendar().getTime() );
}
```

### 2.5。比较日期

我们可以使用`Calendar`类的`compareTo()`方法来比较日期。如果基准日期在未来，则结果为正，如果基准数据在我们比较的日期的过去，则结果为负:

```java
@Test
public void test_Compare_Date_FirstDate_Greater_SecondDate() {
    GregorianCalendar firstDate = new GregorianCalendar(2018, 6, 28);
    GregorianCalendar secondDate = new GregorianCalendar(2018, 5, 28);
    assertTrue(1 == firstDate.compareTo(secondDate));
}

@Test
public void test_Compare_Date_FirstDate_Smaller_SecondDate() {
    GregorianCalendar firstDate = new GregorianCalendar(2018, 5, 28);
    GregorianCalendar secondDate = new GregorianCalendar(2018, 6, 28);
    assertTrue(-1 == firstDate.compareTo(secondDate));
}

@Test
public void test_Compare_Date_Both_Dates_Equal() {
    GregorianCalendar firstDate = new GregorianCalendar(2018, 6, 28);
    GregorianCalendar secondDate = new GregorianCalendar(2018, 6, 28);
    assertTrue(0 == firstDate.compareTo(secondDate));
}
```

### 2.6。格式化日期

我们可以通过使用`ZonedDateTime`和`DateTimeFormatter`的组合将`GregorianCalendar`转换成特定的格式，以获得想要的输出:

```java
@Test
public void test_dateFormatdMMMuuuu() {
    String expectedDate = new GregorianCalendar(2018, 6, 28).toZonedDateTime()
      .format(DateTimeFormatter.ofPattern("d MMM uuuu"));
    assertEquals("28 Jul 2018", expectedDate);
}
```

### 2.7。获取关于日历的信息

`GregorianCalendar`提供了几个 get 方法，可以用来获取不同的日历属性。让我们看看我们有哪些不同的选择:

*   **`getActualMaximum(int field)`–**返回指定日历字段的最大值，并考虑当前时间值。以下示例将为`DAY_OF_MONTH`字段返回值 30，因为六月有 30 天:

    ```java
    GregorianCalendar calendar = new GregorianCalendar(2018 , 5, 28);
    assertTrue(30 == calendar.getActualMaximum(calendar.DAY_OF_MONTH));
    ```

*   **`getActualMinimum(int field)`–**返回指定日历字段的最小值，并考虑当前时间值:

    ```java
    GregorianCalendar calendar = new GregorianCalendar(2018 , 5, 28);
    assertTrue(1 == calendar.getActualMinimum(calendar.DAY_OF_MONTH));
    ```

*   **`getGreatestMinimum(int field)`–**返回给定日历字段的最大最小值:

    ```java
    GregorianCalendar calendar = new GregorianCalendar(2018 , 5, 28);
    assertTrue(1 == calendar.getGreatestMinimum(calendar.DAY_OF_MONTH));
    ```

*   **`getLeastMaximum(int field)`–**返回给定日历字段的最小最大值。对于`DAY_OF_MONTH`字段，这是 28，因为二月可能只有 28 天:

    ```java
    GregorianCalendar calendar = new GregorianCalendar(2018 , 5, 28);
    assertTrue(28 == calendar.getLeastMaximum(calendar.DAY_OF_MONTH));
    ```

*   **`getMaximum(int field)`–**返回给定日历字段的最大值:

    ```java
    GregorianCalendar calendar = new GregorianCalendar(2018 , 5, 28);
    assertTrue(31 == calendar.getMaximum(calendar.DAY_OF_MONTH));
    ```

*   **`getMinimum(int field)`–**返回给定日历字段的最小值:

    ```java
    GregorianCalendar calendar = new GregorianCalendar(2018 , 5, 28);
    assertTrue(1 == calendar.getMinimum(calendar.DAY_OF_MONTH));
    ```

*   **`getWeekYear()`–**返回该`GregorianCalendar` :

    ```java
    GregorianCalendar calendar = new GregorianCalendar(2018 , 5, 28);
    assertTrue(2018 == calendar.getWeekYear());
    ```

    所代表的年的星期
*   **`getWeeksInWeekYear()`–**返回日历年中一周的周数:

    ```java
    GregorianCalendar calendar = new GregorianCalendar(2018 , 5, 28);
    assertTrue(52 == calendar.getWeeksInWeekYear());
    ```

*   **`isLeapYear()`–**如果年份是闰年，则返回 true:

    ```java
    GregorianCalendar calendar = new GregorianCalendar(2018 , 5, 28);
    assertTrue(false == calendar.isLeapYear(calendar.YEAR));
    ```

## 3。结论

在本文中，我们探索了`GregorianCalendar`的某些方面。

与往常一样，GitHub 上的示例代码[可用。](https://web.archive.org/web/20220628093933/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-date-operations-1)