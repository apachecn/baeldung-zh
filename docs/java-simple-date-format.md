# 简单日期格式指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-simple-date-format>

## 1。简介

在本教程中，我们将对`SimpleDateFormat `类进行一次**的深度旅行。**

我们将看看**简单的实例化** **和格式化样式**，以及该类为**处理地区和时区**公开的有用方法。

## 2。简单实例化

首先，让我们看看如何实例化一个新的`SimpleDateFormat `对象。

有 [4 个可能的构造函数](https://web.archive.org/web/20220625080956/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/text/SimpleDateFormat.html#constructor.summary)——但是为了和名字保持一致，让我们把事情简单化。我们需要开始的是我们想要的的[日期模式](#date_time_patterns)的 **`String `表示。**

让我们从破折号分隔的日期模式开始，如下所示:

```java
"dd-MM-yyyy"
```

这将正确地设置日期格式，从当月的当天开始，到今年的当月，最后到今年。我们可以用一个简单的单元测试来测试我们的新格式化程序。我们将实例化一个新的`SimpleDateFormat `对象，并传入一个已知的日期:

```java
SimpleDateFormat formatter = new SimpleDateFormat("dd-MM-yyyy");
assertEquals("24-05-1977", formatter.format(new Date(233345223232L))); 
```

在上面的代码中，`formatter`将毫秒`l` `ong `转换为人类可读的日期——1977 年 5 月 24 日。

### 2.1。工厂方法

虽然`SimpleDateFormat`是一个可以快速构建日期格式化程序的方便的类，但是我们鼓励**在`DateFormat`类** `getDateFormat()`、`getDateTimeFormat()`、`getTimeFormat()`上使用工厂方法。

当使用这些工厂方法时，上面的示例看起来有些不同:

```java
DateFormat formatter = DateFormat.getDateInstance(DateFormat.SHORT);
assertEquals("5/24/77", formatter.format(new Date(233345223232L)));
```

从上面我们可以看出，格式化选项的数量是由`DateFormat `类上的[字段预先确定的。这在很大程度上**限制了我们可用的格式化选项**，这就是为什么我们在本文中坚持使用`SimpleDateFormat `。](https://web.archive.org/web/20220625080956/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/text/DateFormat.html#field.detail)

### 2.2.线程安全

`SimpleDateFormat `的 [JavaDoc](https://web.archive.org/web/20220625080956/https://github.com/openjdk/jdk/blob/76507eef639c41bffe9a4bb2b8a5083291f41383/src/java.base/share/classes/java/text/SimpleDateFormat.java#L427) 明确声明:

> 日期格式不同步。建议为每个线程创建单独的格式实例。如果多个线程同时访问一种格式，它必须在外部同步。

**所以`SimpleDateFormat `实例不是线程安全的**，我们应该在并发环境中小心使用它们。

解决这个问题的最佳方法是将它们与`[ThreadLocal](/web/20220625080956/https://www.baeldung.com/java-threadlocal)`结合使用。**这样，每个线程都有自己的`SimpleDateFormat `** **实例，共享的缺乏使得程序线程安全:**

```java
private final ThreadLocal<SimpleDateFormat> formatter = ThreadLocal
  .withInitial(() -> new SimpleDateFormat("dd-MM-yyyy"));
```

`[withInitial](https://web.archive.org/web/20220625080956/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/ThreadLocal.html#withInitial(java.util.function.Supplier)) `方法的参数是`SimpleDateFormat `实例的提供者。每次`ThreadLocal `需要创建一个实例时，它都会使用这个供应商。

然后我们可以通过`ThreadLocal`实例使用格式化程序:

```java
formatter.get().format(date)
```

`ThreadLocal.get() `方法首先初始化当前线程的`SimpleDateFormat `,然后重用该实例。

**我们称这种技术为`thread confinement`,因为我们将每个实例的使用限制在一个特定的线程上。**

还有另外两种方法可以解决同样的问题:

*   使用`synchronized `块或`ReentrantLock` s
*   按需创建丢弃的`SimpleDateFormat `实例

这两种方法都不推荐:前者在争用率高时会导致严重的性能下降，而后者会创建大量对象，给垃圾收集带来压力。

值得一提的是，**从 Java 8 开始，引入了一个新的`[DateTimeFormatter](/web/20220625080956/https://www.baeldung.com/java-datetimeformatter)`类**。新的`DateTimeFormatter` 类是**不可变的和线程安全的。如果我们使用 Java 8 或更高版本，建议使用新的`DateTimeFormatter` 类。**

## 3。解析日期

`SimpleDateFormat `和`DateFormat `不仅允许我们格式化日期——还可以反向操作。使用`parse `方法，我们可以**输入日期的`String `表示并返回`Date `** 对象的等价物:

```java
SimpleDateFormat formatter = new SimpleDateFormat("dd-MM-yyyy");
Date myDate = new Date(233276400000L);
Date parsedDate = formatter.parse("24-05-1977");
assertEquals(myDate.getTime(), parsedDate.getTime());
```

这里需要注意的是，构造函数中提供的**模式应该与使用`parse `方法解析的**日期格式相同。

## 4。日期时间模式

在格式化日期时，提供了大量不同的选项。虽然完整的列表可以在 [JavaDocs](https://web.archive.org/web/20220625080956/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/text/SimpleDateFormat.html) 中找到，但是让我们探索一些更常用的选项:

| 信 | 日期部分 | 例子 |
| M | 月 | 12;十二月 |
| y | 年 | Ninety-four |
| d | 天 | 23;孟人 |
| H | 小时 | 03 |
| m | 分钟 | Fifty-seven |

日期组件返回的**输出也很大程度上依赖于在`String`中使用的字符数**。例如，让我们以六月为例。如果我们将日期字符串定义为:

```java
"MM"
```

那么我们的结果将显示为数字代码–06。但是，如果我们在日期字符串中添加另一个 M:

```java
"MMM"
```

然后我们得到的格式化日期显示为单词`Jun`。

## 5。应用区域设置

`SimpleDateFormat `类也**支持广泛的区域设置**，这是在调用构造函数时设置的。

让我们通过用法语格式化日期来实践这一点。我们将实例化一个`SimpleDateFormat `对象，同时将`Locale.FRANCE `提供给构造函数。

```java
SimpleDateFormat franceDateFormatter = new SimpleDateFormat("EEEEE dd-MMMMMMM-yyyy", Locale.FRANCE);
Date myWednesday = new Date(1539341312904L);
assertTrue(franceDateFormatter.format(myWednesday).startsWith("vendredi"));
```

通过提供一个给定的日期，一个星期三的下午，我们可以断言我们的`franceDateFormatter `已经正确地格式化了日期。新日期正确地以`Vendredi `开始——法语是星期三！

值得注意的是，在构造函数–**的[地区版本中有一个小问题，虽然支持许多地区，但不能保证完全覆盖](https://web.archive.org/web/20220625080956/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/text/SimpleDateFormat.html#%3Cinit%3E(java.lang.String,java.util.Locale))**。Oracle 建议在`DateFormat `类上使用工厂方法，以确保语言环境覆盖。

## 6。改变时区

由于`SimpleDateFormat `扩展了`DateFormat `类，我们也可以**使用`setTimeZone `方法**来操纵时区。让我们来看看这是怎么回事:

```java
Date now = new Date();

SimpleDateFormat simpleDateFormat = new SimpleDateFormat("EEEE dd-MMM-yy HH:mm:ssZ");

simpleDateFormat.setTimeZone(TimeZone.getTimeZone("Europe/London"));
logger.info(simpleDateFormat.format(now));

simpleDateFormat.setTimeZone(TimeZone.getTimeZone("America/New_York"));
logger.info(simpleDateFormat.format(now));
```

在上面的例子中，我们为同一个`SimpleDateFormat` 对象上的两个不同时区提供了相同的`Date `。我们还在模式`String` 的末尾添加了**‘Z’字符来表示时区差异**。然后为用户记录来自`format `方法的输出。

点击 run，我们可以看到与两个时区相关的当前时间:

```java
INFO: Friday 12-Oct-18 12:46:14+0100
INFO: Friday 12-Oct-18 07:46:14-0400
```

## 7。总结

在本教程中，我们深入探究了`SimpleDateFormat`的复杂性。

我们已经了解了如何用**实例化`SimpleDateFormat `** ，以及**模式`String `如何影响日期的格式化**。

在最终试验使用时区的**之前，我们尝试用**改变输出字符串的区域设置**。**

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220625080956/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-datetime-string)