# Java 时钟类指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-clock>

## 1。概述

在本教程中，我们将从`java.time`包中的**查看 Java `Clock`类。我们将解释什么是`Clock`类以及我们如何使用它。**

## 2。`Clock`班

`Clock`在 Java 8 中添加，使用最佳可用系统时钟提供对某一时刻的访问，并用作时间提供者，可有效地用于测试目的。

当前的日期和时间取决于时区，对于全球化的应用程序，时间提供者是必要的，以确保日期和时间是用正确的时区创建的。

这个类帮助我们测试我们的代码更改在不同的时区工作，或者——当使用固定时钟时——时间不影响我们的代码。

`Clock`类是抽象的，所以我们不能创建它的实例。可以使用以下工厂方法:

*   **`offset(Clock, Duration)`–**返回偏移指定`Duration`的时钟。其主要用例是模拟未来或过去的运行
*   **`systemUTC() – `** 返回代表 UTC 时区的时钟
*   **`fixed(Instant, ZoneId) – `** 总是返回相同的`Instant`。最主要的用例是在测试中，固定的时钟确保测试不依赖于当前的时钟

我们将研究`Clock`类中大多数可用的方法。

### 2.1.`instant()`

此方法返回表示时钟定义的当前时刻的时刻:

```
Clock clock = Clock.systemDefaultZone();
Instant instant = clock.instant();
System.out.println(instant);
```

将产生:

```
2018-04-07T03:59:35.555Z
```

### 2.2.`systemUTC()`

该方法返回一个代表 UTC 时区当前时刻的`Clock`对象:

```
Clock clock = Clock.systemUTC();
System.out.println("UTC time :: " + clock.instant());
```

将产生:

```
UTC time :: 2018-04-04T17:40:12.353Z
```

### 2.3.`system()`

这个静态方法返回由给定时区 id 标识的时区的`Clock`对象:

```
Clock clock = Clock.system(ZoneId.of("Asia/Calcutta"));
System.out.println(clock.instant());
```

将产生:

```
2018-04-04T18:00:31.376Z
```

### 2.4.`systemDefaultZone()`

这个静态方法返回一个代表当前时刻的`Clock`对象，并使用运行它的系统的默认时区:

```
Clock clock = Clock.systemDefaultZone();
System.out.println(clock);
```

上述行产生以下结果(假设我们的默认时区是“亚洲/加尔各答”):

```
SystemClock[Asia/Calcutta]
```

我们可以通过传递`ZoneId.systemDefault()`来实现相同的行为:

```
Clock clock = Clock.system(ZoneId.systemDefault());
```

### 2.5.`millis()`

这个方法以毫秒为单位返回时钟的当前时刻。提供**是为了允许在高性能用例中使用时钟，在这些用例中创建对象是不可接受的**。这种方法可以用在我们本该使用`System.currentTimeInMillis()`的地方:

```
Clock clock = Clock.systemDefaultZone();
System.out.println(clock.millis());
```

将产生:

```
1523104441258
```

### 2.6.`offset()`

这个静态方法从指定的基础时钟返回一个添加了指定持续时间的瞬间。

如果持续时间为负，则产生的时钟时刻将早于给定的基础时钟。

**使用`offset`，我们可以得到给定基准时钟的过去和未来的瞬间。**如果我们传递一个零持续时间，那么我们将得到与给定基础时钟相同的时钟:

```
Clock baseClock = Clock.systemDefaultZone();

// result clock will be later than baseClock
Clock clock = Clock.offset(baseClock, Duration.ofHours(72));
System.out.println(clock5.instant());

// result clock will be same as baseClock                           
clock = Clock.offset(baseClock, Duration.ZERO);
System.out.println(clock.instant());

// result clock will be earlier than baseClock            
clock = Clock.offset(baseClock, Duration.ofHours(-72));
System.out.println(clock.instant());
```

将产生:

```
2018-04-10T13:24:07.347Z
2018-04-07T13:24:07.348Z
2018-04-04T13:24:07.348Z
```

### 2.7.`tick()`

这个静态方法返回从指定时钟**舍入到指定持续时间**的最近事件的瞬间。指定的时钟持续时间必须是正数:

```
Clock clockDefaultZone = Clock.systemDefaultZone();
Clock clocktick = Clock.tick(clockDefaultZone, Duration.ofSeconds(30));

System.out.println("Clock Default Zone: " + clockDefaultZone.instant());
System.out.println("Clock tick: " + clocktick.instant());
```

将产生:

```
Clock Default Zone: 2018-04-07T16:42:05.473Z
Clock tick: 2018-04-07T16:42:00Z
```

### 2.8.`tickSeconds()`

这个静态方法返回给定时区的当前时刻，以整秒为单位。该时钟将始终将`nano-of-second`字段设置为零:

```
ZoneId zoneId = ZoneId.of("Asia/Calcutta");
Clock clock = Clock.tickSeconds(zoneId);
System.out.println(clock.instant());
```

将产生:

```
2018-04-07T17:40:23Z
```

同样可以通过使用`tick()`来实现:

```
Clock clock = Clock.tick(Clock.system(ZoneId.of("Asia/Calcutta")), Duration.ofSeconds(1));
```

### 2.9.`tickMinutes()`

这个静态方法返回指定时区的时钟时刻，以整分钟为单位。该时钟总是将`nano-of-second`和`second-of-minute`字段设置为零:

```
ZoneId zoneId = ZoneId.of("Asia/Calcutta");
Clock clock = Clock.tickMinutes(zoneId);
System.out.println(clock.instant());
```

将产生:

```
2018-04-07T17:26:00Z
```

同样可以通过使用`tick()`来实现:

```
Clock clock = Clock.tick(Clock.system(ZoneId.of("Asia/Calcutta")), Duration.ofMinutes(1));
```

### 2.10.`withZone()`

此方法返回此时钟的不同时区的副本。

如果我们有一个特定时区时钟实例，我们可以为不同的时区复制一个时钟:

```
ZoneId zoneSingapore = ZoneId.of("Asia/Singapore");  
Clock clockSingapore = Clock.system(zoneSingapore); 
System.out.println(clockSingapore.instant());

ZoneId zoneCalcutta = ZoneId.of("Asia/Calcutta");
Clock clockCalcutta = clockSingapore.withZone(zoneCalcutta);
System.out.println(clockCalcutta.instant());
```

将产生:

```
2018-04-07T17:55:43.035Z
2018-04-07T17:55:43.035Z
```

### 2.11.`getZone()`

该方法返回给定`Clock`的时区。

```
Clock clock = Clock.systemDefaultZone();
ZoneId zone = clock.getZone();
System.out.println(zone.getId());
```

将产生:

```
Asia/Calcutta
```

### `2.12\. fixed()`

这个方法返回的时钟**总是返回同一个时刻**。这种方法的主要用例是在测试中，固定时钟确保测试不依赖于当前时钟。

```
Clock fixedClock = Clock.fixed(Instant.parse("2018-04-29T10:15:30.00Z"),
ZoneId.of("Asia/Calcutta"));
System.out.println(fixedClock);
```

将产生:

```
FixedClock[2018-04-29T10:15:30Z,Asia/Calcutta]
```

## 3。结论

在本文中，我们深入探讨了 Java `Clock`类以及我们可以通过可用的方法使用它的不同方式。

与往常一样，GitHub 上的[提供了代码示例。](https://web.archive.org/web/20221208143837/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-time-measurements)