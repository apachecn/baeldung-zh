# Lambda Behave 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/lambda-behave>

## 1。概述

在本文中，我们将讨论一个名为 [Lambda Behave](https://web.archive.org/web/20221206085450/https://github.com/RichardWarburton/lambda-behave) 的新的基于 Java 的测试框架。

顾名思义，这个测试框架旨在与 Java 8 Lambdas 一起工作。此外，在本文中，我们将研究这些规范，并查看每个规范的示例。

我们需要包含的 Maven 依赖项是:

```java
<dependency>           
    <groupId>com.insightfullogic</groupId>
    <artifactId>lambda-behave</artifactId>
    <version>0.4</version>
</dependency> 
```

最新版本可以在这里找到[。](https://web.archive.org/web/20221206085450/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.insightfullogic%22%20AND%20a%3A%22lambda-behave%22)

## 2。基础知识

该框架的目标之一是实现良好的可读性。语法鼓励我们用完整的句子而不是几个单词来描述测试用例。

我们可以利用参数化测试，当我们不想将测试用例绑定到一些预定义的值时，我们可以生成随机参数。

## 3。Lambda 行为测试实现

每个规范套件都以`Suite.describe.`开头。此时，我们有几个内置方法来声明我们的规范。因此，`Suite` 就像一个 JUnit 测试类，规范就像 JUnit 中用`@Test`标注的方法。

为了描述一个测试，我们使用`should().` 类似地，如果我们将期望λ参数命名为`“expect”,`，我们可以通过`expect.that()`说出我们期望从测试中得到什么结果。

如果我们想在一个规范之前或之后设置或删除任何数据，我们可以以同样的方式使用`it.isSetupWith()` 和`it.isConcludedWith().` ，对于在`Suite`之前和之后做的事情，我们将使用`it.initiatizesWith()`和`it.completesWith().`

让我们看一个简单的`Calculator`类测试规范的例子:

```java
public class Calculator {

    public int add() {
        return this.x + this.y;
    }

    public int divide(int a, int b) {
        if (b == 0) {
            throw new ArithmeticException();
        }
        return a / b;
    }
}
```

我们将从`Suite.describe` 开始，然后添加代码来初始化`Calculator.`

接下来，我们将通过编写规范来测试`add()` 方法:

```java
{
    Suite.describe("Lambda behave example tests", it -> {
        it.isSetupWith(() -> {
            calculator = new Calculator(1, 2);
        });

        it.should("Add the given numbers", expect -> {
            expect.that(calculator.add()).is(3);
        });
}
```

这里，为了更好的可读性，我们将变量命名为`“it”`和`“expect”`。因为这些是 lambda 参数名，我们可以用我们选择的任何名称来替换它们。

`should()`的第一个论点用简单的英语描述了这个测试应该检查什么。第二个参数是 lambda，它表明我们期望`add()`方法应该返回 3。

让我们为除以 0 添加另一个测试用例，并验证我们是否得到一个异常:

```java
it.should("Throw an exception if divide by 0", expect -> {
    expect.exception(ArithmeticException.class, () -> {
        calculator.divide(1, 0);
    });
});
```

在这种情况下，我们期待一个异常，所以我们声明`expect.exception()`，并在其中编写应该抛出异常的代码。

**注意，每个规范的文本描述必须是唯一的。**

## 4。数据驱动规范

该框架允许在规范级别进行测试参数化。

为了创建一个例子，让我们给我们的`Calculator`类添加一个方法:

```java
public int add(int a, int b) {
    return a + b;
}
```

让我们为它编写一个数据驱动的测试:

```java
it.uses(2, 3, 5)
  .and(23, 10, 33)
  .toShow("%d + %d = %d", (expect, a, b, c) -> {
    expect.that(calculator.add(a, b)).is(c);
});
```

`uses()`方法用于指定不同列数的输入数据。前两个参数是`add()` 函数参数，第三个是预期结果。这些参数也可用于测试中所示的描述。

`toShow()` 用于描述使用参数的测试–输出如下:

```java
0: 2 + 3 = 5 (seed: 42562700892554)(Lambda behave example tests)
1: 23 + 10 = 33 (seed: 42562700892554)(Lambda behave example tests)
```

## 5。生成的规范-基于属性的测试

通常，当我们编写一个单元测试时，我们希望关注适用于我们系统的更广泛的属性。

例如，当我们测试一个`String`反转函数时，我们可能会检查如果我们反转一个特定的`String`两次，我们会以原来的`String.` 结束

基于属性的测试关注的是通用属性，而不是硬编码的特定测试参数。我们可以通过使用随机生成的测试用例来实现这一点。

这种策略类似于使用数据驱动的规范，但是我们没有指定数据表，而是指定了要生成的测试用例的数量。

因此，我们的`String`基于反转属性的测试应该是这样的:

```java
it.requires(2)
  .example(Generator.asciiStrings())
  .toShow("Reversing a String twice returns the original String", 
    (expect, str) -> {
        String same = new StringBuilder(str)
          .reverse().reverse().toString();
        expect.that(same).isEqualTo(str);
   });
```

我们已经使用`requires()`方法指出了所需测试用例的数量。我们使用`example()`子句来说明我们需要什么类型的对象以及如何需要。

该规范的输出是:

```java
0: Reversing a String twice returns the original String(ljL+qz2) 
  (seed: 42562700892554)(Lambda behave example tests)
1: Reversing a String twice returns the original String(g) 
  (seed: 42562700892554)(Lambda behave example tests)
```

### 5.1。确定性测试用例生成

当我们使用自动生成的测试用例时，隔离测试失败变得相当困难。例如，**如果我们的功能每 1000 次失败一次，那么自动生成 10 个用例的规范将不得不反复运行以观察错误。**

因此，我们需要确定性地重新运行测试的能力，包括以前失败的案例。

Lambda Behave 能够处理这个问题。正如前面测试用例的输出所示，它打印出了用于生成随机测试用例集的种子。因此，如果任何事情失败了，我们可以使用种子来重新创建先前生成的测试用例。

我们可以看看测试用例的输出，识别种子:`(seed: 42562700892554)`。现在，为了再次生成相同的测试集，我们可以使用`SourceGenerator`。

`SourceGenerator`包含`deterministicNumbers()`方法，该方法只将种子作为参数:

```java
 it.requires(2)
   .withSource(SourceGenerator.deterministicNumbers(42562700892554L))
   .example(Generator.asciiStrings())
   .toShow("Reversing a String twice returns the original String", 
     (expect, str) -> {
       String same = new StringBuilder(str).reverse()
         .reverse()
         .toString();
       expect.that(same).isEqualTo(str);
});
```

在运行这个测试时，我们将得到与前面看到的相同的输出。

## 6。结论

在本文中，我们看到了如何在一个名为 Lambda Behave 的新的流畅测试框架中使用 Java 8 lambda 表达式编写单元测试。

和往常一样，这些例子的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221206085450/https://github.com/eugenp/tutorials/tree/master/testing-modules/junit-4)