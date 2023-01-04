# Java 中的周期和持续时间

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-period-duration>

## 1。概述

在这个快速教程中，我们将看看 Java 8 中引入的两个处理日期的新类:`Period`和`Duration`。

这两个类都可以用来表示时间量或确定两个日期之间的差异。这两个类的主要区别在于，`Period`使用基于日期的值，而`Duration`使用基于时间的值。

## 2。`Period`阶级

`Period`类使用年、月和日来表示一段时间。

我们可以通过使用`between()`方法获得一个`Period`对象作为两个日期之间的差值:

```
LocalDate startDate = LocalDate.of(2015, 2, 20);
LocalDate endDate = LocalDate.of(2017, 1, 15);

Period period = Period.between(startDate, endDate);
```

然后，我们可以使用方法`getYears(), getMonths(), getDays()`确定周期的日期单位:

```
LOG.info("Years:" + period.getYears() + 
  " months:" + period.getMonths() + 
  " days:"+period.getDays());
```

在这种情况下， `isNegative()`方法可用于确定`endDate`是否高于`startDate`，如果任何单位为负，该方法将返回`true`:

```
assertFalse(period.isNegative());
```

如果`isNegative()`返回假，那么`startDate`早于`endDate`值。

另一种**创建`Period`对象的方法是基于天数、月数、周数或年数**使用专用方法:

```
Period fromUnits = Period.of(3, 10, 10);
Period fromDays = Period.ofDays(50);
Period fromMonths = Period.ofMonths(5);
Period fromYears = Period.ofYears(10);
Period fromWeeks = Period.ofWeeks(40);

assertEquals(280, fromWeeks.getDays());
```

如果只有一个值存在，例如通过使用`ofDays()`方法，则其他单位的值为 0。

在使用`ofWeeks()`方法的情况下，参数值通过乘以 7 来设置天数。

我们还可以通过解析文本序列来**创建一个`Period`对象，该文本序列的格式必须是“PnYnMnD”:**

```
Period fromCharYears = Period.parse("P2Y");
assertEquals(2, fromCharYears.getYears());

Period fromCharUnits = Period.parse("P2Y3M5D");
assertEquals(5, fromCharUnits.getDays());
```

周期值可以通过使用形式为`plusX()`和`minusX()`的方法来增加或减少，其中 X 代表日期单位:

```
assertEquals(56, period.plusDays(50).getDays());
assertEquals(9, period.minusMonths(2).getMonths());
```

## 3。`Duration`阶级

`Duration`类表示以秒或纳秒为单位的时间间隔，最适合在需要更高精度的情况下处理更短的时间。

我们可以使用 `between()`方法将两个瞬间的差异确定为一个`Duration`对象:

```
Instant start = Instant.parse("2017-10-03T10:15:30.00Z");
Instant end = Instant.parse("2017-10-03T10:16:30.00Z");

Duration duration = Duration.between(start, end);
```

然后我们可以使用`getSeconds()`或`getNanoseconds()`方法来确定时间单位的值:

```
assertEquals(60, duration.getSeconds());
```

或者，我们可以从两个 LocalDateTime 实例中获得一个 Duration 实例:

```
LocalTime start = LocalTime.of(1, 20, 25, 1024);
LocalTime end = LocalTime.of(3, 22, 27, 1544);

Duration.between(start, end).getSeconds();
```

`isNegative()`方法可用于验证结束时刻是否高于开始时刻:

```
assertFalse(duration.isNegative());
```

我们也可以**基于几个时间单位**获得一个`Duration`对象，使用方法`ofDays(), ofHours(), ofMillis(), ofMinutes(), ofNanos(), ofSeconds()`:

```
Duration fromDays = Duration.ofDays(1);
assertEquals(86400, fromDays.getSeconds());

Duration fromMinutes = Duration.ofMinutes(60);
```

要基于文本序列创建一个`Duration`对象，其格式必须为“PnDTnHnMn.nS”:

```
Duration fromChar1 = Duration.parse("P1DT1H10M10.5S");
Duration fromChar2 = Duration.parse("PT10M");
```

**可以使用`toDays(), toHours(), toMillis(), toMinutes()`将持续时间转换为其他时间单位**:

```
assertEquals(1, fromMinutes.toHours());
```

持续时间值可以通过使用形式为`plusX()`或`minusX()`的方法来增加或减少，其中 X 可以代表天、小时、毫秒、分钟、纳秒或秒:

```
assertEquals(120, duration.plusSeconds(60).getSeconds());     
assertEquals(30, duration.minusSeconds(30).getSeconds());
```

我们还可以使用带有参数的 `plus()`和`minus()`方法，该参数指定要加或减的`TemporalUnit`:

```
assertEquals(120, duration.plus(60, ChronoUnit.SECONDS).getSeconds());     
assertEquals(30, duration.minus(30, ChronoUnit.SECONDS).getSeconds());
```

## 4.结论

在本教程中，我们已经展示了如何使用`Period`和`Duration`类。

和往常一样，例子的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221129021232/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-8-datetime)