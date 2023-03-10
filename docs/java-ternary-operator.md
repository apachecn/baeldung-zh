# Java 中的三元运算符

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-ternary-operator>

## 1.概观

三元条件运算符 `?:`允许我们在 Java 中定义表达式。**它是也返回值的`if-else`语句的浓缩形式。**

在本教程中，我们将学习何时以及如何使用三元结构。我们将从查看它的语法开始，然后探究它的用法。

## 延伸阅读:

## [Java 中的控制结构](/web/20221030195354/https://www.baeldung.com/java-control-structures)

Learn about the control structures you can use in Java.[Read more](/web/20221030195354/https://www.baeldung.com/java-control-structures) →

## [Java 中的 If-Else 语句](/web/20221030195354/https://www.baeldung.com/java-if-else)

Learn how to use the if-else statement in Java.[Read more](/web/20221030195354/https://www.baeldung.com/java-if-else) →

## [如何在 Java 8 流中使用 if/else 逻辑](/web/20221030195354/https://www.baeldung.com/java-8-streams-if-else-logic)

Learn how to apply if/else logic to Java 8 Streams.[Read more](/web/20221030195354/https://www.baeldung.com/java-8-streams-if-else-logic) →

## 2.句法

Java 中的三元运算符`?:`是唯一接受三个操作数的运算符**:**

```java
booleanExpression ? expression1 : expression2
```

第一个操作数必须是一个`boolean`表达式，第二个和第三个操作数可以是任何返回值的表达式。如果第一个操作数的计算结果为`true`，三元结构将返回`expression1`作为输出，否则返回`expression2`。

## 3.三元运算符示例

让我们考虑这个`if-else`结构:

```java
int num = 8;
String msg = "";
if(num > 10) {
    msg = "Number is greater than 10";
}
else {
    msg = "Number is less than or equal to 10";
}
```

这里，我们根据对`num`的条件评估，为`msg`赋值。

我们可以通过简单地用一个三元结构替换`if-else`语句来使这段代码更加可读和安全:

```java
final String msg = num > 10 
  ? "Number is greater than 10" 
  : "Number is less than or equal to 10";
```

## 4.表达式评估

**使用 Java 三元结构时，在运行时只计算右边表达式中的一个，即`expression1`或`expression2`。**

我们可以通过编写一个简单的`JUnit`测试用例来验证这一点:

```java
@Test
public void whenConditionIsTrue_thenOnlyFirstExpressionIsEvaluated() {
    int exp1 = 0, exp2 = 0;
    int result = 12 > 10 ? ++exp1 : ++exp2;

    assertThat(exp1).isEqualTo(1);
    assertThat(exp2).isEqualTo(0);
    assertThat(result).isEqualTo(1);
}
```

我们的`boolean`表达式 `12 > 10`总是计算为`true`，所以`exp2`的值保持不变。

同样，让我们考虑一下在`false`条件下会发生什么:

```java
@Test
public void whenConditionIsFalse_thenOnlySecondExpressionIsEvaluated() {
    int exp1 = 0, exp2 = 0;
    int result = 8 > 10 ? ++exp1 : ++exp2;

    assertThat(exp1).isEqualTo(0);
    assertThat(exp2).isEqualTo(1);
    assertThat(result).isEqualTo(1);
}
```

`exp1`的值保持不变，而`exp2`的值增加 1。

## 5.嵌套三元运算符

我们可以将三元运算符嵌套到我们选择的任何级别。

所以，这个结构在 Java 中是有效的:

```java
String msg = num > 10 ? "Number is greater than 10" : 
  num > 5 ? "Number is greater than 5" : "Number is less than equal to 5";
```

为了提高上面代码的可读性，我们可以在任何需要的地方使用大括号`()` :

```java
String msg = num > 10 ? "Number is greater than 10" 
  : (num > 5 ? "Number is greater than 5" : "Number is less than equal to 5");
```

然而**，**请注意，不建议在现实世界中使用这种深度嵌套的三元结构。这是因为它降低了代码的可读性和维护难度。

## 6.结论

在这篇简短的文章中，我们学习了 Java 中的三元运算符。不可能用三元运算符替换每个`if-else`结构。但是在某些情况下，它是一个很好的工具，可以使我们的代码更短，可读性更好。

像往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221030195354/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-operators)