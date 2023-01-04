# 覆盖 Java 测试的系统时间

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-override-system-time>

## 1.概观

在这个快速教程中，我们将关注覆盖测试的系统时间的**不同方法。**

有时在我们的代码中有一个关于当前日期的逻辑。也许一些函数调用比如`new Date()`或者`Calendar.getInstance()`，最终会调用`System.CurrentTimeMillis`。

关于`Java Clock`的使用介绍，请参考[这里的](/web/20221129012227/https://www.baeldung.com/java-clock)这篇文章。或者，对 AspectJ 的使用，[这里是](/web/20221129012227/https://www.baeldung.com/aspectj)。

## 2.使用时钟输入 `java.time`

**`Java 8`中的`java.time`包包含一个抽象类`java.time.Clock`** ，目的是允许在需要时插入备用时钟。有了它，我们可以插入我们自己的实现或者找到一个已经满足我们需要的实现。

为了实现我们的目标，**上面的库包含了静态方法来产生特殊的实现**。我们将使用其中的两个，它们返回一个不可变的、线程安全的和可序列化的实现。

第一个是`fixed`。从中，**我们可以获得一个`Clock`，它总是返回相同的** `**Instant**, `，确保测试不依赖于当前时钟。

要使用它，我们需要一个`Instant`和一个`ZoneOffset`:

```java
Instant.now(Clock.fixed( 
  Instant.parse("2018-08-22T10:00:00Z"),
  ZoneOffset.UTC))
```

**第二种静法是`offset`** 。在这个例子中，一个时钟包装了另一个时钟，使得返回的对象能够获得比指定持续时间晚或早的时刻。

换句话说，**可以模拟未来、过去或任意时间点的跑步**:

```java
Clock constantClock = Clock.fixed(ofEpochMilli(0), ZoneId.systemDefault());

// go to the future:
Clock clock = Clock.offset(constantClock, Duration.ofSeconds(10));

// rewind back with a negative value:
clock = Clock.offset(constantClock, Duration.ofSeconds(-5));

// the 0 duration returns to the same clock:
clock = Clock.offset(constClock, Duration.ZERO);
```

使用`Duration`类，可以从纳秒到天进行操作。此外，我们可以否定一个持续时间，这意味着获得一个长度被否定的持续时间的副本。

## 3.使用面向方面的编程

另一种覆盖系统时间的方法是通过 AOP。使用这种方法，**我们能够编织`System`类来返回一个预定义的值，我们可以在我们的测试用例**中设置这个值。

此外，可以编织应用程序类，将对`System.currentTimeMillis()`或`new Date()`的调用重定向到我们自己的另一个实用程序类。

实现这一点的一种方法是使用 AspectJ:

```java
public aspect ChangeCallsToCurrentTimeInMillisMethod {
    long around(): 
      call(public static native long java.lang.System.currentTimeMillis()) 
        && within(user.code.base.pckg.*) {
          return 0;
      }
} 
```

在上面的例子中，**我们在一个指定的包**中捕捉每个对`System.currentTimeMillis`()的调用，在这个例子中是`user.code.base.pckg.*`、**，并且每当这个事件发生**时返回 0。

正是在这个地方，我们可以声明自己的实现来获得以毫秒为单位的所需时间。

使用 AspectJ 的一个优点是，它直接在字节码级别上操作，因此不需要原始源代码就可以工作。

因此，我们不需要重新编译它。

## 4。嘲讽`Instant.now()`法

我们可以用 [`Instant`](/web/20221129012227/https://www.baeldung.com/current-date-time-and-timestamp-in-java-8) 类来表示时间轴上的一个瞬间点。通常，我们可以用它来记录应用程序中的事件时间戳。这个类的`now()`方法允许我们从 UTC 时区的系统时钟中获取当前时刻。

让我们看看在测试时改变其行为的一些替代方法。

### 4.1。用`Clock` 重载`now()`

我们可以用一个固定的 [`Clock`](/web/20221129012227/https://www.baeldung.com/java-clock) 实例重载`now()`方法。**在`java.time`包中的许多类都有一个`now()`方法，该方法带有一个`Clock`参数**，这使得它成为我们的首选方法:

```java
@Test
public void givenFixedClock_whenNow_thenGetFixedInstant() {
    String instantExpected = "2014-12-22T10:15:30Z";
    Clock clock = Clock.fixed(Instant.parse(instantExpected), ZoneId.of("UTC"));

    Instant instant = Instant.now(clock);

    assertThat(instant.toString()).isEqualTo(instantExpected);
}
```

### 4.2。使用`Mockito`

此外，如果我们需要在不发送参数的情况下修改`now()`方法的行为，我们可以使用`[Mockito](/web/20221129012227/https://www.baeldung.com/mockito-mock-static-methods)`:

```java
@Test
public void givenInstantMock_whenNow_thenGetFixedInstant() {
    String instantExpected = "2014-12-22T10:15:30Z";
    Clock clock = Clock.fixed(Instant.parse(instantExpected), ZoneId.of("UTC"));
    Instant instant = Instant.now(clock);

    try (MockedStatic<Instant> mockedStatic = mockStatic(Instant.class)) {
        mockedStatic.when(Instant::now).thenReturn(instant);
        Instant now = Instant.now();
        assertThat(now.toString()).isEqualTo(instantExpected);
    }
}
```

### 4.3。使用 `JMockit`

或者，我们可以使用 [`JMockit`](/web/20221129012227/https://www.baeldung.com/jmockit-101) 库。

`JMockit`为我们提供了两种[嘲讽静态方法的方式](/web/20221129012227/https://www.baeldung.com/jmockit-static-method)。一个是使用`MockUp`类:

```java
@Test
public void givenInstantWithJMock_whenNow_thenGetFixedInstant() {
    String instantExpected = "2014-12-21T10:15:30Z";
    Clock clock = Clock.fixed(Instant.parse(instantExpected), ZoneId.of("UTC"));
    new MockUp<Instant>() {
        @Mock
        public Instant now() {
            return Instant.now(clock);
        }
    };

    Instant now = Instant.now();

    assertThat(now.toString()).isEqualTo(instantExpected);
}
```

而另一个是使用 [`Expectations`](/web/20221129012227/https://www.baeldung.com/jmockit-expectations) 的类:

```java
@Test
public void givenInstantWithExpectations_whenNow_thenGetFixedInstant() {
    Clock clock = Clock.fixed(Instant.parse("2014-12-23T10:15:30.00Z"), ZoneId.of("UTC"));
    Instant instantExpected = Instant.now(clock);
    new Expectations(Instant.class) {
        {
            Instant.now();
            result = instantExpected;
        }
    };

    Instant now = Instant.now();

    assertThat(now).isEqualTo(instantExpected);
}
```

## 5。嘲讽`LocalDateTime.now()`法

`java.time`包中另一个有用的类是 [`LocalDateTime`](/web/20221129012227/https://www.baeldung.com/java-8-date-time-intro) 类。此类表示 ISO-8601 日历系统中不带时区的日期时间。这个类的`now()`方法允许我们从默认时区的系统时钟中获取当前的日期时间。

我们可以使用与之前看到的相同的替代方案来模拟它。比如用固定的`Clock`重载`now()`:

```java
@Test
public void givenFixedClock_whenNow_thenGetFixedLocalDateTime() {
    Clock clock = Clock.fixed(Instant.parse("2014-12-22T10:15:30.00Z"), ZoneId.of("UTC"));
    String dateTimeExpected = "2014-12-22T10:15:30";

    LocalDateTime dateTime = LocalDateTime.now(clock);

    assertThat(dateTime).isEqualTo(dateTimeExpected);
}
```

## 6。结论

在本文中，我们探索了覆盖测试系统时间的不同方法。首先，我们看了原生包`java.time`和它的`Clock`类。接下来，我们看到了如何应用方面来编织`System`类。最后，我们看到了在`Instant`和`LocalDateTime`类上模仿`now()`方法的不同选择。

As always, code samples can be found [over on GitHub](https://web.archive.org/web/20221129012227/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-time-measurements).