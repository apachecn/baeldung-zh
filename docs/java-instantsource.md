# Java 17 中的 InstantSource 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-instantsource>

## 1.概观

在本教程中，我们将深入 Java 17 中引入的`InstantSource`接口，该接口**提供了当前时刻**的可插拔表示，并避免引用时区。

## 2.`InstantSource`界面

正如我们在最初的[提案](https://web.archive.org/web/20220627185817/https://mail.openjdk.java.net/pipermail/core-libs-dev/2021-May/077213.html)和一个[相关问题](https://web.archive.org/web/20220627185817/https://github.com/ThreeTen/threeten-extra/issues/150)中看到的，这个接口的第一个目标是创建一个由`java.time.Clock`提供的时区的抽象。它还简化了在测试检索实例的代码部分时存根的创建。

它是在 Java 17 **中添加的，目的是提供一种安全的方式来访问当前时刻**，正如我们在下面的例子中看到的:

```java
class AQuickTest {
    InstantSource source;
    ...
    Instant getInstant() {
        return source.instant();
    }
}
```

然后，我们可以简单地得到一个瞬间:

```java
var quickTest = new AQuickTest(InstantSource.system());
quickTest.getInstant();
```

它的实现创建了可以在任何地方使用的对象来检索实例，并且它提供了一种有效的方法来创建用于测试目的的存根实现。

让我们更深入地了解一下使用这个接口的好处。

## 3.问题和解决方案

为了更好地理解`InstantSource`接口，让我们深入研究它所要解决的问题以及它提供的实际解决方案。

### 3.1.测试问题

测试涉及检索`Instant`的代码通常是一场噩梦，尤其是当获取那个瞬间的方法是基于当前的数据解决方案时，比如`LocalDateTime.now().`

为了让测试提供一个特定的日期，我们通常会创建一些变通办法，比如创建一个外部日期工厂，并在测试中提供一个存根实例。

让我们看看下面的代码，作为解决这个问题的一个例子。

`InstantExample`类使用一个`InstantWrapper`(或变通方法)来恢复一个瞬间:

```java
class InstantExample {
    InstantWrapper instantWrapper;
    Instant getCurrentInstantFromInstantWrapper() {
        return instantWrapper.instant();
    }
}
```

我们的`InstantWrapper`工作区类本身看起来是这样的:

```java
class InstantWrapper {
    Clock clock;
    InstantWrapper() {
        this.clock = Clock.systemDefaultZone();
    }
    InstantWrapper(ZonedDateTime zonedDateTime) {
        this.clock = Clock.fixed(zonedDateTime.toInstant(), zonedDateTime.getZone());
    }
    Instant instant() {
        return clock.instant();
    }
}
```

然后，我们可以用它来为测试提供一个固定的瞬间:

```java
// given
LocalDateTime now = LocalDateTime.now();
InstantExample tested = new InstantExample(InstantWrapper.of(now), null);
Instant currentInstant = now.toInstant(ZoneOffset.UTC);
// when
Instant returnedInstant = tested.getCurrentInstantFromWrapper();
// then
assertEquals(currentInstant, returnedInstant);
```

### 3.2.测试问题的解决方案

本质上，我们上面应用的解决方法就是`InstantSource`所做的。**它提供了一个`Instants`的外部工厂，我们可以在任何需要**的地方使用。Java 17 提供了一个默认的系统级实现(在`Clock`类中)，我们也可以提供自己的实现:

```java
class InstantExample {
    InstantSource instantSource;
    Instant getCurrentInstantFromInstantSource() {
        return instantSource.instant();
    }
}
```

`InstantSource`是可插拔的。也就是说，它可以使用依赖注入框架注入，或者只是作为构造函数参数传递到我们正在测试的对象中。因此，我们可以很容易地创建一个存根`InstantSource,` 提供给被测试的对象，并让它返回我们想要测试的时刻:

```java
// given
LocalDateTime now = LocalDateTime.now();
InstantSource instantSource = InstantSource.fixed(now.toInstant(ZoneOffset.UTC));
InstantExample tested = new InstantExample(null, instantSource);
Instant currentInstant = instantSource.instant();
// when
Instant returnedInstant = tested.getCurrentInstantFromInstantSource();
// then
assertEquals(currentInstant, returnedInstant);
```

### 3.3.时区问题

**当我们需要一个`Instant`的时候，我们有很多不同的地方可以从**得到，像`Instant.now()`、`Clock.systemDefaultZone().instant()`甚至`LocalDateTime.now.toInstant(zoneOffset)`。问题是，根据我们选择的口味，**可能会引入时区问题**。

例如，让我们看看当我们在`Clock`类上请求 instant 时会发生什么:

```java
Clock.systemDefaultZone().instant();
```

该代码将产生以下结果:

```java
2022-01-05T06:47:15.001890204Z
```

让我们从不同的来源问同样的问题:

```java
LocalDateTime.now().toInstant(ZoneOffset.UTC);
```

这会产生以下输出:

```java
2022-01-05T07:47:15.001890204Z
```

我们应该得到相同的瞬间，但事实上，两者之间有 60 分钟的差异。

最糟糕的是，可能有两个或更多的开发人员在代码的不同部分使用这两个即时源来处理同一代码。如果是这样，我们就有问题了。

在这一点上，我们通常不想处理时区问题。但是，为了创造这一瞬间，我们需要一个来源，而这个来源总是附带一个时区。

### 3.4.时区问题的解决方案

**`InstantSource`将我们从选择瞬间的来源**中抽象出来。这个选择已经为我们做好了。可能是另一个程序员已经建立了一个系统范围的定制实现，或者我们正在使用 Java 17 提供的实现，我们将在下一节中看到。

正如`InstantExample`所示，我们插上了一个`InstantSource`，我们不需要知道其他任何东西。我们可以去掉我们的`InstantWrapper`工作区，只使用插件`InstantSource `来代替。

现在我们已经看到了使用这个接口的好处，让我们通过查看它的静态和实例方法来看看它还能提供什么。

## 4.工厂方法

以下工厂方法可用于创建 InstantSource 对象:

*   `**system() –**` 默认全系统实施
*   `**tick(InstantSource, Duration) –**` 返回一个被截断为给定持续时间的最接近表示的`InstantSource`
***   `**fixed(Instant) – **`返回一个`InstantSource`，表示**总是产生同一个`Instant`***   `**offset(InstantSource, Duration) –**` 返回一个`InstantSource` ，由**向`Instant` s 提供给定的偏移量****

 **让我们看看这些方法的一些基本用法。

### 4.1.`system()`

Java 17 中当前默认的实现是`Clock.SystemInstantSource`类。

```java
Instant i = InstantSource.system().instant();
```

### 4.2.`tick()`

基于前面的例子:

```java
Instant i = InstantSource.system().instant();
System.out.println(i);
```

运行这段代码后，我们将得到以下输出:

```java
2022-01-05T07:44:44.861040341Z
```

但是，如果我们应用 2 小时的分笔成交点持续时间:

```java
Instant i = InstantSource.tick(InstantSource.system(), Duration.ofHours(2)).instant();
```

然后，我们会得到下面的结果:

```java
2022-01-05T06:00:00Z
```

### 4.3.`fixed()`

当我们需要创建一个存根`InstantSource`用于测试时，这种方法很方便:

```java
LocalDateTime fixed = LocalDateTime.of(2022, 1, 1, 0, 0);
Instant i = InstantSource.fixed(fixed.toInstant(ZoneOffset.UTC)).instant();
System.out.println(i);
```

上述内容总是返回相同的瞬间:

```java
2022-01-01T00:00:00Z
```

### 4.4.`offset()`

基于前面的例子，我们将对固定的`InstantSource`应用一个偏移量，看看它返回什么:

```java
LocalDateTime fixed = LocalDateTime.of(2022, 1, 1, 0, 0);
InstantSource fixedSource = InstantSource.fixed(fixed.toInstant(ZoneOffset.UTC));
Instant i = InstantSource.offset(fixedSource, Duration.ofDays(5)).instant();
System.out.println(i);
```

执行此代码后，我们将获得以下输出:

```java
2022-01-06T00:00:00Z
```

## 5.实例方法

可用于与`InstantSource` 实例交互的方法有`:`

*   `**instant() – **`返回`InstantSource`给定的电流`Instant`
*   `**millis() –**` 返回由`InstantSource`提供的当前`Instant`的毫秒表示
*   `**withZone(ZoneId) –**` 接收一个`ZoneId`并根据给定的`InstantSource`和指定的`ZoneId` 返回一个**时钟**

### 5.1.`instant()`

这种方法的最基本用法是:

```java
Instant i = InstantSource.system().instant();
System.out.println(i);
```

运行此代码将向我们显示以下输出:

```java
2022-01-05T08:29:17.641839778Z
```

### 5.2.`millis()`

为了从一个`InstantSource`获得纪元:

```java
long m = InstantSource.system().millis();
System.out.println(m);
```

运行之后，我们将得到以下结果:

```java
1641371476655
```

### 5.3.`withZone()`

让我们为特定的`ZoneId`获取一个`Clock`实例:

```java
Clock c = InstantSource.system().withZone(ZoneId.of("-4"));
System.out.println(c);
```

这将简单地打印以下内容:

```java
SystemClock[-04:00]
```

## 6.结论

在本文中，我们浏览了`InstantSource`接口，列举了创建它来解决的重要问题，并展示了我们如何在日常工作中利用它的真实例子。

像往常一样，代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220627185817/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-17)**