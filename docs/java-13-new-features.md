# Java 13 中的新特性

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-13-new-features>

[This article is part of a series:](javascript:void(0);)[• New Features in Java 8](/web/20220824083843/https://www.baeldung.com/java-8-new-features)
[• New Features in Java 9](/web/20220824083843/https://www.baeldung.com/new-java-9)
[• New Features in Java 10](/web/20220824083843/https://www.baeldung.com/java-10-overview)
[• New Features in Java 11](/web/20220824083843/https://www.baeldung.com/java-11-new-features)
[• New Features in Java 12](/web/20220824083843/https://www.baeldung.com/java-12-new-features)
• New Features in Java 13 (current article)[• New Features in Java 14](/web/20220824083843/https://www.baeldung.com/java-14-new-features)
[• What's New in Java 15](/web/20220824083843/https://www.baeldung.com/java-15-new)
[• New Features in Java 16](/web/20220824083843/https://www.baeldung.com/java-16-new-features)
[• New Features in Java 17](/web/20220824083843/https://www.baeldung.com/java-17-new-features)

## 1.概观

【2019 年 9 月见证了 JDK 13 的发布，根据 Java 新发布的六个月节奏。在本文中，我们将看看这个版本中引入的新特性和改进。

## 2.预览开发人员功能

Java 13 引入了两个新的语言特性，尽管是在预览模式下。这意味着这些特性已经完全实现，可供开发人员评估，但还没有准备好投入生产。此外，根据反馈，它们可以在未来的版本中被删除或永久保留。

**我们需要指定`–enable-preview`作为命令行标志来使用预览功能**。让我们深入研究一下。

### 2.1.开关表达式(JEP 354)

我们最初在 [JDK 12](/web/20220824083843/https://www.baeldung.com/java-switch#switch-expressions) 中看到开关表情。 [Java 13 的`switch`表达式](https://web.archive.org/web/20220824083843/https://openjdk.java.net/jeps/354)建立在之前版本的基础上，增加了一个新的`yield`语句。

**使用`yield`，我们现在可以有效地从开关表达式**中返回值:

```java
@Test
@SuppressWarnings("preview")
public void whenSwitchingOnOperationSquareMe_thenWillReturnSquare() {
    var me = 4;
    var operation = "squareMe";
    var result = switch (operation) {
        case "doubleMe" -> {
            yield me * 2;
        }
        case "squareMe" -> {
            yield me * me;
        }
        default -> me;
    };

    assertEquals(16, result);
}
```

正如我们所见，现在使用新的`switch.`很容易实现[策略模式](/web/20220824083843/https://www.baeldung.com/java-strategy-pattern)

### 2.2.文本块( [JEP 355](https://web.archive.org/web/20220824083843/https://openjdk.java.net/jeps/355) )

第二个预览功能是[文本块](/web/20220824083843/https://www.baeldung.com/java-text-blocks)用于多行`String`如嵌入式 JSON、XML、HTML 等。

早先，为了在我们的代码中嵌入 JSON，我们将把它声明为一个`String` 文字:

```java
String JSON_STRING 
  = "{\r\n" + "\"name\" : \"Baeldung\",\r\n" + "\"website\" : \"https://www.%s.com/\"\r\n" + "}";
```

现在让我们使用`String` 文本块编写相同的 JSON:

```java
String TEXT_BLOCK_JSON = """
{
    "name" : "Baeldung",
    "website" : "https://www.%s.com/"
}
""";
```

显而易见，不需要转义双引号或添加回车。通过使用文本块，嵌入式 JSON 更容易编写，更容易阅读和维护。

此外，所有`String` 功能都可用:

```java
@Test
public void whenTextBlocks_thenStringOperationsWorkSame() {        
    assertThat(TEXT_BLOCK_JSON.contains("Baeldung")).isTrue();
    assertThat(TEXT_BLOCK_JSON.indexOf("www")).isGreaterThan(0);
    assertThat(TEXT_BLOCK_JSON.length()).isGreaterThan(0);
} 
```

另外，`java.lang.String`现在有三种新的方法来操作文本块:

*   `stripIndent()`–模仿编译器删除附带的空白
*   `translateEscapes()`–将转义序列如`“\\t”`翻译成`“\t”`
*   `formatted()`–与`String::format,` 相同，但用于文本块

让我们快速看一个`String::formatted`例子:

```java
assertThat(TEXT_BLOCK_JSON.formatted("baeldung").contains("www.baeldung.com")).isTrue();
assertThat(String.format(JSON_STRING,"baeldung").contains("www.baeldung.com")).isTrue(); 
```

由于文本块是一个预览功能，可以在未来的版本中删除，这些新方法被标记为不推荐使用。

## 3.动态光盘档案(JEP 350)

类数据共享(CDS)已经成为 Java HotSpot VM 的一个突出特性有一段时间了。它允许在不同的 JVM 之间共享**类元数据，以减少启动时间和内存占用**。JDK 10 通过添加应用程序光盘( [AppCDS](https://web.archive.org/web/20220824083843/https://openjdk.java.net/jeps/310) )扩展了这一能力——让开发者能够在共享档案中包含应用程序类。JDK 12 进一步增强了这一功能，默认情况下包括了[的光盘存档](https://web.archive.org/web/20220824083843/https://openjdk.java.net/jeps/341)。

然而，归档应用程序类的过程非常繁琐。为了生成归档文件，开发人员必须先试运行他们的应用程序来创建一个类列表，然后将它转储到归档文件中。之后，这个档案可以用来在 JVM 之间共享元数据。

借助[动态归档](https://web.archive.org/web/20220824083843/https://openjdk.java.net/jeps/350)，JDK 13 简化了这一过程。现在**我们可以在应用程序退出的时候生成一个共享的归档文件**。这消除了试运行的需要。

为了使应用程序能够在默认的系统归档文件之上创建一个动态共享归档文件，我们需要添加一个选项`-XX:ArchiveClassesAtExit`并将归档文件名称指定为参数:

```java
java -XX:ArchiveClassesAtExit=<archive filename> -cp <app jar> AppName
```

然后，我们可以使用新创建的归档文件运行带有`-XX:SharedArchiveFile`选项的相同应用程序:

```java
java -XX:SharedArchiveFile=<archive filename> -cp <app jar> AppName
```

## 4.ZGC:取消提交未使用的内存(JEP 351)

[Z 垃圾收集器](/web/20220824083843/https://www.baeldung.com/jvm-zgc-garbage-collector)是 Java 11 中引入的一种低延迟垃圾收集机制，因此 GC 暂停时间从未超过 10 毫秒。然而，与 G1 和谢南多厄等其他 HotSpot VM GCs 不同，它不具备将未使用的堆内存返回给操作系统的功能。 [Java 13 给 ZGC 增加了这个功能](https://web.archive.org/web/20220824083843/https://openjdk.java.net/jeps/351)。

现在，随着性能的提高，我们减少了内存占用。

从 Java 13 开始， **ZGC 现在默认将未提交的内存返回给操作系统**，直到达到指定的最小堆大小。如果我们不想使用这个特性，我们可以通过以下方式回到 Java 11 的方式:

*   使用选项`-XX:-ZUncommit,`或
*   设置相等的最小(`-Xms`)和最大(`-Xmx`)堆大小

此外，ZGC 现在支持的最大堆大小为 16TB。早些时候，4TB 是极限。

## 5.重新实现传统套接字 API (JEP 353)

自从 Java 诞生以来，我们已经将 Socket ( `java.net.Socket` 和`java.net.ServerSocket`)API 视为 Java 不可或缺的一部分。然而，在过去的二十年里，它们从未被现代化过。它们是用遗留的 Java 和 C 语言编写的，笨重且难以维护。

Java 13 逆势而上，[取代了底层实现](https://web.archive.org/web/20220824083843/https://openjdk.java.net/jeps/353)，让 API 与未来的用户模式线程保持一致。提供者接口现在指向的是`NioSocketImpl`，而不是`PlainSocketImpl`。这个新编码的实现基于与`java.nio`相同的内部基础设施。

同样，我们有办法回到使用`PlainSocketImpl`的状态。我们可以启动 JVM，将系统属性`-Djdk.net.usePlainSocketImpl`设置为`true`，以使用旧的实现。默认为`NioSocketImpl.`

## 6.杂项变更

除了上面列出的 jep 之外，Java 13 给了我们一些更显著的变化:

*   `java.nio`–增加了方法`FileSystems.newFileSystem(Path, Map<String, ?>)`
*   添加了新的官方日本纪元名称
*   `javax.crypto`–支持 MS 下一代加密技术(CNG)
*   `javax.security`–添加属性`jdk.sasl.disabledMechanisms`以禁用 SASL 机制
*   `javax.xml.crypto`–引入新的`String`常量来表示规范的 XML 1.1 URIs
*   `javax.xml.parsers`——添加了新方法来实例化支持名称空间的 DOM 和 SAX 工厂
*   Unicode 支持升级到 12.1 版
*   增加了对 Kerberos 主体名称规范化和跨领域引用的支持

此外，一些原料药[被提议删除](https://web.archive.org/web/20220824083843/http://cr.openjdk.java.net/~iris/se/13/latestSpec/#APIs-proposed-for-removal)。这些包括上面列出的三个`String` 方法，以及`javax.security.cert` API。

移除的内容包括 JavaDoc 工具中的`rmic` 工具和[旧功能。JDK 1.4 `SocketImpl`之前的实现也不再受支持。](https://web.archive.org/web/20220824083843/https://bugs.java.com/bugdatabase/view_bug.do?bug_id=JDK-8215608)

## 7.结论

在本文中，我们看到了 Java 13 实现的所有五个 JDK 增强提案。我们还列出了其他一些值得注意的增删。

和往常一样，源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220824083843/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-13)

Next **»**[New Features in Java 14](/web/20220824083843/https://www.baeldung.com/java-14-new-features)**«** Previous[New Features in Java 12](/web/20220824083843/https://www.baeldung.com/java-12-new-features)