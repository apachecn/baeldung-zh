# Java 14 的新特性

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-14-new-features>

[This article is part of a series:](javascript:void(0);)[• New Features in Java 8](/web/20220824092108/https://www.baeldung.com/java-8-new-features)
[• New Features in Java 9](/web/20220824092108/https://www.baeldung.com/new-java-9)
[• New Features in Java 10](/web/20220824092108/https://www.baeldung.com/java-10-overview)
[• New Features in Java 11](/web/20220824092108/https://www.baeldung.com/java-11-new-features)
[• New Features in Java 12](/web/20220824092108/https://www.baeldung.com/java-12-new-features)
[• New Features in Java 13](/web/20220824092108/https://www.baeldung.com/java-13-new-features)
• New Features in Java 14 (current article)[• What's New in Java 15](/web/20220824092108/https://www.baeldung.com/java-15-new)
[• New Features in Java 16](/web/20220824092108/https://www.baeldung.com/java-16-new-features)
[• New Features in Java 17](/web/20220824092108/https://www.baeldung.com/java-17-new-features)

## 1.概观

Java 14 于 2020 年 3 月 17 日发布，根据 Java 的新发布节奏，比它的前一版本整整晚了六个月。

在本教程中，**我们将查看语言**版本 14 的新特性和不推荐的特性。

我们也有关于 Java 14 的更详细的文章[，提供了对新特性的深入了解。](/web/20220824092108/https://www.baeldung.com/tag/java-14/)

## 2.从早期版本延续下来的功能

Java 14 继承了以前版本的一些特性。我们一个一个来看。

### 2.1.开关表情( [JEP 361](https://web.archive.org/web/20220824092108/https://openjdk.java.net/jeps/361) )

这些最初是作为预览功能在 JDK 12 中引入的，即使在 Java 13 中，它们也只是作为预览功能继续存在。但是现在， [**开关表达式**](/web/20220824092108/https://www.baeldung.com/java-13-new-features#switch-expression) **已经标准化，成为开发套件**的重要组成部分。

这实际上意味着这个特性现在可以用在产品代码中，而不仅仅是供开发人员试验的预览模式。

举个简单的例子，让我们考虑这样一个场景，我们将一周中的某几天指定为工作日或周末。

在此增强之前，我们会将其写成:

```
boolean isTodayHoliday;
switch (day) {
    case "MONDAY":
    case "TUESDAY":
    case "WEDNESDAY":
    case "THURSDAY":
    case "FRIDAY":
        isTodayHoliday = false;
        break;
    case "SATURDAY":
    case "SUNDAY":
        isTodayHoliday = true;
        break;
    default:
        throw new IllegalArgumentException("What's a " + day);
}
```

使用开关表达式，我们可以更简洁地编写相同的内容:

```
boolean isTodayHoliday = switch (day) {
    case "MONDAY", "TUESDAY", "WEDNESDAY", "THURSDAY", "FRIDAY" -> false;
    case "SATURDAY", "SUNDAY" -> true;
    default -> throw new IllegalArgumentException("What's a " + day);
};
```

### 2.2.文本块( [JEP 368](https://web.archive.org/web/20220824092108/https://openjdk.java.net/jeps/368) )

文本块继续他们获得主流升级的旅程，并且仍然可以作为预览功能使用。

除了来自 JDK 13 的使多行字符串更容易使用的功能，在他们的第二个预览版中，**文本块现在有两个新的转义序列**:

*   \:指示行尾，这样就不会引入新的行字符
*   \s:表示单个空格

例如:

```
String multiline = "A quick brown fox jumps over a lazy dog; the lazy dog howls loudly.";
```

现在可以写成:

```
String multiline = """
    A quick brown fox jumps over a lazy dog; \
    the lazy dog howls loudly.""";
```

这对人眼来说提高了句子的可读性，但不会在`dog;`后添加新的一行。

## 3.新的预览功能

### 3.1.模式匹配为`instanceof` ( [JEP 305](https://web.archive.org/web/20220824092108/https://openjdk.java.net/jeps/305) )

JDK 14 为 [`instanceof`](/web/20220824092108/https://www.baeldung.com/java-pattern-matching-instanceof) 引入了模式匹配，目的是消除样板代码，让开发人员的生活稍微轻松一些。

为了理解这一点，让我们考虑一个简单的例子。

在这个专题之前，我们写道:

```
if (obj instanceof String) {
    String str = (String) obj;
    int len = str.length();
    // ...
}
```

现在，我们开始使用`obj`时不需要像 String 一样多的代码:

```
if (obj instanceof String str) {
    int len = str.length();
    // ...
}
```

在未来的版本中， **Java 将会为其他结构提供模式匹配，比如`switch`** 。

### 3.2.记录( [JEP 359](https://web.archive.org/web/20220824092108/https://openjdk.java.net/jeps/359) )

[记录](/web/20220824092108/https://www.baeldung.com/java-record-keyword)的引入是为了减少数据模型 POJOs 中重复的样板代码。**它们简化了日常开发，提高了效率，极大地降低了人为错误的风险**。

例如，带有`id`和`password`的`User`的数据模型可以简单地定义为:

```
public record User(int id, String password) { };
```

正如我们所看到的，**我们使用了一个新的关键字，**`**record,**`**。这个简单的声明将自动为我们添加一个构造函数、getters、`equals`、`hashCode`和`toString`方法。**

 **让我们看看 JUnit 的实际应用:

```
private User user1 = new User(0, "UserOne");

@Test
public void givenRecord_whenObjInitialized_thenValuesCanBeFetchedWithGetters() {
    assertEquals(0, user1.id());
    assertEquals("UserOne", user1.password());
}

@Test
public void whenRecord_thenEqualsImplemented() {
    User user2 = user1;
    assertTrue(user1, user2);
}

@Test
public void whenRecord_thenToStringImplemented() {
    assertTrue(user1.toString().contains("UserOne"));
}
```

## 4.新的生产功能

除了这两个新的预览特性，Java 14 还提供了一个具体的产品就绪特性。

### 4.1.乐于助人`NullPointerExceptions` ( [JEP 358](https://web.archive.org/web/20220824092108/https://openjdk.java.net/jeps/358) )

以前，`NullPointerException`的堆栈跟踪没有太多故事可讲，除了在给定文件中的给定行有一些值为空。

尽管这些信息很有用，但它们只是建议调试一行代码，而不是让开发人员通过查看日志来了解整个情况。

**现在 Java 已经[通过增加指出给定代码行](/web/20220824092108/https://www.baeldung.com/java-14-nullpointerexception)**中的`null`确切内容的能力，使这变得更加容易。

例如，考虑这个简单的片段:

```
int[] arr = null;
arr[0] = 1;
```

之前，在运行这段代码时，日志会说:

```
Exception in thread "main" java.lang.NullPointerException
at com.baeldung.MyClass.main(MyClass.java:27)
```

但是现在，给定相同的场景，日志可能会说:

```
java.lang.NullPointerException: Cannot store to int array because "a" is null
```

正如我们所看到的，现在我们确切地知道哪个变量导致了异常。

## 5.孵化特征

这些是 Java 团队提出的非最终 API 和工具，并提供给我们进行实验。它们不同于预览功能，在包`jdk.incubator`中作为单独的模块提供。

### 5.1.外部存储器访问 API ( [JEP 370](https://web.archive.org/web/20220824092108/https://openjdk.java.net/jeps/370) )

这是一个新的 API，允许 Java 程序以安全有效的方式访问堆外的外来内存，比如本地内存。

许多 Java 库，如 [mapDB](/web/20220824092108/https://www.baeldung.com/mapdb) 和 [memcached](https://web.archive.org/web/20220824092108/https://memcached.org/) 确实访问外部内存，现在是 Java API 本身提供一个更干净的解决方案的时候了。本着这一目的，该团队提出了这种 JEP，作为其现有的访问非堆内存方式的替代方法—`ByteBuffer`API 和`sun.misc.Unsafe` API。

这个 API 建立在三个主要的抽象之上，即*内存段*、*内存地址*和*内存布局*，它是访问堆内存和非堆内存的安全方式。

### 5.2.包装工具( [JEP 343](https://web.archive.org/web/20220824092108/https://openjdk.java.net/jeps/343) )

传统上，为了交付 Java 代码，应用程序开发人员会简单地发送一个 JAR 文件，用户应该在他们自己的 JVM 中运行这个文件。

然而，**用户更期待一个安装程序，他们双击它就可以在他们的本地平台上安装软件包**,比如 Windows 或 macOS。

这个 JEP 的目标正是如此。开发人员可以使用`[jlink](/web/20220824092108/https://www.baeldung.com/jlink)`将 JDK 压缩到所需的最少模块，然后使用这个打包工具创建一个轻量级映像，可以作为`exe`安装在 Windows 上，或者作为`dmg`安装在 macOS 上。

## 6.JVM/HotSpot 特性

### 6.1.Windows 上的 ZGC([JEP 365](https://web.archive.org/web/20220824092108/https://openjdk.java.net/jeps/365))和 MAC OS([JEP 364](https://web.archive.org/web/20220824092108/https://openjdk.java.net/jeps/364))——实验

Z 垃圾收集器是一个可伸缩的、低延迟的垃圾收集器，最初是作为一个实验性特性在 Java 11 中引入的。但是最初，唯一支持的平台是 Linux/x64。

在收到了 Linux ZGC 的积极反馈后， **Java 14 也将其支持移植到了 Windows 和 macOS 上**。虽然仍然是一个实验功能，但它已经准备好在[下一个 JDK 版本](https://web.archive.org/web/20220824092108/https://openjdk.java.net/jeps/377)中投入生产。

### 6.2.G1 的 NUMA 感知内存分配( [JEP 345](https://web.archive.org/web/20220824092108/https://openjdk.java.net/jeps/345)

与并行收集器不同，G1 垃圾收集器目前还没有实现非一致内存访问(NUMA)。

考虑到跨多个套接字运行单个 JVM 所带来的性能提升，**引入这个 JEP 是为了让 G1 收集器也能感知 NUMA**。

在这一点上，没有计划复制相同的其他热点收藏家。

### 6.3.JFR 事件流( [JEP 349](https://web.archive.org/web/20220824092108/https://openjdk.java.net/jeps/349) )

通过这一改进，JDK 的飞行记录器数据现已暴露，因此可以对其进行持续监测。这涉及到对包`jdk.jfr.consumer`的修改，以便用户现在可以直接读取或流式传输记录数据。

## 7.已弃用或已移除的功能

Java 14 弃用了几个特性:

*   Solaris 和 SPARC 端口([JEP 362](https://web.archive.org/web/20220824092108/https://openjdk.java.net/jeps/362))——因为这种 Unix 操作系统和 RISC 处理器在过去几年中没有积极开发
*   `ParallelScavenge` + `SerialOld` GC 组合([JEP 366](https://web.archive.org/web/20220824092108/https://openjdk.java.net/jeps/366))–因为这是一种很少使用的 GC 算法组合，并且需要大量的维护工作

也有一些删除:

*   并发标记清除(CMS)垃圾收集器([JEP 363](https://web.archive.org/web/20220824092108/https://openjdk.java.net/jeps/363))——被 Java 9 弃用，这个垃圾收集器被 G1 作为默认垃圾收集器取代。此外，现在有其他更高性能的替代品使用，如 ZGC 和谢南多阿，因此删除
*   Pack200 工具和 API([JEP 367](https://web.archive.org/web/20220824092108/https://openjdk.java.net/jeps/367))–这些在 Java 11 中已被弃用，现在已被移除

## 8.结论

在本教程中，我们看了 Java 14 的各种 jep。

总之，**这个版本的语言**有 16 个主要特性，包括预览特性、孵化器、弃用和移除。我们一个接一个地看了它们，以及它们的语言特性和例子。

和往常一样，源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220824092108/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-14)

Next **»**[What's New in Java 15](/web/20220824092108/https://www.baeldung.com/java-15-new)**«** Previous[New Features in Java 13](/web/20220824092108/https://www.baeldung.com/java-13-new-features)**