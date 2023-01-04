# 检查在 Java 中是否至少三分之二的布尔值为真

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-check-two-of-three-booleans>

## 1.概观

`boolean`是 Java 的[原语](/web/20220815224946/https://www.baeldung.com/java-primitives)之一。这是一个非常简单的数据类型，只有两个值:`true`和`false`。

在本教程中，我们将研究一个问题:检查给定的三个`boolean`中是否至少有两个`true`。

## 2.问题简介

这个问题很简单。我们将被给予三次`booleans`。如果其中至少有两个是`true`，我们的方法应该返回`true`。

解决这个问题对我们来说不是挑战。然而，在本教程中，我们将探索一些好的解决方案。此外，我们将讨论每种方法是否可以很容易地扩展到解决一个一般问题:**给定`n`布尔，检查是否至少其中的`x`是`true`** 。

我们将通过单元测试来验证每种方法。因此，让我们首先创建一个`Map`对象来保存测试用例以及预期的结果:

```java
static final Map<boolean[], Boolean> TEST_CASES_AND_EXPECTED = ImmutableMap.of(
    new boolean[]{true, true, true}, true,
    new boolean[]{true, true, false}, true,
    new boolean[]{true, false, false}, false,
    new boolean[]{false, false, false}, false
); 
```

如上面的代码所示，`TEST_CASES_AND_EXPECTED`图包含四个场景及其预期结果。稍后，我们将遍历这个 map 对象，并将每个`boolean`数组作为参数传递给每个方法，并验证该方法是否返回预期值。

接下来，我们来看看如何解决问题。

## 3.在三个布尔中循环

解决这个问题的最直接的想法可能是**遍历三个给定的布尔值并计算`trues`** 。

一旦计数器大于或等于`2`，我们停止检查并返回`true`。否则，三只`booleans`中的`trues`数量小于 2。于是，我们返回`false`:

```java
public static boolean twoOrMoreAreTrueByLoop(boolean a, boolean b, boolean c) {
    int count = 0;
    for (boolean i : new Boolean[] { a, b, c }) {
        count += i ? 1 : 0;
        if (count >= 2) {
            return true;
        }
    }
    return false;
} 
```

接下来，让我们使用`TEST_CASES_AND_EXPECTED`地图来测试这种方法是否有效:

```java
TEST_CASES_AND_EXPECTED.forEach((array, expected) -> 
  assertThat(ThreeBooleans.twoOrMoreAreTrueByLoop(array[0], array[1], array[2])).isEqualTo(expected));
```

如果我们运行这个测试，不出所料，它通过了。

这种方法非常容易理解。此外，假设我们将方法的参数更改为一个`boolean`数组(或一个`Collection`)和一个`int x`。在这种情况下，**可以很容易地扩展成解决问题的通用解决方案:给定`n booleans,`检查它们中是否至少有`x`是`true`** :

```java
public static boolean xOrMoreAreTrueByLoop(boolean[] booleans, int x) {
    int count = 0;
    for (boolean i : booleans) { 
        count += i ? 1 : 0;
        if (count >= x) {
            return true;
        }
    }
    return false;
} 
```

## 4.将布尔值转换为数字

类似地，**我们可以将三个布尔值转换成数字，计算它们的总和，并检查它是否大于`2`**:

```java
public static boolean twoOrMoreAreTrueBySum(boolean a, boolean b, boolean c) {
    return (a ? 1 : 0) + (b ? 1 : 0) + (c ? 1 : 0) >= 2;
} 
```

让我们执行测试，以确保它按预期工作:

```java
TEST_CASES_AND_EXPECTED.forEach((array, expected) -> 
  assertThat(ThreeBooleans.twoOrMoreAreTrueBySum(array[0], array[1], array[2])).isEqualTo(expected)); 
```

我们也可以将这种方法转化为一种通用的解决方案，至少从`n booleans`开始检查`x trues`:

```java
public static boolean xOrMoreAreTrueBySum(Boolean[] booleans, int x) {
    return Arrays.stream(booleans)
      .mapToInt(b -> Boolean.TRUE.equals(b) ? 1 : 0)
      .sum() >= x;
} 
```

我们已经使用了[流 API](/web/20220815224946/https://www.baeldung.com/java-8-streams) 将每个`boolean`转换成一个`int`，并在上面的代码中计算总和。

## 5.使用逻辑运算符

我们已经通过将布尔值转换成整数解决了这个问题。或者，我们可以使用[逻辑运算](/web/20220815224946/https://www.baeldung.com/java-operators#logical-operators)来确定三个布尔中是否至少有两个是`true.`

我们可以对每两个布尔值执行逻辑 AND ( `&&`)运算。因此，我们将对给定的三个布尔值进行三次 AND 运算。**如果三个布尔中的两个是`true`，那么至少一个逻辑 AND 运算应该产生`true`** :

```java
public static boolean twoOrMoreAreTrueByOpeators(boolean a, boolean b, boolean c) {
    return (a && b) || (a && c) || (b && c);
}
```

接下来，如果我们使用`TEST_CASES_AND_EXPECTED`图测试这个方法，它也通过了:

```java
TEST_CASES_AND_EXPECTED.forEach((array, expected) -> 
  assertThat(ThreeBooleans.twoOrMoreAreTrueByOpeators(array[0], array[1], array[2])).isEqualTo(expected)); 
```

现在，让我们考虑一下是否可以将这种方法推广到一般情况。**只有当`x`为 2 时才有效。同样，如果`n`足够大，我们可能需要构建一个长的逻辑操作链**。

因此，它不适用于一般问题。

## 6.使用卡诺图

**[卡诺图](https://web.archive.org/web/20220815224946/https://en.wikipedia.org/wiki/Karnaugh_map)是一种简化[布尔代数](/web/20220815224946/https://www.baeldung.com/cs/boolean-algebra-basic-laws)表达式**的方法。同样，我们可以从卡诺图中写出表达式。所以，有时候，它可以帮助我们解决复杂的布尔代数问题。

接下来，我们来看看如何用卡诺图解决这个问题。假设我们有三个布尔，A、B 和 C，我们可以构建一个卡诺图:

```java
 | C | !C
------|---|----
 A  B | 1 | 1 
 A !B | 1 | 0
!A !B | 0 | 0
!A  B | 1 | 0
```

在上表中，A、B 和 C 表示它们的`true`值。相反地！一，！b，还有！c 表示它们的`false`值。

因此，正如我们所见，该表涵盖了给定的三种布尔值的所有可能的组合。此外，我们可以找到至少两个布尔为真的所有组合情况。对于这些情况，我们将“1”写入表中。因此，有两个包含 1 的组:第一行(组 1)和第一列(组 2)。

**因此，产生 1 的最终布尔代数表达式将是:`(the expression to obtain all ones in group1 ) || (the expression to obtain all ones in group2)`。**

接下来，我们分而治之。

*   第一组(第一行)-A 和 B 都是正确的。不管 C 的值是多少，我们都会有一个。因此，我们有:`A && B`
*   第二组(第一列)-首先，C 始终为真。而且，A 和 B 中至少要有一个真，因此，我们得到:C && (A || B)

最后，让我们将这两组结合起来，得出解决方案:

```java
public static boolean twoorMoreAreTrueByKarnaughMap(boolean a, boolean b, boolean c) {
    return (c && (a || b)) || (a && b);
} 
```

现在，让我们测试该方法是否如预期的那样工作:

```java
TEST_CASES_AND_EXPECTED.forEach((array, expected) -> 
  assertThat(ThreeBooleans.twoorMoreAreTrueByKarnaughMap(array[0], array[1], array[2])).isEqualTo(expected)); 
```

如果我们执行测试，它就通过了。也就是说，方法完成了工作。

然而，如果我们尝试使用这种方法来解决一般问题，当`n`很大时，生成表格可能是一项困难的工作。

因此，**尽管卡诺图擅长解决复杂的布尔代数问题，但它并不适合一些动态和一般的任务**。

## 7.使用`xor`操作符

最后，让我们看看另一个有趣的方法。

在这个问题中，我们有三个布尔。此外，我们知道一个`boolean`只能有两个不同的值:`true`和`false`。

所以，让我们先从三个布尔中取任意两个，比如说`a`和`b`。然后，我们检查表达式`a != b`的结果:

*   `a != b`为`true`—`a`或`b`为真。所以，如果`c`是真的，那么我们有两个真。否则，我们在三个布尔中有两个 false。也就是说，`c`的值就是答案。
*   `a != b`是`false`–`a`和`b`具有相同的值。因为我们只有三个布尔值，`a`(或 `b`)值就是答案。

因此，我们可以得出解决方案:`a != b ? c : a`。此外，`a != b`检查实际上是一个[异或运算](/web/20220815224946/https://www.baeldung.com/java-xor-operator)。因此，解决方案可以简单到:

```java
public static boolean twoOrMoreAreTrueByXor(boolean a, boolean b, boolean c) {
    return a ^ b ? c : a;
} 
```

当我们使用`TEST_CASES_AND_EXPECTED`图测试该方法时，测试将通过:

```java
TEST_CASES_AND_EXPECTED.forEach((array, expected) -> 
  assertThat(ThreeBooleans.twoOrMoreAreTrueByXor(array[0], array[1], array[2])).isEqualTo(expected)); 
```

这个解决方案非常紧凑和复杂。但是，**我们不能扩展它来解决一般问题**。

## 8.结论

在本文中，我们探索了几种不同的方法来检查三个给定的布尔值中是否至少有两个为真。

此外，我们已经讨论了哪种方法可以很容易地扩展到解决一个更普遍的问题:检查在 *n* 布尔值中是否至少有`x`个 trues。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220815224946/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-operators-2)