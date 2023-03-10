# Vavr 中的模式匹配指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/vavr-pattern-matching>

## 1。概述

在本文中，我们将重点讨论 Vavr 的模式匹配。如果您不知道 Vavr 是什么，请先阅读`Vavr` `‘s Overview`。

模式匹配是 Java 中固有的特性。人们可以把它看作是一个`switch-case`语句的**高级形式。**

Vavr 的模式匹配的好处是，它让我们不用写一堆`switch` cases 或者`if-then-else`语句。因此，**减少了代码量**，并以人类可读的方式表示条件逻辑。

我们可以通过进行以下导入来使用模式匹配 API:

```java
import static io.vavr.API.*;
```

## 2。模式匹配如何工作

正如我们在上一篇文章中看到的，模式匹配可以用来替换`switch`块:

```java
@Test
public void whenSwitchWorksAsMatcher_thenCorrect() {
    int input = 2;
    String output;
    switch (input) {
    case 0:
        output = "zero";
        break;
    case 1:
        output = "one";
        break;
    case 2:
        output = "two";
        break;
    case 3:
        output = "three";
        break;
    default:
        output = "unknown";
        break;
    }

    assertEquals("two", output);
}
```

或者多个`if`语句:

```java
@Test
public void whenIfWorksAsMatcher_thenCorrect() {
    int input = 3;
    String output;
    if (input == 0) {
        output = "zero";
    }
    if (input == 1) {
        output = "one";
    }
    if (input == 2) {
        output = "two";
    }
    if (input == 3) {
        output = "three";
    } else {
        output = "unknown";
    }

    assertEquals("three", output);
}
```

到目前为止，我们看到的代码片段非常冗长，因此容易出错。当使用模式匹配时，我们使用三个主要的构件:两个静态方法`Match`、`Case`和原子模式。

原子模式表示应该被评估以返回布尔值的条件:

*   `**$()**`:类似于 switch 语句中`default`情况的通配符模式。它处理找不到匹配的情况
*   `**$(value)**`:这是 equals 模式，其中一个值与输入相比就是 equals。
*   `**$(predicate)**`:这是一种条件模式，在这种模式下，谓词函数应用于输入，产生的布尔值用于做出决策。

`switch`和`if`方法可以用一段更短更简洁的代码来代替，如下所示:

```java
@Test
public void whenMatchworks_thenCorrect() {
    int input = 2;
    String output = Match(input).of(
      Case($(1), "one"), 
      Case($(2), "two"), 
      Case($(3), "three"), 
      Case($(), "?"));

    assertEquals("two", output);
}
```

如果输入不匹配，则评估通配符模式:

```java
@Test
public void whenMatchesDefault_thenCorrect() {
    int input = 5;
    String output = Match(input).of(
      Case($(1), "one"), 
      Case($(), "unknown"));

    assertEquals("unknown", output);
}
```

如果没有通配符模式，并且输入不匹配，我们将得到一个匹配错误:

```java
@Test(expected = MatchError.class)
public void givenNoMatchAndNoDefault_whenThrows_thenCorrect() {
    int input = 5;
    Match(input).of(
      Case($(1), "one"), 
      Case($(2), "two"));
}
```

在本节中，我们已经介绍了 Vavr 模式匹配的基础知识，接下来的几节将介绍处理代码中可能遇到的不同情况的各种方法。

## 3。与选项匹配

正如我们在上一节中看到的，通配符模式`$()`匹配缺省情况，即没有为输入找到匹配。

然而，包含通配符模式的另一种替代方法是将匹配操作的返回值包装在一个`Option`实例中:

```java
@Test
public void whenMatchWorksWithOption_thenCorrect() {
    int i = 10;
    Option<String> s = Match(i)
      .option(Case($(0), "zero"));

    assertTrue(s.isEmpty());
    assertEquals("None", s.toString());
}
```

想更好的了解 Vavr 中的`Option`，可以参考入门文章。

## 4。匹配内置谓词

Vavr 附带了一些内置的谓词，使我们的代码更易于阅读。因此，我们最初的例子可以用谓词进一步改进:

```java
@Test
public void whenMatchWorksWithPredicate_thenCorrect() {
    int i = 3;
    String s = Match(i).of(
      Case($(is(1)), "one"), 
      Case($(is(2)), "two"), 
      Case($(is(3)), "three"),
      Case($(), "?"));

    assertEquals("three", s);
}
```

Vavr 提供了比这更多的谓词。例如，我们可以让我们的条件检查输入的类别:

```java
@Test
public void givenInput_whenMatchesClass_thenCorrect() {
    Object obj=5;
    String s = Match(obj).of(
      Case($(instanceOf(String.class)), "string matched"), 
      Case($(), "not string"));

    assertEquals("not string", s);
}
```

或者输入是否为`null`:

```java
@Test
public void givenInput_whenMatchesNull_thenCorrect() {
    Object obj=5;
    String s = Match(obj).of(
      Case($(isNull()), "no value"), 
      Case($(isNotNull()), "value found"));

    assertEquals("value found", s);
}
```

我们可以使用`contains`样式，而不是用`equals`样式匹配值。这样，我们可以用`isIn`谓词检查输入是否存在于值列表中:

```java
@Test
public void givenInput_whenContainsWorks_thenCorrect() {
    int i = 5;
    String s = Match(i).of(
      Case($(isIn(2, 4, 6, 8)), "Even Single Digit"), 
      Case($(isIn(1, 3, 5, 7, 9)), "Odd Single Digit"), 
      Case($(), "Out of range"));

    assertEquals("Odd Single Digit", s);
}
```

我们可以对谓词做更多的事情，比如将多个谓词组合成一个匹配用例。为了只在输入通过一组给定谓词时进行匹配，我们可以使用`allOf`谓词来`AND`谓词。

一个实际的例子是，我们想要检查一个数字是否包含在一个列表中，就像我们在前面的例子中所做的那样。问题是这个列表也包含空值。因此，我们希望应用一个过滤器，除了拒绝不在列表中的数字，还将拒绝空值:

```java
@Test
public void givenInput_whenMatchAllWorks_thenCorrect() {
    Integer i = null;
    String s = Match(i).of(
      Case($(allOf(isNotNull(),isIn(1,2,3,null))), "Number found"), 
      Case($(), "Not found"));

    assertEquals("Not found", s);
}
```

为了在输入匹配任何给定组时进行匹配，我们可以使用`anyOf`谓词对谓词进行 OR 运算。

假设我们按出生年份筛选候选人，我们只想要 1990 年、1991 年或 1992 年出生的候选人。

如果找不到这样的候选人，那么我们只能接受 1986 年出生的人，我们也希望在我们的准则中明确这一点:

```java
@Test
public void givenInput_whenMatchesAnyOfWorks_thenCorrect() {
    Integer year = 1990;
    String s = Match(year).of(
      Case($(anyOf(isIn(1990, 1991, 1992), is(1986))), "Age match"), 
      Case($(), "No age match"));
    assertEquals("Age match", s);
}
```

最后，我们可以使用`noneOf`方法确保所提供的谓词都不匹配。

为了证明这一点，我们可以否定上例中的条件，从而得到不在上述年龄组中的候选人:

```java
@Test
public void givenInput_whenMatchesNoneOfWorks_thenCorrect() {
    Integer year = 1990;
    String s = Match(year).of(
      Case($(noneOf(isIn(1990, 1991, 1992), is(1986))), "Age match"), 
      Case($(), "No age match"));

    assertEquals("No age match", s);
}
```

## 5.与自定义谓词匹配

在上一节中，我们探讨了 Vavr 的内置谓词。但是 Vavr 并不止于此。有了 lambdas 的知识，我们可以构建和使用我们自己的谓词，甚至只需内联编写它们。

有了这些新知识，我们可以在前一节的第一个示例中嵌入一个谓词，并将其重写为:

```java
@Test
public void whenMatchWorksWithCustomPredicate_thenCorrect() {
    int i = 3;
    String s = Match(i).of(
      Case($(n -> n == 1), "one"), 
      Case($(n -> n == 2), "two"), 
      Case($(n -> n == 3), "three"), 
      Case($(), "?"));
    assertEquals("three", s);
}
```

如果我们需要更多的参数，我们也可以用一个函数接口来代替谓词。contains 示例可以像这样重写，虽然有点冗长，但是它给了我们更多的权力来控制我们的谓词所做的事情:

```java
@Test
public void givenInput_whenContainsWorks_thenCorrect2() {
    int i = 5;
    BiFunction<Integer, List<Integer>, Boolean> contains 
      = (t, u) -> u.contains(t);

    String s = Match(i).of(
      Case($(o -> contains
        .apply(i, Arrays.asList(2, 4, 6, 8))), "Even Single Digit"), 
      Case($(o -> contains
        .apply(i, Arrays.asList(1, 3, 5, 7, 9))), "Odd Single Digit"), 
      Case($(), "Out of range"));

    assertEquals("Odd Single Digit", s);
}
```

在上面的例子中，我们创建了一个 Java 8 `BiFunction`，它只是检查两个参数之间的`isIn`关系。

你也可以用 Vavr 的`FunctionN`来做这个。因此，如果内置谓词不完全符合您的需求，或者您想要控制整个评估，那么就使用定制谓词。

## 6。对象分解

对象分解是将 Java 对象分解成其组成部分的过程。例如，考虑提取雇员的生物数据和雇佣信息的情况:

```java
public class Employee {

    private String name;
    private String id;

    //standard constructor, getters and setters
}
```

我们可以将一个雇员的记录分解成它的组成部分:`name`和`id`。这在 Java 中非常明显:

```java
@Test
public void givenObject_whenDecomposesJavaWay_thenCorrect() {
    Employee person = new Employee("Carl", "EMP01");

    String result = "not found";
    if (person != null && "Carl".equals(person.getName())) {
        String id = person.getId();
        result="Carl has employee id "+id;
    }

    assertEquals("Carl has employee id EMP01", result);
}
```

我们创建一个 employee 对象，然后在应用过滤器之前首先检查它是否为 null，以确保我们最终得到一个名为`Carl`的雇员的记录。然后我们继续检索他的`id`。Java 方式是可行的，但它冗长且容易出错。

在上面的例子中，我们所做的基本上是将我们所知道的与即将到来的相匹配。我们知道我们需要一个名为`Carl`的雇员，所以我们尝试将这个名字与传入的对象相匹配。

然后我们分解他的细节，得到一个人类可读的输出。空支票只是我们不需要的防御开销。

使用 Vavr 的模式匹配 API，我们可以忘记不必要的检查，只关注重要的内容，从而产生非常紧凑和可读的代码。

要使用这个条款，我们必须在您的项目中安装一个额外的`vavr-match`依赖项。你可以通过点击[这个链接](https://web.archive.org/web/20220122023440/https://search.maven.org/classic/#search%7Cga%7C1%7Cvavr-match)得到它。

上述代码可以写成如下形式:

```java
@Test
public void givenObject_whenDecomposesVavrWay_thenCorrect() {
    Employee person = new Employee("Carl", "EMP01");

    String result = Match(person).of(
      Case(Employee($("Carl"), $()),
        (name, id) -> "Carl has employee id "+id),
      Case($(),
        () -> "not found"));

    assertEquals("Carl has employee id EMP01", result);
}
```

上例中的关键结构是原子模式`$(“Carl”)`和`$()`，值模式和通配符模式分别是。我们在 [Vavr 介绍文章](/web/20220122023440/https://www.baeldung.com/vavr)中详细讨论了这些。

这两种模式都从匹配的对象中检索值，并将它们存储到 lambda 参数中。值模式`$(“Carl”)`只有在检索到的值与其中的内容匹配时才能匹配，即`carl`。

另一方面，**通配符模式`$()`匹配其位置上的任何值**，并将该值检索到`id` lambda 参数中。

为了让这种分解工作，我们需要定义分解模式，或者正式称为`unapply`模式的东西。

这意味着我们必须教会模式匹配 API 如何分解我们的对象，从而为每个要分解的对象生成一个条目:

```java
@Patterns
class Demo {
    @Unapply
    static Tuple2<String, String> Employee(Employee Employee) {
        return Tuple.of(Employee.getName(), Employee.getId());
    }

    // other unapply patterns
}
```

注释处理工具将生成一个名为`DemoPatterns.java`的类，我们必须将它静态导入到我们想要应用这些模式的任何地方:

```java
import static com.baeldung.vavr.DemoPatterns.*;
```

我们也可以分解内置的 Java 对象。

例如，`java.time.LocalDate`可以分解为年、月和月中的日。让我们把它的`unapply`图案加到`Demo.java`上:

```java
@Unapply
static Tuple3<Integer, Integer, Integer> LocalDate(LocalDate date) {
    return Tuple.of(
      date.getYear(), date.getMonthValue(), date.getDayOfMonth());
}
```

然后是测试:

```java
@Test
public void givenObject_whenDecomposesVavrWay_thenCorrect2() {
    LocalDate date = LocalDate.of(2017, 2, 13);

    String result = Match(date).of(
      Case(LocalDate($(2016), $(3), $(13)), 
        () -> "2016-02-13"),
      Case(LocalDate($(2016), $(), $()),
        (y, m, d) -> "month " + m + " in 2016"),
      Case(LocalDate($(), $(), $()),  
        (y, m, d) -> "month " + m + " in " + y),
      Case($(), 
        () -> "(catch all)")
    );

    assertEquals("month 2 in 2017",result);
}
```

## 7 .**。模式匹配中的副作用**

默认情况下，`Match`的行为类似于一个表达式，这意味着它返回一个结果。然而，我们可以通过在 lambda 中使用辅助函数`run`来迫使它产生副作用。

它采用一个方法引用或 lambda 表达式并返回`Void.`

**考虑一个场景**,当输入是一位偶数时，我们想打印一些内容，当输入是一位奇数时，我们想打印另一个内容，当输入不是这些内容时，我们想抛出一个异常。

偶数打印机:

```java
public void displayEven() {
    System.out.println("Input is even");
}
```

奇数打印机:

```java
public void displayOdd() {
    System.out.println("Input is odd");
}
```

和匹配功能:

```java
@Test
public void whenMatchCreatesSideEffects_thenCorrect() {
    int i = 4;
    Match(i).of(
      Case($(isIn(2, 4, 6, 8)), o -> run(this::displayEven)), 
      Case($(isIn(1, 3, 5, 7, 9)), o -> run(this::displayOdd)), 
      Case($(), o -> run(() -> {
          throw new IllegalArgumentException(String.valueOf(i));
      })));
}
```

它将打印:

```java
Input is even
```

## 8。结论

在本文中，我们探讨了 Vavr 中模式匹配 API 最重要的部分。事实上，多亏了 Vavr，我们现在可以编写更简单、更简洁的代码，而无需冗长的开关和 if 语句。

要获得本文的完整源代码，您可以查看 Github 项目。