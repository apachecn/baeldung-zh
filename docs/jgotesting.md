# 使用 JGoTesting 进行测试

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jgotesting>

## 1。概述

[JGoTesting](https://web.archive.org/web/20221128111138/https://gitlab.com/tastapod/jgotesting) 是一个受 Go 的测试包启发的**兼容 JUnit 的测试框架。**

在本文中，我们将探索 JGoTesting 框架的关键特性，并实现示例来展示其功能。

## 2。Maven 依赖关系

首先，让我们将`jgotesting`依赖项添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>org.jgotesting</groupId>
    <artifactId>jgotesting</artifactId>
    <version>0.12</version>
</dependency>
```

这个产品的最新版本可以在[这里](https://web.archive.org/web/20221128111138/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.jgotesting%22%20AND%20a%3A%22jgotesting%22)找到。

## 3。简介

JGoTesting 允许我们编写与 JUnit 兼容的测试。对于 JGoTesting 提供的每个断言方法，JUnit 中都有一个具有相同签名的方法，因此实现这个库非常简单。

然而，**与 JUnit 不同，当断言失败时，JGoTesting 不会停止测试的执行**。相反，失败被记录为一个`event`，并且只有当所有断言都被执行时才呈现给我们。

## 4。JGoTesting in Action

在这一节中，我们将看到如何设置 JGoTesting 的例子，并探索其可能性。

### 4.1。入门

为了编写我们的测试，让我们首先导入 JGoTesting 的断言方法:

```java
import static org.jgotesting.Assert.*; // same methods as JUnit
import static org.jgotesting.Check.*; // aliases starting with "check"
import static org.jgotesting.Testing.*;
```

该库需要一个标有`@Rule`注释的强制`JGoTestRule`实例。这表明该类中的所有测试都将由 JGoTesting 管理。

让我们创建一个类来声明这样的规则:

```java
public class JGoTestingUnitTest {

    @Rule
    public final JGoTestRule test = new JGoTestRule();

    //...
}
```

### 4.2。编写测试

JGoTesting 提供了两组断言方法来编写我们的测试。第一组中的方法名称以`assert`开头，是与 JUnit 兼容的方法，其他的以`check`开头。

两组方法的行为相同，并且库提供了它们之间的一一对应关系。

这里有一个使用两个版本测试一个数字是否等于另一个数字的例子:

```java
@Test
public void whenComparingIntegers_thenEqual() {
    int anInt = 10;

    assertEquals(anInt, 10);
    checkEquals(anInt, 10);
}
```

API 的其余部分是不言自明的，所以我们不会深入讨论细节。对于接下来的所有例子，我们将只关注方法的`check`版本。

### 4.3。故障事件和消息

当检查失败时，JGoTesting 记录失败，以便测试用例继续执行。**测试结束后，报告故障**。

下面的示例展示了这一点:

```java
@Test
public void whenComparingStrings_thenMultipleFailingAssertions() {
    String aString = "The test string";
    String anotherString = "The test String";

    checkEquals("Strings are not equal!", aString, equalTo(anotherString));
    checkTrue("String is longer than one character", aString.length() == 1);
    checkTrue("A failing message", aString.length() == 2);
}
```

执行测试后，我们得到以下输出:

```java
org.junit.ComparisonFailure: Strings are not equal!
  expected:<[the test s]tring> but was:<[The Test S]tring>
// ...
java.lang.AssertionError: String is longer than one character
// ...
java.lang.AssertionError: Strings are not the same
  expected the same:<the test string> was not:<The Test String>
```

除了在每个方法中传递失败消息，我们还可以记录它们，以便它们只在测试至少有一个失败时出现。

让我们编写一个测试方法来实践这一点:

```java
@Test
public void whenComparingNumbers_thenLoggedMessage() {
    log("There was something wrong when comparing numbers");

    int anInt = 10;
    int anotherInt = 10;

    checkEquals(anInt, 10);
    checkTrue("First number should be bigger", 10 > anotherInt);
    checkSame(anInt, anotherInt);
}
```

测试执行后，我们得到以下输出:

```java
org.jgotesting.events.LogMessage: There was something wrong
  when comparing numbers
// ...
java.lang.AssertionError: First number should be bigger
```

注意，除了可以用`String.format()`、*、*方法格式化消息的`logf()`之外，我们还可以使用`logIf()` 和`logUnless()`方法基于条件表达式记录消息。

### 4.4。中断测试

JGoTesting 提供了几种当测试用例不能通过给定的前提条件时终止测试用例的方法。

下面是一个由于所需文件不存在而提前结束的**测试的例子:**

```java
@Test
public void givenFile_whenDoesnotExists_thenTerminated() throws Exception {
    File aFile = new File("a_dummy_file.txt");

    terminateIf(aFile.exists(), is(false));

    // this doesn't get executed
    checkEquals(aFile.getName(), "a_dummy_file.txt");
}
```

注意，我们也可以使用`terminate()`和`terminateUnless()`方法来中断测试执行。

### 4.5。链接

`JGoTestRule`类也有一个**流畅的 API，我们可以用它将检查链接在一起**。

让我们看一个例子，它使用我们的`JGoTestRule`实例将对`String`对象的多个检查链接在一起:

```java
@Test
public void whenComparingStrings_thenMultipleAssertions() {
    String aString = "This is a string";
    String anotherString = "This Is a String";

    test.check(aString, equalToIgnoringCase(anotherString))
      .check(aString.length() == 16)
      .check(aString.startsWith("This"));
}
```

### 4.6。自定义检查

除了`boolean`表达式和`Matcher`实例， **`JGoTestRule`的方法可以接受一个自定义的`Checker`对象来做检查**。这是一个单一的抽象方法接口，可以使用 lambda 表达式实现。

下面是一个使用上述接口验证`String`是否匹配特定正则表达式的例子:

```java
@Test
public void givenChecker_whenComparingStrings_thenEqual() throws Exception {
    Checker<String> aChecker = s -> s.matches("\\d+");

    String aString = "1235";

    test.check(aString, aChecker);
}
```

## 5。结论

在这个快速教程中，我们探索了 JGoTesting 为我们编写测试提供的特性。

我们展示了兼容 JUnit 的`assert`方法以及它们的`check`对应物。我们还看到了这个库如何记录和报告失败事件，并且我们使用 lambda 表达式编写了一个定制的`Checker`。

和往常一样，本文的完整源代码可以在 Github 上找到[。](https://web.archive.org/web/20221128111138/https://github.com/eugenp/tutorials/tree/master/testing-modules/assertion-libraries)