# 用 JUnit 对 System.out.println()进行单元测试

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-testing-system-out-println>

## 1。概述

当单元测试时，我们可能偶尔想要测试我们通过`System.out.println()`写到[标准输出](/web/20221120085317/https://www.baeldung.com/linux/pipes-redirection#standard-io)的消息。

尽管我们通常[更喜欢](/web/20221120085317/https://www.baeldung.com/java-system-out-println-vs-loggers)一个[日志框架](/web/20221120085317/https://www.baeldung.com/java-logging-intro)而不是直接与标准输出交互，但有时这是不可能的。

在这个快速教程中，**我们将看看使用 [JUnit](/web/20221120085317/https://www.baeldung.com/tag/junit/) 进行单元测试`System.out.println()`的几种方法。**

## 2。一种简单的打印方法

在本教程中，我们测试的重点将是一个简单的方法，该方法写入标准输出流:

```java
private void print(String output) {
    System.out.println(output);
} 
```

快速提醒一下，`out`变量是一个`public static final PrintStream`对象，它代表用于系统范围的[标准输出流](/web/20221120085317/https://www.baeldung.com/java-lang-system)。

## 3。使用核心 Java

现在让我们看看如何编写一个单元测试来检查我们发送给`println`方法的内容。然而，在我们编写实际的单元测试之前，我们需要在测试中提供一些初始化:

```java
private final PrintStream standardOut = System.out;
private final ByteArrayOutputStream outputStreamCaptor = new ByteArrayOutputStream();

@BeforeEach
public void setUp() {
    System.setOut(new PrintStream(outputStreamCaptor));
}
```

**在`setUp`方法中，我们用`ByteArrayOutputStream`** 将标准输出流重新分配给新的`PrintStream`。正如我们将要看到的，这个输出流就是现在要打印值的地方:

```java
@Test
void givenSystemOutRedirection_whenInvokePrintln_thenOutputCaptorSuccess() {
    print("Hello Baeldung Readers!!");

    Assert.assertEquals("Hello Baeldung Readers!!", outputStreamCaptor.toString()
      .trim());
}
```

在我们用选择的文本调用了`print`方法之后，我们可以验证`outputStreamCaptor`包含了我们期望的内容。**我们称之为`trim`方法来删除`System.out.println()`添加的新行。**

由于标准输出流是由系统的其他部分使用的共享静态资源，**当我们的测试终止时，我们应该注意将它恢复到原始状态:**

```java
@AfterEach
public void tearDown() {
    System.setOut(standardOut);
}
```

这确保了我们在以后的其他测试中不会有任何不想要的副作用。

## 4。使用系统规则

在本节中，**我们将看看一个整洁的外部库，名为[系统规则](https://web.archive.org/web/20221120085317/https://stefanbirkner.github.io/system-rules/)，它提供了一组 [JUnit 规则](/web/20221120085317/https://www.baeldung.com/junit-4-rules)，用于测试使用`System`类**的代码。

让我们从将[依赖项](https://web.archive.org/web/20221120085317/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22system-rules%22)添加到我们的`pom.xml`开始:

```java
<dependency>
    <groupId>com.github.stefanbirkner</groupId>
    <artifactId>system-rules</artifactId>
    <version>1.19.0</version>
    <scope>test</scope>
</dependency>
```

现在，我们可以使用库提供的`SystemOutRule`编写一个测试:

```java
@Rule
public final SystemOutRule systemOutRule = new SystemOutRule().enableLog();

@Test
public void givenSystemOutRule_whenInvokePrintln_thenLogSuccess() {
    print("Hello Baeldung Readers!!");

    Assert.assertEquals("Hello Baeldung Readers!!", systemOutRule.getLog()
      .trim());
}
```

**相当酷！使用`SystemOutRule,`我们可以拦截对** `**System.out**.` 的写入。首先，我们通过调用规则上的`enableLog`方法开始记录写入`System.out` 的所有内容。然后我们简单地调用`getLog` 来将文本写到`System.out`，因为我们调用了`enableLog`。

这条规则还包括一个简便的方法，该方法返回的日志总是以行分隔符为`\n`

```java
Assert.assertEquals("Hello Baeldung Readers!!\n", systemOutRule.getLogWithNormalizedLineSeparator());
```

## 5。使用 JUnit5 和 Lambdas 的系统规则

在[6 月 5 日](/web/20221120085317/https://www.baeldung.com/junit-5)，规则模型被[扩展](/web/20221120085317/https://www.baeldung.com/junit-5-extensions)所取代。幸运的是，上一节中介绍的系统规则库有一个[变体](https://web.archive.org/web/20221120085317/https://github.com/stefanbirkner/system-lambda)准备用于 JUnit5。

系统 Lambda 可从 [Maven Central](https://web.archive.org/web/20221120085317/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22system-lambda%22) 获得。因此，我们可以继续将它添加到我们的`pom.xml`:

```java
<dependency>
    <groupId>com.github.stefanbirkner</groupId>
    <artifactId>system-lambda</artifactId>
    <version>1.0.0</version>
    <scope>test</scope>
</dependency>
```

现在让我们使用这个版本的库来实现我们的测试:

```java
@Test
void givenTapSystemOut_whenInvokePrintln_thenOutputIsReturnedSuccessfully() throws Exception {

    String text = tapSystemOut(() -> {
        print("Hello Baeldung Readers!!");
    });

    Assert.assertEquals("Hello Baeldung Readers!!", text.trim());
}
```

在这个版本中，**我们使用了`tapSystemOut`方法，该方法执行语句并让我们捕获传递给`System.out`** 的内容。

## 6。结论

在本教程中，我们学习了几种测试`System.out.println`的方法。在第一种方法中，我们看到了如何使用 core Java 重定向编写标准输出流的位置。

然后我们看到了如何使用一个很有前途的外部库 System Rules，首先使用 JUnit 4 风格的规则，然后使用 lambdas。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20221120085317/https://github.com/eugenp/tutorials/tree/master/testing-modules/testing-libraries)