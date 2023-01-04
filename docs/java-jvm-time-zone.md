# 如何设置 JVM 时区

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-jvm-time-zone>

## 1.概观

当涉及到时间戳时，我们应用程序的用户可能会要求很高。他们希望我们的应用程序自动检测他们的时区，并在正确的时区显示时间戳。

在本教程中，我们将看看修改 JVM 时区的几种方法。我们还将了解一些与管理时区相关的陷阱。

## 2.时区介绍

默认情况下，JVM 从操作系统中读取时区信息。**该信息被传递给 [`TimeZone`](https://web.archive.org/web/20221108234443/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/TimeZone.html) 类，该类存储时区并计算夏令时**。

我们可以调用方法`getDefault,` ，它将返回程序运行的时区。此外，我们可以使用`TimeZone.getAvailableIDs()`从应用程序获得支持的时区 id 列表。

在命名时区时，Java 依赖于 [`tz database`](https://web.archive.org/web/20221108234443/https://en.wikipedia.org/wiki/List_of_tz_database_time_zones#List) 的命名约定。

## 3.更改时区

在这一节中，我们将看看在 JVM 中改变时区的几种方法。

### 3.1.设置环境变量

让我们从了解如何使用环境变量来更改时区开始。我们可以添加或修改环境变量`TZ.`

例如，在基于 Linux 的环境中，我们可以使用`export`命令:

```java
export TZ="America/Sao_Paulo"
```

设置环境变量后，我们可以看到我们正在运行的应用程序的时区现在是`America/Sao_Paulo:`

```java
Calendar calendar = Calendar.getInstance();
assertEquals(calendar.getTimeZone(), TimeZone.getTimeZone("America/Sao_Paulo"));
```

### 3.2.设置 JVM 参数

**设置环境变量的另一种方法是设置 JVM 参数`user.timezone`。**这个 JVM 参数优先于环境变量`TZ`。

例如，我们可以在运行应用程序时使用标志`-D` :

```java
java -Duser.timezone="Asia/Kolkata" com.company.Main
```

同样，我们也可以从应用程序中设置 JVM 参数:

```java
System.setProperty("user.timezone", "Asia/Kolkata");
```

我们现在可以看到时区是亚洲/加尔各答:

```java
Calendar calendar = Calendar.getInstance();
assertEquals(calendar.getTimeZone(), TimeZone.getTimeZone("Asia/Kolkata"));
```

### 3.3.从应用程序中设置时区

**最后，我们还可以使用`TimeZone`类**从应用程序中修改 JVM 时区。这种方法优先于环境变量和 JVM 参数。

设置默认时区很容易:

```java
TimeZone.setDefault(TimeZone.getTimeZone("Portugal"));
```

正如所料，时区现在是`Portugal`:

```java
Calendar calendar = Calendar.getInstance();
assertEquals(calendar.getTimeZone(), TimeZone.getTimeZone("Portugal"));
```

## 4.陷阱

### 4.1.使用三个字母的时区 id

尽管可以使用三个字母的 id 来表示时区，[但不建议使用](https://web.archive.org/web/20221108234443/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/TimeZone.html)。

相反，我们应该使用更长的名字，因为三个字母的 id 是不明确的。例如，IST 可以是印度标准时间、爱尔兰标准时间或以色列标准时间。

### 4.2.全局设置

请注意，上述每种方法都是为整个应用程序全局设置时区。然而，在现代应用程序中，设置时区往往比这更微妙。

例如，我们可能需要将时间转换成最终用户的时区，因此全球时区没有多大意义。如果不需要全局时区，可以考虑直接在每个日期时间实例上指定时区。无论是 [`ZonedDateTime`还是`OffsetDateTime`](/web/20221108234443/https://www.baeldung.com/java-zoneddatetime-offsetdatetime) 都是一个方便的类。

## 5.结论

在本教程中，我们解释了几种修改 JVM 时区的方法。我们看到，我们可以设置一个系统范围的环境变量，更改一个 JVM 参数，或者从我们的应用程序中以编程方式修改它。

和往常一样，本文中使用的所有例子都可以在 GitHub 上找到。