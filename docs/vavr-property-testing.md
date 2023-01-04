# Vavr 的性能测试示例

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/vavr-property-testing>

## 1。概述

在本文中，我们将会看到`Property Testing`的概念及其在`vavr-test` 库`.`中的实现

`Property based testing` (PBT)允许我们指定一个程序的高层行为，关于它应该遵守的不变量。

## 2。什么是性能测试？

属性是一个不变量和一个`input values generator`的组合。对于每个生成的值，不变量被视为谓词，并检查它是否为该值生成 true 或 false。

只要有一个值为假，这个属性就被认为是伪造的，检查就会中止。如果某个属性在特定数量的样本数据后不能失效，则该属性被认为是满足的。

由于这种行为，如果不满足条件，我们的测试会很快失败，而无需做不必要的工作。

## 3。Maven 依赖关系

首先，我们需要向 [`vavr-test`](https://web.archive.org/web/20220627182007/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22io.javaslang%22%20AND%20a%3A%22javaslang-test%22) 库中添加一个 Maven 依赖项:

```java
<dependency>
    <groupId>io.vavr</groupId>
    <artifactId>jvavr-test</artifactId>
    <version>${vavr.test.version}</version>
</dependency>

<properties>
    <vavr.test.version>2.0.5</vavr.test.version> 
</properties>
```

## 4。编写基于属性的测试

让我们考虑一个返回字符串流的函数。它是一个无限的 0 以上的流，根据简单的规则将数字映射到字符串。我们在这里使用了一个有趣的 Vavr 特性，称为[模式匹配](/web/20220627182007/https://www.baeldung.com/vavr-pattern-matching):

```java
private static Predicate<Integer> divisibleByTwo = i -> i % 2 == 0;
private static Predicate<Integer> divisibleByFive = i -> i % 5 == 0;

private Stream<String> stringsSupplier() {
    return Stream.from(0).map(i -> Match(i).of(
      Case($(divisibleByFive.and(divisibleByTwo)), "DividedByTwoAndFiveWithoutRemainder"),
      Case($(divisibleByFive), "DividedByFiveWithoutRemainder"),
      Case($(divisibleByTwo), "DividedByTwoWithoutRemainder"),
      Case($(), "")));
}
```

为这种方法编写单元测试很容易出错，因为我们很可能会忘记一些边缘情况，并且基本上不会覆盖所有可能的场景。

幸运的是，我们可以编写一个基于属性的测试来覆盖所有的边缘情况。首先，我们需要定义哪种数字应该作为测试的输入:

```java
Arbitrary<Integer> multiplesOf2 = Arbitrary.integer()
  .filter(i -> i > 0)
  .filter(i -> i % 2 == 0 && i % 5 != 0);
```

我们指定输入数需要满足两个条件——它需要大于零，并且需要能被 2 整除而没有余数，但不能被 5 整除。

接下来，我们需要定义一个条件来检查被测试的函数是否为给定的参数返回正确的值:

```java
CheckedFunction1<Integer, Boolean> mustEquals
  = i -> stringsSupplier().get(i).equals("DividedByTwoWithoutRemainder");
```

要开始一个基于属性的测试，我们需要使用`[Property](https://web.archive.org/web/20220627182007/https://static.javadoc.io/io.javaslang/javaslang-test/2.0.4/javaslang/test/Property.Property5.html)` 类:

```java
CheckResult result = Property
  .def("Every second element must equal to DividedByTwoWithoutRemainder")
  .forAll(multiplesOf2)
  .suchThat(mustEquals)
  .check(10_000, 100);

result.assertIsSatisfied();
```

我们指定，对于所有为 2 的倍数的任意整数，必须满足`mustEquals` 谓词。`check()` 方法获取生成的输入的大小和测试运行的次数。

我们可以快速编写另一个测试来验证`stringsSupplier()` 函数是否为每个输入数字返回一个`DividedByTwoAndFiveWithoutRemainder string` ,这个数字可以被 2 和 5 整除而没有余数。

`Arbitrary`供应商和`CheckedFunction`需要变更:

```java
Arbitrary<Integer> multiplesOf5 = Arbitrary.integer()
  .filter(i -> i > 0)
  .filter(i -> i % 5 == 0 && i % 2 == 0);

CheckedFunction1<Integer, Boolean> mustEquals
  = i -> stringsSupplier().get(i).endsWith("DividedByTwoAndFiveWithoutRemainder");
```

然后我们可以运行基于属性的测试一千次:

```java
Property.def("Every fifth element must equal to DividedByTwoAndFiveWithoutRemainder")
  .forAll(multiplesOf5)
  .suchThat(mustEquals)
  .check(10_000, 1_000)
  .assertIsSatisfied();
```

## 5。结论

在这篇简短的文章中，我们了解了基于属性的测试的概念。

我们使用 vavr `-test` 库创建测试；我们使用了`Arbitrary, CheckedFunction,` 和`Property` 类来定义使用`vavr-test.` 的基于属性的测试

所有这些例子和代码片段的实现都可以在 GitHub 上找到[——这是一个 Maven 项目，所以它应该很容易导入和运行。](https://web.archive.org/web/20220627182007/https://github.com/eugenp/tutorials/tree/master/vavr)