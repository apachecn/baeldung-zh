# Java 12 中的新特性

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-12-new-features>

[This article is part of a series:](javascript:void(0);)[• New Features in Java 8](/web/20220824083948/https://www.baeldung.com/java-8-new-features)
[• New Features in Java 9](/web/20220824083948/https://www.baeldung.com/new-java-9)
[• New Features in Java 10](/web/20220824083948/https://www.baeldung.com/java-10-overview)
[• New Features in Java 11](/web/20220824083948/https://www.baeldung.com/java-11-new-features)
• New Features in Java 12 (current article)[• New Features in Java 13](/web/20220824083948/https://www.baeldung.com/java-13-new-features)
[• New Features in Java 14](/web/20220824083948/https://www.baeldung.com/java-14-new-features)
[• What's New in Java 15](/web/20220824083948/https://www.baeldung.com/java-15-new)
[• New Features in Java 16](/web/20220824083948/https://www.baeldung.com/java-16-new-features)
[• New Features in Java 17](/web/20220824083948/https://www.baeldung.com/java-17-new-features)

## 1.介绍

在本教程中，我们将对 Java 12 的一些新特性进行快速、高层次的概述。官方文档中提供了所有新特性的完整列表。

## 2.语言变化和功能

Java 12 引入了许多新的语言特性。在这一节中，我们将讨论几个最有趣的例子，并提供代码示例以便更好地理解。

### 2.1.字符串类新方法

Java 12 在[和`String`类](/web/20220824083948/https://www.baeldung.com/java-string)中提供了两个新方法。

第一个–`indent`根据整数参数调整每行的缩进。如果参数大于零，将在每一行的开头插入新的空格。另一方面，如果参数小于零，它会从每一行的乞求中移除空格。如果给定行没有包含足够的空白，则所有前导空白字符都将被删除。

现在，让我们看一个基本的例子。首先，我们将文本缩进四个空格，然后我们将删除整个缩进:

```java
String text = "Hello Baeldung!\nThis is Java 12 article.";

text = text.indent(4);
System.out.println(text);

text = text.indent(-10);
System.out.println(text);
```

输出如下所示:

```java
 Hello Baeldung!
    This is Java 12 article.

Hello Baeldung!
This is Java 12 article.
```

注意，即使我们传递了值-10，这超过了我们的缩进量，也只有空格受到影响。其他字符保持不变。

第二种新方法是`transform`。它接受单个实参函数作为将应用于字符串的参数。

例如，让我们使用 transform 方法来恢复字符串:

```java
@Test
public void givenString_thenRevertValue() {
    String text = "Baeldung";
    String transformed = text.transform(value ->
      new StringBuilder(value).reverse().toString()
    );

    assertEquals("gnudleaB", transformed);
}
```

### 2.2.`File::mismatch`方法

Java 12 在[的`nio.file.Files`实用程序类](/web/20220824083948/https://www.baeldung.com/java-nio-2-file-api)中引入了一个新的`mismatch`方法:

```java
public static long mismatch(Path path, Path path2) throws IOException
```

方法用于比较两个文件，并在它们的内容中查找第一个不匹配字节的位置。

返回值将在 0 到较小文件的字节大小之间，如果文件相同，则为-1L。

现在我们来看两个例子。在第一个示例中，我们将创建两个相同的文件，并尝试找出不匹配的地方。返回值应该是-1L:

```java
@Test
public void givenIdenticalFiles_thenShouldNotFindMismatch() {
    Path filePath1 = Files.createTempFile("file1", ".txt");
    Path filePath2 = Files.createTempFile("file2", ".txt");
    Files.writeString(filePath1, "Java 12 Article");
    Files.writeString(filePath2, "Java 12 Article");

    long mismatch = Files.mismatch(filePath1, filePath2);
    assertEquals(-1, mismatch);
}
```

在第二个例子中，我们将创建两个包含“Java 12 文章”和“Java 12 教程”内容的文件。不匹配方法应该返回 8L，因为它是第一个不同的字节:

```java
@Test
public void givenDifferentFiles_thenShouldFindMismatch() {
    Path filePath3 = Files.createTempFile("file3", ".txt");
    Path filePath4 = Files.createTempFile("file4", ".txt");
    Files.writeString(filePath3, "Java 12 Article");
    Files.writeString(filePath4, "Java 12 Tutorial");

    long mismatch = Files.mismatch(filePath3, filePath4);
    assertEquals(8, mismatch);
}
```

### 2.3.发球收集器

Java 12 中引入了一个新的`teeing`收集器，作为对`Collectors`类的补充[:](/web/20220824083948/https://www.baeldung.com/java-8-collectors)

```java
Collector<T, ?, R> teeing(Collector<? super T, ?, R1> downstream1,
  Collector<? super T, ?, R2> downstream2, BiFunction<? super R1, ? super R2, R> merger)
```

它由两个下游收集器组成。每个元素都由两个下游收集器处理。然后将它们的结果传递给合并函数，并转换成最终结果。

tee collector 的示例用法是计算一组数字的平均值。第一个收集器参数将对这些值求和，第二个参数将给出所有数字的计数。合并函数将获取这些结果并计算平均值:

```java
@Test
public void givenSetOfNumbers_thenCalculateAverage() {
    double mean = Stream.of(1, 2, 3, 4, 5)
      .collect(Collectors.teeing(Collectors.summingDouble(i -> i), 
        Collectors.counting(), (sum, count) -> sum / count));
    assertEquals(3.0, mean);
}
```

### 2.4.压缩数字格式

Java 12 附带了一个新的[数字格式化程序](/web/20220824083948/https://www.baeldung.com/java-number-formatting)—`CompactNumberFormat`。它的设计是基于给定地区提供的模式，以较短的形式表示一个数字。

我们可以通过`NumberFormat`类中的`getCompactNumberInstance`方法获得它的实例:

```java
public static NumberFormat getCompactNumberInstance(Locale locale, NumberFormat.Style formatStyle)
```

如前所述，locale 参数负责提供正确的格式模式。格式样式可以是短的或长的。为了更好地理解格式样式，让我们考虑一下美国地区的数字 1000。短样式会将其格式化为“10K”，长样式会将其格式化为“一万”。

现在让我们来看一个例子，这个例子将这篇文章下的赞数压缩成两种不同的风格:

```java
@Test
public void givenNumber_thenCompactValues() {
    NumberFormat likesShort = 
      NumberFormat.getCompactNumberInstance(new Locale("en", "US"), NumberFormat.Style.SHORT);
    likesShort.setMaximumFractionDigits(2);
    assertEquals("2.59K", likesShort.format(2592));

    NumberFormat likesLong = 
      NumberFormat.getCompactNumberInstance(new Locale("en", "US"), NumberFormat.Style.LONG);
    likesLong.setMaximumFractionDigits(2);
    assertEquals("2.59 thousand", likesLong.format(2592));
}
```

## 3.预览更改

一些新功能仅作为预览版提供。要启用它们，我们需要在 IDE 中切换适当的设置，或者明确地告诉编译器使用预览功能:

```java
javac -Xlint:preview --enable-preview -source 12 src/main/java/File.java
```

### 3.1.切换表达式(预览)

Java 12 中引入的最流行的特性是[开关表达式](/web/20220824083948/https://www.baeldung.com/java-switch)。

作为演示，让我们比较一下新旧 switch 语句。我们将使用它们根据来自`LocalDate`实例的`DayOfWeek`枚举来区分工作日和周末。

首先，让我们看看旧的语法:

```java
DayOfWeek dayOfWeek = LocalDate.now().getDayOfWeek();
String typeOfDay = "";
switch (dayOfWeek) {
    case MONDAY:
    case TUESDAY:
    case WEDNESDAY:
    case THURSDAY:
    case FRIDAY:
        typeOfDay = "Working Day";
        break;
    case SATURDAY:
    case SUNDAY:
        typeOfDay = "Day Off";
}
```

现在，让我们看看相同的逻辑开关表达式:

```java
typeOfDay = switch (dayOfWeek) {
    case MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY -> "Working Day";
    case SATURDAY, SUNDAY -> "Day Off";
};
```

新的 switch 语句不仅更加简洁，可读性更好。它们还消除了对 break 语句的需要。第一次匹配后，代码执行不会失败。

另一个显著的区别是，我们可以将 switch 语句直接赋给变量。这在以前是不可能的。

也可以在 switch 表达式中执行代码，而不返回任何值:

```java
switch (dayOfWeek) {
    case MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY -> System.out.println("Working Day");
    case SATURDAY, SUNDAY -> System.out.println("Day Off");
}
```

更复杂的逻辑应该用花括号括起来:

```java
case MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY -> {
    // more logic
    System.out.println("Working Day")
}
```

请注意，我们可以在新旧语法之间进行选择。Java 12 开关表达式只是一个扩展，而不是替代。

### 3.2.`instanceof`的模式匹配(预览)

Java 12 中引入的另一个预览特性是针对`instanceof`的[模式匹配。](/web/20220824083948/https://www.baeldung.com/java-pattern-matching-instanceof)

在以前的 Java 版本中，当使用 if 语句和`[instanceof](/web/20220824083948/https://www.baeldung.com/java-instanceof),`时，我们必须显式地对对象进行类型转换才能访问它的特性:

```java
Object obj = "Hello World!";
if (obj instanceof String) {
    String s = (String) obj;
    int length = s.length();
}
```

使用 Java 12，我们可以直接在语句中声明新的类型转换变量:

```java
if (obj instanceof String s) {
    int length = s.length();
}
```

编译器会自动为我们注入类型化的`String s`变量。

## 4.JVM 变化

Java 12 附带了几个 JVM 增强。在这一部分，我们将快速浏览几个最重要的。

### 4.1.Shenandoah:一个低暂停时间的垃圾收集器

Shenandoah 是一个实验性的[垃圾收集(GC)](/web/20220824083948/https://www.baeldung.com/jvm-garbage-collectors) 算法，目前不包含在默认的 Java 12 版本中。

它通过在运行 Java 线程的同时执行撤离工作来减少 GC 暂停时间。这意味着使用 Shenandoah，暂停时间不依赖于堆的大小，应该是一致的。200 GB 堆或 2 GB 堆的垃圾收集应该具有类似的低暂停行为。

从版本 15 开始，谢南多厄将成为主线 JDK 建造的一部分。

### 4.2.微基准测试套件

Java 12 为 JDK 源代码引入了一套大约 100 个微基准测试。

这些测试将允许在 JVM 上进行连续的性能测试，并且对每个希望在 JVM 上工作或创建新的微基准的开发人员都很有用。

### 4.3.默认 CDS 档案

类数据共享(CDS)特性有助于减少多个 Java 虚拟机之间的启动时间和内存占用。它使用一个构建时生成的默认类列表，该列表包含选定的核心库类。

Java 12 带来的变化是默认情况下启用 CDS 归档。要在 CD 关闭的情况下运行程序，我们需要将 Xshare 标志设置为 off:

```java
java -Xshare:off HelloWorld.java
```

请注意，这可能会延迟程序的启动时间。

## 5.结论

在本文中，我们看到了 Java 12 中实现的大多数新特性。我们还列出了其他一些值得注意的增删。和往常一样，源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220824083948/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-12)

Next **»**[New Features in Java 13](/web/20220824083948/https://www.baeldung.com/java-13-new-features)**«** Previous[New Features in Java 11](/web/20220824083948/https://www.baeldung.com/java-11-new-features)