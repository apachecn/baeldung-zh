# 将毫秒持续时间格式化为 HH:MM:SS

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-ms-to-hhmmss>

## 1.概观

持续时间是用小时、分钟、秒、毫秒等表示的时间量。我们可能希望将持续时间格式化成某种特定的时间模式。

我们可以通过在一些 JDK 库的帮助下编写自定义代码或者利用第三方库来实现这一点。

在这个快速教程中，我们将看看如何编写简单的代码来将给定的持续时间格式化为 HH:MM:SS 格式。

## 2.Java 解决方案

持续时间有多种表达方式——例如，以分钟、秒和毫秒为单位，或者用 Java `Duration`来表示，它有自己特定的格式。

本部分和后续部分将重点关注使用一些 JDK 库将时间间隔(运行时间)格式化为 HH:MM:SS，以毫秒为单位。出于示例目的，我们将 38114000 毫秒格式化为 10:35:14 (HH:MM:SS)。

### 2.1.`Duration`

**从 Java 8 开始，引入了 [`Duration`](/web/20220926202440/https://www.baeldung.com/java-period-duration) 类来处理不同单位的时间间隔。**`Duration`类附带了许多助手方法来从持续时间中获取小时、分钟和秒。

要使用`Duration` 类将间隔格式化为 HH:MM:SS，我们需要使用`Duration`类中的工厂方法`ofMillis` 从我们的间隔中初始化`Duration` 对象。这将把间隔转换成我们可以使用的`Duration`对象:

```java
Duration duration = Duration.ofMillis(38114000);
```

为了便于从秒到我们想要的单位的计算，我们需要得到我们的持续时间或间隔中的总秒数:

```java
long seconds = duration.getSeconds();
```

然后，一旦我们有了秒数，我们就为我们想要的格式生成相应的小时、分钟和秒:

```java
long HH = seconds / 3600;
long MM = (seconds % 3600) / 60;
long SS = seconds % 60;
```

最后，我们格式化生成的值:

```java
String timeInHHMMSS = String.format("%02d:%02d:%02d", HH, MM, SS);
```

让我们试试这个解决方案:

```java
assertThat(timeInHHMMSS).isEqualTo("10:35:14");
```

**如果我们使用的是 Java 9 或更高版本，我们可以使用一些助手方法直接获得单位，而不必执行任何计算**:

```java
long HH = duration.toHours();
long MM = duration.toMinutesPart();
long SS = duration.toSecondsPart();
String timeInHHMMSS = String.format("%02d:%02d:%02d", HH, MM, SS); 
```

上面的代码片段将给出与上面测试相同的结果:

```java
assertThat(timeInHHMMSS).isEqualTo("10:35:14");
```

### 2.2.`TimeUnit`

就像上一节讨论的`Duration `类一样，`TimeUnit`表示给定粒度的时间。它提供了一些帮助方法来转换单位——在我们的例子中是小时、分钟和秒——并在这些单位中执行计时和延迟操作。

要将以毫秒为单位的持续时间格式化为 HH:MM:SS 格式，我们需要做的就是使用`TimeUnit`中相应的助手方法:

```java
long HH = TimeUnit.MILLISECONDS.toHours(38114000);
long MM = TimeUnit.MILLISECONDS.toMinutes(38114000) % 60;
long SS = TimeUnit.MILLISECONDS.toSeconds(38114000) % 60;
```

然后，根据上面生成的单位格式化持续时间:

```java
String timeInHHMMSS = String.format("%02d:%02d:%02d", HH, MM, SS);
assertThat(timeInHHMMSS).isEqualTo("10:35:14");
```

## 3.使用第三方库

我们可以选择使用第三方库方法来尝试不同的途径，而不是自己编写。

### 3.1 .Apache common(Apache 公共)

为了使用 [Apache Commons](/web/20220926202440/https://www.baeldung.com/java-commons-lang-3) ，我们需要将 [commons-lang3](https://web.archive.org/web/20220926202440/https://search.maven.org/search?q=g:org.apache.commons%20a:commons-lang3) 添加到我们的项目中:

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

正如所料，这个库在它的`DurationFormatUtils`类中有`formatDuration `和其他单元格式化方法:

```java
String timeInHHMMSS = DurationFormatUtils.formatDuration(38114000, "HH:MM:SS", true);
assertThat(timeInHHMMSS).isEqualTo("10:35:14");
```

### 3.2.Joda 时间

当我们使用 Java 8 之前的 Java 版本时， [Joda Time](/web/20220926202440/https://www.baeldung.com/joda-time) 库就派上了用场，因为它有方便的助手方法来表示和格式化时间单位。为了使用 Joda 时间，让我们将 [joda 时间依赖关系](https://web.archive.org/web/20220926202440/https://search.maven.org/search?q=g:joda-time%20a:joda-time)添加到我们的项目中:

```java
<dependency>
    <groupId>joda-time</groupId>
    <artifactId>joda-time</artifactId>
    <version>2.10.10</version>
</dependency>
```

Joda Time 有一个`Duration`类来表示时间。首先，我们将以毫秒为单位的时间间隔转换为 Joda Time `Duration`对象的实例:

```java
Duration duration = new Duration(38114000);
```

然后，我们使用`Duration`中的`toPeriod `方法从上面的持续时间中获得周期，该方法将它转换或初始化为 Joda Time 中的`Period`类的实例:

```java
Period period = duration.toPeriod();
```

我们使用相应的助手方法从`Period`获取单位(小时、分钟和秒):

```java
long HH = period.getHours();
long MM = period.getMinutes();
long SS = period.getSeconds();
```

最后，我们可以格式化持续时间并测试结果:

```java
String timeInHHMMSS = String.format("%02d:%02d:%02d", HH, MM, SS);
assertThat(timeInHHMMSS).isEqualTo("10:35:14");
```

## 4.结论

在本教程中，我们学习了如何将持续时间格式化为特定的格式(在我们的例子中是 HH:MM:SS)。

首先，我们使用 Java 自带的`Duration `和`TimeUnit `类来获得所需的单元，并在`Formatter`的帮助下格式化它们。

最后，我们看了如何使用一些第三方库来实现这个结果。

和往常一样，完整的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220926202440/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-datetime-string)