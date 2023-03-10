# Java 中使用位运算符的位屏蔽

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-bitmasking>

## 1.概观

在本教程中，我们将了解如何使用按位运算符实现低级位屏蔽。我们将看到如何将一个单独的`int`变量作为一个单独数据的容器，类似于[位集](/web/20221128120651/https://www.baeldung.com/java-bitset)。

## 2.位屏蔽

位屏蔽允许我们在一个数值变量中存储多个值。我们不再把这个变量看作一个整数，而是把它的每一位都看作一个独立的值。

因为一位可以等于零或一，我们也可以认为它是假或真。我们也可以将一组位切片，并将其视为一个更小的数字变量，甚至是一个`String`。

### 2.1.例子

假设我们有一个最小的内存占用，并且需要在一个`int`变量中存储一个用户账户的所有信息。前八位(来自 32 个可用位)将存储`boolean`信息，如“账户是否活跃？”或者“账户溢价了吗？”

至于剩余的 24 位，我们将把它们转换成三个字符，作为用户的标识符。

### 2.2.编码

我们的用户将有一个标识符“AAA”，他将有一个活跃和高级帐户(存储在前两位)。在二进制表示中，它看起来像:

```java
String stringRepresentation = "01000001010000010100000100000011";
```

使用内置的`Integer#parseUnsignedInt`方法，这可以很容易地编码成一个`int`变量:

```java
int intRepresentation = Integer.parseUnsignedInt(stringRepresentation, 2);
assertEquals(intRepresentation, 1094795523);
```

### 2.3.解码

这个过程也可以用`Integer#toBinaryString`方法逆转:

```java
String binaryString = Integer.toBinaryString(intRepresentation);
String stringRepresentation = padWithZeros(binaryString);
assertEquals(stringRepresentation, "01000001010000010100000100000011");
```

## 3.提取一位

### 3.1.开眼钎子

如果我们想检查我们的帐户变量的第一位，我们需要的是按位“`and”`操作符和数字“`one` `“`作为位掩码。因为二进制形式的数字“`one`”只有第一位设置为 1，其余的都是 0，**将从变量中删除所有的位，只留下第一位完好无损**:

```java
10000010100000101000001000000011
00000000000000000000000000000001
-------------------------------- &
00000000000000000000000000000001
```

然后我们需要检查产生的值是否不等于零:

```java
intRepresentation & 1 != 0
```

### 3.2.任意位置的位

如果我们想要检查一些其他的位，我们需要**创建一个适当的掩码，它需要在给定的位置有一个位设置为 1，其余的设置为 0**。最简单的方法就是改变我们已经有的面具:

```java
1 << (position - 1)
```

上面的代码行将变量`position`设置为 3，将我们的掩码从:

`00000000000000000000000000000001`
到:

```java
00000000000000000000000000000100
```

所以现在，位等式看起来像这样:

```java
10000010100000101000001000000011
00000000000000000000000000000100
-------------------------------- &
00000000000000000000000000000000
```

将所有这些放在一起，我们可以编写一个在给定位置提取单个位的方法:

```java
private boolean extractValueAtPosition(int intRepresentation, int position) {
    return ((intRepresentation) & (1 << (position - 1))) != 0;
}
```

为了同样的效果，我们也可以反方向移动`intRepresentation`变量，而不是改变掩码。

## 4.提取多个位

我们可以用类似的方法从一个整数中提取多个位。让我们提取用户帐户变量的最后三个字节，并将它们转换成一个字符串。首先，**我们需要通过将变量右移**来去掉前八位:

```java
int lastThreeBites = intRepresentation >> 8;
String stringRepresentation = getStringRepresentation(lastThreeBites);
assertEquals(stringRepresentation, "00000000010000010100000101000001");
```

我们仍然有 32 位，因为`int`将总是有 32 位。然而，现在我们感兴趣的是前 24 位，其余的都是零，很容易被忽略。**我们创建的`int`变量可以很容易地用作一个整数 ID** ，但是因为我们想要一个字符串 ID，我们还有一个步骤要做。

我们将把二进制的字符串表示分成八个字符的组，将它们解析成`char`变量，并将它们连接成一个最终的`String`。

为了方便起见，我们也将忽略空字节:

```java
Arrays.stream(stringRepresentation.split("(?<=\\G.{8})"))
  .filter(eightBits -> !eightBits.equals("00000000"))
  .map(eightBits -> (char)Integer.parseInt(eightBits, 2))
  .collect(StringBuilder::new, StringBuilder::append, StringBuilder::append)
  .toString();
```

## 5.应用位掩码

除了提取和检查单个位的值，我们还可以创建一个掩码来同时检查多个位的值。我们想检查我们的用户是否有一个活跃的高级帐户，所以他的变量的前两位都设置为 1。

我们可以使用以前的方法分别检查它们，但创建一个将它们都选中的遮罩会更快:

```java
int user = Integer.parseUnsignedInt("00000000010000010100000101000001", 2);
int mask = Integer.parseUnsignedInt("00000000000000000000000000000011", 2);
int masked = user & mask;
```

因为我们的用户有一个活动帐户，但它不是高级帐户，所以屏蔽值只有第一位设置为 1:

```java
assertEquals(getStringRepresentation(masked), "00000000000000000000000000000001");
```

现在，我们可以轻松、廉价地断言用户是否满足我们的条件:

```java
assertFalse((user & mask) == mask);
```

## 6.结论

在本教程中，我们学习了如何使用位运算符创建位掩码，并应用它们从整数中提取二进制信息。和往常一样，GitHub 上的所有代码示例[都是可用的。](https://web.archive.org/web/20221128120651/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-operators-2)