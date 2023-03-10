# Java 中的阿姆斯特朗数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-armstrong-numbers>

## 1.概观

在这个快速教程中，我们将学习什么是阿姆斯特朗数，以及如何通过创建一个 Java 程序来检查和找到它们。

## 2.问题简介

首先，让我们了解什么是阿姆斯特朗数。

**给定一个正整数`n`位的`i`，如果整数`i`位的`n-th`次幂之和等于`i.`** 阿姆斯特朗数形成 [OEIS 序列 A005188](https://web.archive.org/web/20221018120236/https://oeis.org/A005188#:~:text=A005188%20%2D%20OEIS&text=(Greetings%20from%20The%20On%2DLine%20Encyclopedia%20of%20Integer%20Sequences!)&text=Armstrong%20(or%20pluperfect%2C%20or%20Plus,th%20powers%20of%20their%20digits.&text=A%20finite%20sequence%2C%20the%2088th%20and%20last%20term%20being%20115132219018763992565095597973971522401.) ，则该整数为阿姆斯特朗数。

几个例子可以帮助我们快速理解阿姆斯特朗数:

*   `1` : `pow(1,1) = 1` - > 1 是阿姆斯壮的号码。
*   `123` : `pow(1, 3) + pow(2, 3) + pow(3, 3) = 1 + 8 +27 = 36 != 123` - > 123 不是阿姆斯特朗的号码。
*   `1634` : `pow(1, 4) + pow(6, 4) + pow(3, 4) + pow(4, 4) = 1 + 1296 + 81 + 256 = 1643` - > 1634 是阿姆斯壮的号码。

因此，我们希望有一个 Java 程序来方便地检查一个给定的数字是否是阿姆斯特朗数。此外，我们希望产生一个小于给定极限的 OEIS 序列 A005188。

为了简单起见，我们将使用单元测试断言来验证我们的方法是否按预期工作。

## 3.解决问题的想法

现在，我们已经了解了阿姆斯特朗的数字，让我们来看看这个问题，并考虑解决它的想法。

首先，**生成一个有极限的 OEIS 序列 A005188 可以转化为从 0 到给定的极限，找出所有的 Armstrong 数。**如果我们有一种方法来检查一个整数是否是阿姆斯特朗数，那么很容易从整数范围中过滤出非阿姆斯特朗数，并得到想要的序列。

因此，首要问题是创建 Armstrong 数检查方法。一个简单的检查方法是两步走:

*   步骤 1–将给定的整数分解成一个数字列表，例如`12345 -> [1, 2, 3, 4, 5]`
*   步骤 2–对于列表中的每个数字，计算`pow(digit, list.size())`，然后对结果求和，最后将总和与最初给定的整数进行比较

接下来，让我们将这个想法转换成 Java 代码。

## 4.创建阿姆斯特朗数方法

正如我们已经讨论过的，让我们首先把给定的整数转换成一个数字列表:

```java
static List<Integer> digitsInList(int n) {
    List<Integer> list = new ArrayList<>();
    while (n > 0) {
        list.add(n % 10);
        n = n / 10;
    }
    return list;
} 
```

如上面的代码所示，我们在一个 [`while`](/web/20221018120236/https://www.baeldung.com/java-while-loop) 循环中从`n`中提取数字。**在每一步中，我们通过`n % 10`取一个数字，然后通过`n = n / 10`** 将数字缩小。

或者，我们可以将数字转换成一个字符串，并使用`[split()](/web/20221018120236/https://www.baeldung.com/string/split)`方法获得一个字符串中的数字列表。最后，我们可以将每个数字再次转换回整数。在这里，我们没有采取这种方法。

现在我们已经创建了检查方法，我们可以转到步骤 2: `pow()`计算和求和:

```java
static boolean isArmstrong(int n) {
    if (n < 0) {
        return false;
    }
    List<Integer> digitsList = digitsInList(n);
    int len = digitsList.size();
    int sum = digitsList.stream()
      .mapToInt(d -> (int) Math.pow(d, len))
      .sum();
    return n == sum;
} 
```

正如我们在`isArmstrong() ` check 方法中看到的，我们使用了 Java `[Stream](/web/20221018120236/https://www.baeldung.com/java-8-streams)`的`mapToInt()`方法将每个数字转换成`pow()`计算后的结果，然后用[对列表中的结果求和](/web/20221018120236/https://www.baeldung.com/java-stream-sum#using-intstreamsum)。

最后，我们将总和与初始整数进行比较，以确定该数是否是阿姆斯特朗数。

值得一提的是**我们也可以将`mapToInt()` 和`sum()`方法调用合并成一个 [`reduce()`](/web/20221018120236/https://www.baeldung.com/java-stream-reduce) 调用**:

```java
int sum = digits.stream()
  .reduce(0, (subtotal, digit) -> subtotal + (int) Math.pow(digit, len));
```

接下来，让我们创建一个方法来生成 OEIS 序列 A005188，直到一个极限:

```java
static List<Integer> getA005188Sequence(int limit) {
    if (limit < 0) {
        throw new IllegalArgumentException("The limit cannot be a negative number.");
    }
    return IntStream.range(0, limit)
      .boxed()
      .filter(ArmstrongNumberUtil::isArmstrong)
      .collect(Collectors.toList());
} 
```

正如我们在上面的代码中看到的，我们再次使用了 Stream API 来过滤 Armstrong 数字并生成序列。

## 5.测试

现在，让我们创建一些测试来验证我们的方法是否如预期的那样工作。首先，让我们从一些测试数据开始:

```java
static final Map<Integer, Boolean> ARMSTRONG_MAP = ImmutableMap.of(
  0, true,
  1, true,
  2, true,
  153, true,
  370, true,
  407, true,
  42, false,
  777, false,
  12345, false); 
```

现在，让我们将上面的`Map`中的每个数字传递给我们的检查方法，看看是否返回预期的结果:

```java
ARMSTRONG_MAP.forEach((number, result) -> assertEquals(result, ArmstrongNumberUtil.isArmstrong(number))); 
```

如果我们进行测试，就会通过。因此，检查方法正确地完成了工作。

接下来，让我们准备两个预期的序列，并测试`getA005188Sequence()`是否也像预期的那样工作:

```java
List<Integer> A005188_SEQ_1K = ImmutableList.of(0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 153, 370, 371, 407);
List<Integer> A005188_SEQ_10K = ImmutableList.of(0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 153, 370, 371, 407, 1634, 8208, 9474);

assertEquals(A005188_SEQ_1K, ArmstrongNumberUtil.getA005188Sequence(1000));
assertEquals(A005188_SEQ_10K, ArmstrongNumberUtil.getA005188Sequence(10000)); 
```

如果我们试一试，我们的测试就会通过。

## 6.结论

在本文中，我们已经讨论了什么是阿姆斯特朗数。此外，我们还创建了一些方法来检查一个整数是否是 Armstrong 数，并生成 OEIS 序列 A005188，直到达到给定的限制。

像往常一样，这里展示的所有代码片段都可以在 GitHub 上获得。