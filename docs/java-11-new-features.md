# Java 11 的新特性

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-11-new-features>

[This article is part of a series:](javascript:void(0);)[• New Features in Java 8](/web/20220625225150/https://www.baeldung.com/java-8-new-features)
[• New Features in Java 9](/web/20220625225150/https://www.baeldung.com/new-java-9)
[• New Features in Java 10](/web/20220625225150/https://www.baeldung.com/java-10-overview)
• New Features in Java 11 (current article)[• New Features in Java 12](/web/20220625225150/https://www.baeldung.com/java-12-new-features)
[• New Features in Java 13](/web/20220625225150/https://www.baeldung.com/java-13-new-features)
[• New Features in Java 14](/web/20220625225150/https://www.baeldung.com/java-14-new-features)
[• What's New in Java 15](/web/20220625225150/https://www.baeldung.com/java-15-new)
[• New Features in Java 16](/web/20220625225150/https://www.baeldung.com/java-16-new-features)
[• New Features in Java 17](/web/20220625225150/https://www.baeldung.com/java-17-new-features)

## 1.概观

甲骨文于 2018 年 9 月发布 Java 11，仅比其前身 10 版晚 6 个月。

**Java 11 是继 Java 8 之后的第一个长期支持(LTS)版本。**甲骨文也于 2019 年 1 月停止支持`Java` 8。因此，我们中的许多人将升级到 Java 11。

在本教程中，我们将看看选择 Java 11 JDK 的选项。然后我们将探索 Java 11 中引入的新特性、删除的特性和性能增强。

## 延伸阅读:

## [Java 11 字符串 API 新增功能](/web/20220625225150/https://www.baeldung.com/java-11-string-api)

Learn about additions to the String API in Java 11.[Read more](/web/20220625225150/https://www.baeldung.com/java-11-string-api) →

## [Java 11 Lambda 参数的局部变量语法](/web/20220625225150/https://www.baeldung.com/java-var-lambda-params)

Learn how to use var syntax with lambda expressions in Java 11[Read more](/web/20220625225150/https://www.baeldung.com/java-var-lambda-params) →

## [用 Java 11 否定一个谓词方法引用](/web/20220625225150/https://www.baeldung.com/java-negate-predicate-method-reference)

Learn how to negate a predicate method reference with Java 11.[Read more](/web/20220625225150/https://www.baeldung.com/java-negate-predicate-method-reference) →

## 2.甲骨文与开放的 JDK

Java 10 是我们可以在没有许可的情况下进行商业使用的最后一个免费的 Oracle JDK 版本。从 Java 11 开始，Oracle 不再提供免费的长期支持(LTS)。

**谢天谢地，甲骨文继续提供[开放 JDK](https://web.archive.org/web/20220625225150/https://jdk.java.net/) 版本，我们可以免费下载使用。**

除了甲骨文，我们还可以考虑其他的开放 JDK 提供商。

## 3.开发者功能

让我们看看对通用 API 的更改，以及对开发人员有用的其他一些特性。

### 3.1.新的字符串方法

**Java 11 在`String`类**中增加了几个[新方法](/web/20220625225150/https://www.baeldung.com/java-11-string-api):`isBlank`、`lines`、`strip`、`stripLeading`、`stripTrailing,`和`repeat`。

让我们看看如何利用新的方法从多行字符串中提取非空白的剥离行:

```java
String multilineString = "Baeldung helps \n \n developers \n explore Java.";
List<String> lines = multilineString.lines()
  .filter(line -> !line.isBlank())
  .map(String::strip)
  .collect(Collectors.toList());
assertThat(lines).containsExactly("Baeldung helps", "developers", "explore Java.");
```

这些方法可以减少操作字符串对象所涉及的样板文件的数量，并使我们不必导入库。

在`strip`方法的情况下，它们提供了与更熟悉的`trim`方法相似的功能；但是，有了更好的控制和 Unicode 支持。

### 3.2.新的文件方法

此外，现在从文件中读写`String`更加容易。

**我们可以使用来自*文件*类:**的新的`readString`和`writeString`静态方法

```java
Path filePath = Files.writeString(Files.createTempFile(tempDir, "demo", ".txt"), "Sample text");
String fileContent = Files.readString(filePath);
assertThat(fileContent).isEqualTo("Sample text");
```

### 3.3.集合到数组中

`java.util.Collection`接口包含一个新的默认`toArray`方法，该方法接受一个`IntFunction`参数。

这使得从集合中创建正确类型的数组更加容易:

```java
List sampleList = Arrays.asList("Java", "Kotlin");
String[] sampleArray = sampleList.toArray(String[]::new);
assertThat(sampleArray).containsExactly("Java", "Kotlin");
```

### 3.4.Not 谓词方法

一个静态的 [`not`方法](/web/20220625225150/https://www.baeldung.com/java-negate-predicate-method-reference)被添加到了`Predicate` 接口中。我们可以用它来否定一个现有的谓词，很像`negate` 方法:

```java
List<String> sampleList = Arrays.asList("Java", "\n \n", "Kotlin", " ");
List withoutBlanks = sampleList.stream()
  .filter(Predicate.not(String::isBlank))
  .collect(Collectors.toList());
assertThat(withoutBlanks).containsExactly("Java", "Kotlin");
```

**虽然`not(isBlank)`比`isBlank` `.negate()`读起来更自然，但最大的好处是我们也可以将`not` 和方法引用一起使用，就像`not(String:isBlank)`。**

### 3.5.Lambda 的局部变量语法

Java 11 中增加了对在 lambda 参数中使用[局部变量语法](/web/20220625225150/https://www.baeldung.com/java-var-lambda-params) ( `var`关键字)的支持。

我们可以利用这个特性将修饰符应用于局部变量，比如定义一个类型注释:

```java
List<String> sampleList = Arrays.asList("Java", "Kotlin");
String resultString = sampleList.stream()
  .map((@Nonnull var x) -> x.toUpperCase())
  .collect(Collectors.joining(", "));
assertThat(resultString).isEqualTo("JAVA, KOTLIN");
```

### 3.6. HTTP 客户端

来自`java.net.http`包的[新 HTTP 客户端](/web/20220625225150/https://www.baeldung.com/java-9-http-client)是在 Java 9 中引入的。现在它已经成为 Java 11 的标准特性。

**新的 HTTP API 提高了整体性能，并支持 HTTP/1.1 和 HTTP/2:**

```java
HttpClient httpClient = HttpClient.newBuilder()
  .version(HttpClient.Version.HTTP_2)
  .connectTimeout(Duration.ofSeconds(20))
  .build();
HttpRequest httpRequest = HttpRequest.newBuilder()
  .GET()
  .uri(URI.create("http://localhost:" + port))
  .build();
HttpResponse httpResponse = httpClient.send(httpRequest, HttpResponse.BodyHandlers.ofString());
assertThat(httpResponse.body()).isEqualTo("Hello from the server!");
```

### 3.7.基于嵌套的访问控制

Java 11 在 JVM 中引入了[嵌套](/web/20220625225150/https://www.baeldung.com/java-nest-based-access-control)的概念和相关的访问规则。

Java 中的类嵌套意味着外部/主类及其所有嵌套类:

```java
assertThat(MainClass.class.isNestmateOf(MainClass.NestedClass.class)).isTrue();
```

嵌套类链接到`NestMembers`属性，而外层类链接到`NestHost`属性:

```java
assertThat(MainClass.NestedClass.class.getNestHost()).isEqualTo(MainClass.class);
```

JVM 访问规则允许嵌套成员之间访问私有成员；然而，在以前的 Java 版本中，[反射 API](/web/20220625225150/https://www.baeldung.com/java-reflection) 拒绝相同的访问。

Java 11 解决了这个问题，并提供了使用反射 API 查询新类文件属性的方法:

```java
Set<String> nestedMembers = Arrays.stream(MainClass.NestedClass.class.getNestMembers())
  .map(Class::getName)
  .collect(Collectors.toSet());
assertThat(nestedMembers).contains(MainClass.class.getName(), MainClass.NestedClass.class.getName());
```

### 3.8.运行 Java 文件

这个版本的一个主要变化是**我们不再需要显式地用`javac` 编译 Java 源文件`:`**

```java
$ javac HelloWorld.java
$ java HelloWorld 
Hello Java 8!
```

相反，我们可以使用`java `命令直接运行文件:

```java
$ java HelloWorld.java
Hello Java 11!
```

## 4.性能增强

现在让我们来看看几个主要目的是提高性能的新特性。

### 4.1.动态类文件常量

Java 类文件格式被扩展以支持名为`CONSTANT_Dynamic`的新常量池格式。

加载新的常量池将把创建委托给一个引导方法，就像链接一个`invokedynamic `调用点将链接委托给一个引导方法一样。

此功能增强了性能，面向语言设计人员和编译器实现人员。

### 4.2.改进的 Aarch64 内部函数

Java 11 优化了 ARM64 或 AArch64 处理器上现有的字符串和数组内部函数。此外，为`java.lang.Math`的`sin, cos,` 和`log`方法实现了新的内部函数。

我们像使用其他函数一样使用一个内部函数；然而，编译器以一种特殊的方式处理内部函数。它利用特定于 CPU 架构的汇编代码来提升性能。

### 4.3.一个无操作垃圾收集器

Java 11 中有一个名为 Epsilon 的新垃圾收集器可以作为实验特性使用。

它被称为 No-Op(无操作),因为它分配内存，但实际上并不收集任何垃圾。因此，Epsilon 适用于模拟内存不足错误。

显然，Epsilon 不适合典型的生产 Java 应用程序；但是，在一些特定的使用案例中，它可能是有用的:

*   性能试验
*   记忆压力测试
*   虚拟机接口测试和
*   极其短暂的工作

为了启用它，使用 `-XX:+UnlockExperimentalVMOptions -XX:+UseEpsilonGC`标志。

### 4.4.飞行记录器

**[Java 飞行记录器](/web/20220625225150/https://www.baeldung.com/java-flight-recorder-monitoring) (JFR)现在在开放的 JDK** 开源，而它曾经是甲骨文 JDK 的商业产品。JFR 是一个分析工具，我们可以用它从运行的 Java 应用程序中收集诊断和分析数据。

要开始 120 秒的 JFR 记录，我们可以使用以下参数:

```java
-XX:StartFlightRecording=duration=120s,settings=profile,filename=java-demo-app.jfr
```

我们可以在生产中使用 JFR，因为它的性能开销通常低于 1%。一旦时间过去，我们可以访问保存在 JFR 文件中的记录数据；然而，为了分析和可视化数据，我们需要利用另一个叫做 [JDK 任务控制](/web/20220625225150/https://www.baeldung.com/java-flight-recorder-monitoring) (JMC)的工具。

## 5.已删除和已弃用的模块

随着 Java 的发展，我们不能再使用任何被移除的特性，并且应该停止使用任何被废弃的特性。让我们快速浏览一下最著名的几个。

### 5.1.Java EE 和 CORBA

Java EE 技术的独立版本可以在第三方网站上获得；因此，Java SE 没有必要包含它们。

Java 9 已经弃用了选定的 Java EE 和 CORBA 模块。在版本 11 中，它现已完全删除:

*   基于 XML 的 Web 服务的 Java API`s (java.xml.ws`
*   XML 绑定的 Java 架构 `(java.xml.bind`
*   JavaBeans 激活框架`(java.activation`
*   常见注释 `(java.xml.ws.annotation`
*   通用对象请求代理架构 `(java.corba)`
*   Java 事务 API `(java.transaction`

### 5.2.JMC 和 JavaFX

JDK 任务控制中心 (JMC)不再包括在 JDK 内。JMC 的独立版本现在可以单独下载。

对于 [JavaFX](/web/20220625225150/https://www.baeldung.com/javafx) 模块也是如此；JavaFX 将在 JDK 之外作为一组独立的模块提供。

### 5.3.已弃用的模块

此外，Java 11 弃用了以下模块:

*   Nashorn JavaScript 引擎，包括 JJS 工具
*   JAR 文件的 Pack200 压缩方案

## 6.杂项变更

Java 11 引入了一些更重要的变化:

*   新的 ChaCha20 和 ChaCha20-Poly1305 密码实现取代了不安全的 RC4 流密码
*   支持与 Curve25519 和 Curve448 的加密密钥协议取代了现有的 ECDH 方案
*   将传输层安全性(TLS)升级到 1.3 版带来了安全性和性能方面的改进
*   引入了低延迟垃圾收集器 ZGC，作为具有低暂停时间的实验特性
*   对 Unicode 10 的支持带来了更多的字符、符号和表情符号

## 7.结论

在本文中，我们探索了 Java 11 的一些新特性。

我们讨论了 Oracle 和开放 JDK 之间的区别。我们还回顾了 API 的变化，以及其他有用的开发特性、性能增强，以及删除或废弃的模块。

和往常一样，源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220625225150/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-11-2)

Next **»**[New Features in Java 12](/web/20220625225150/https://www.baeldung.com/java-12-new-features)**«** Previous[New Features in Java 10](/web/20220625225150/https://www.baeldung.com/java-10-overview)