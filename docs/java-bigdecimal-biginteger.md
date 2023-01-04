# Java 中的 BigDecimal 和 BigInteger

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-bigdecimal-biginteger>

## 1.概观

在本教程中，我们将演示 [`BigDecimal`](https://web.archive.org/web/20221130044147/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html) 和 [`BigInteger`](https://web.archive.org/web/20221130044147/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigInteger.html) 类。

我们将描述这两种数据类型、它们的特征以及它们的使用场景。我们还将简要介绍使用这两个类的各种操作。

## 2.`BigDecimal`

**`BigDecimal`表示不可变的任意精度有符号十进制数**。它由两部分组成:

*   未缩放的值–任意精度的整数
*   scale–一个 32 位整数，表示小数点右边的位数

例如，`BigDecimal` 3.14 的无标度值为 314，标度为 2。

**我们使用`BigDecimal`进行高精度运算。我们也用它来计算需要控制的规模和四舍五入行为**。一个这样的例子是涉及金融交易的计算。

**我们可以从`String`、字符数组、`int`、`long`、`BigInteger`、T6 中创建一个`BigDecimal`对象:**

```java
@Test
public void whenBigDecimalCreated_thenValueMatches() {
    BigDecimal bdFromString = new BigDecimal("0.1");
    BigDecimal bdFromCharArray = new BigDecimal(new char[] {'3','.','1','6','1','5'});
    BigDecimal bdlFromInt = new BigDecimal(42);
    BigDecimal bdFromLong = new BigDecimal(123412345678901L);
    BigInteger bigInteger = BigInteger.probablePrime(100, new Random());
    BigDecimal bdFromBigInteger = new BigDecimal(bigInteger);

    assertEquals("0.1",bdFromString.toString());
    assertEquals("3.1615",bdFromCharArray.toString());
    assertEquals("42",bdlFromInt.toString());
    assertEquals("123412345678901",bdFromLong.toString());
    assertEquals(bigInteger.toString(),bdFromBigInteger.toString());
}
```

我们也可以从`double`创建`BigDecimal`:

```java
@Test
public void whenBigDecimalCreatedFromDouble_thenValueMayNotMatch() {
    BigDecimal bdFromDouble = new BigDecimal(0.1d);
    assertNotEquals("0.1", bdFromDouble.toString());
}
```

但是，在这种情况下，结果与预期不同(即 0.1)。这是因为:

*   `double`构造函数进行精确的翻译
*   0.1 在`double`中没有精确的表示

因此，**我们应该使用 S `tring`构造函数而不是`double`构造函数**。

此外，我们可以使用`valueOf`静态方法将`double`和`long`转换为`BigDecimal`:

```java
@Test
public void whenBigDecimalCreatedUsingValueOf_thenValueMatches() {
    BigDecimal bdFromLong1 = BigDecimal.valueOf(123412345678901L);
    BigDecimal bdFromLong2 = BigDecimal.valueOf(123412345678901L, 2);
    BigDecimal bdFromDouble = BigDecimal.valueOf(0.1d);

    assertEquals("123412345678901", bdFromLong1.toString());
    assertEquals("1234123456789.01", bdFromLong2.toString());
    assertEquals("0.1", bdFromDouble.toString());
}
```

该方法在将`double`转换成`BigDecimal`之前，先将`double`转换成它的`String`表示。此外，它可以重用对象实例。

因此，**我们应该优先使用`valueOf`方法，而不是构造函数**。

## 3.`BigDecimal`上的操作

就像其他的`Number`类(`Integer`、`Long`、`Double`等)。)，`BigDecimal`提供了用于算术运算和比较运算的操作。它还提供缩放操作、舍入和格式转换操作。

它不会重载算术(+、-、/、*)或逻辑(>。< etc)运算符。而是用相应的方法——`add`、`subtract`、`multiply`、`divide`、`compareTo.`

**`BigDecimal`有提取各种属性的方法，比如精度、小数位数、符号**:

```java
@Test
public void whenGettingAttributes_thenExpectedResult() {
    BigDecimal bd = new BigDecimal("-12345.6789");

    assertEquals(9, bd.precision());
    assertEquals(4, bd.scale());
    assertEquals(-1, bd.signum());
}
```

**我们使用`compareTo`方法**比较两位大小数的值:

```java
@Test
public void whenComparingBigDecimals_thenExpectedResult() {
    BigDecimal bd1 = new BigDecimal("1.0");
    BigDecimal bd2 = new BigDecimal("1.00");
    BigDecimal bd3 = new BigDecimal("2.0");

    assertTrue(bd1.compareTo(bd3) < 0);
    assertTrue(bd3.compareTo(bd1) > 0);
    assertTrue(bd1.compareTo(bd2) == 0);
    assertTrue(bd1.compareTo(bd3) <= 0);
    assertTrue(bd1.compareTo(bd2) >= 0);
    assertTrue(bd1.compareTo(bd3) != 0);
}
```

此方法在比较时忽略比例。

另一方面，**`equals`方法认为只有当两个`BigDecimal`对象的值和比例**相等时，它们才相等。因此，用这种方法比较时，`BigDecimals` 1.0 和 1.00 是不相等的。

```java
@Test
public void whenEqualsCalled_thenSizeAndScaleMatched() {
    BigDecimal bd1 = new BigDecimal("1.0");
    BigDecimal bd2 = new BigDecimal("1.00");

    assertFalse(bd1.equals(bd2));
}
```

**我们通过调用相应的方法**来执行算术运算:

```java
@Test
public void whenPerformingArithmetic_thenExpectedResult() {
    BigDecimal bd1 = new BigDecimal("4.0");
    BigDecimal bd2 = new BigDecimal("2.0");

    BigDecimal sum = bd1.add(bd2);
    BigDecimal difference = bd1.subtract(bd2);
    BigDecimal quotient = bd1.divide(bd2);
    BigDecimal product = bd1.multiply(bd2);

    assertTrue(sum.compareTo(new BigDecimal("6.0")) == 0);
    assertTrue(difference.compareTo(new BigDecimal("2.0")) == 0);
    assertTrue(quotient.compareTo(new BigDecimal("2.0")) == 0);
    assertTrue(product.compareTo(new BigDecimal("8.0")) == 0);
}
```

**由于`BigDecimal`是不可变的，这些操作不会修改现有的对象。**相反，它们返回新的对象。

## 4.舍入和`BigDecimal`

通过舍入一个数字，我们用另一个更短、更简单、更有意义的数字来代替它。例如，我们将$24.784917 四舍五入到$24.78，因为我们没有小数美分。

要使用的精度和舍入模式因计算而异。例如，美国联邦纳税申报表指定使用`HALF_UP`四舍五入到整美元金额。

**有两个类控制舍入行为—`RoundingMode`和`MathContext`** 。

`enum [RoundingMode](https://web.archive.org/web/20221130044147/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/RoundingMode.html)` 提供八种成圆模式:

*   `CEILING –`向正无穷大舍入
*   `FLOOR –`向负无穷大舍入
*   `UP –`从零开始舍入
*   `DOWN –`向零舍入
*   `HALF_UP –`向“最近的邻居”舍入，除非两个邻居等距，在这种情况下会向上舍入
*   `HALF_DOWN –`向“最近的邻居”舍入，除非两个邻居等距，在这种情况下向下舍入
*   `HALF_EVEN – `向“最近的邻居”舍入，除非两个邻居等距，在这种情况下，向偶数邻居舍入
*   `UNNECESSARY –`不需要舍入，如果没有精确的结果，则抛出`ArithmeticException`

`HALF_EVEN`舍入模式将舍入操作产生的偏差降至最低。它经常被使用。它也被称为**银行家的四舍五入**。

**[`MathContext`](https://web.archive.org/web/20221130044147/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/MathContext.html) 封装了精度和舍入模式**。有几个预定义的 MathContexts:

*   `DECIMAL32`–7 位数精度和半偶数舍入模式
*   `DECIMAL64`–16 位精度，半偶数舍入模式
*   `DECIMAL128`–34 位精度，半偶数舍入模式
*   `UNLIMITED`–无限精度运算

使用这个类，我们可以使用指定的精度和舍入行为来舍入一个`BigDecimal`数:

```java
@Test
public void whenRoundingDecimal_thenExpectedResult() {
    BigDecimal bd = new BigDecimal("2.5");
    // Round to 1 digit using HALF_EVEN
    BigDecimal rounded = bd
        .round(new MathContext(1, RoundingMode.HALF_EVEN));

    assertEquals("2", rounded.toString());
}
```

现在，让我们通过一个示例计算来研究舍入概念。

让我们写一个方法，在给定数量和单价的情况下，计算一个项目的总支付金额。让我们也应用贴现率和销售税率。我们使用`setScale`方法将最终结果四舍五入到美分:

```java
public static BigDecimal calculateTotalAmount(BigDecimal quantity,
    BigDecimal unitPrice, BigDecimal discountRate, BigDecimal taxRate) { 
    BigDecimal amount = quantity.multiply(unitPrice);
    BigDecimal discount = amount.multiply(discountRate);
    BigDecimal discountedAmount = amount.subtract(discount);
    BigDecimal tax = discountedAmount.multiply(taxRate);
    BigDecimal total = discountedAmount.add(tax);

    // round to 2 decimal places using HALF_EVEN
    BigDecimal roundedTotal = total.setScale(2, RoundingMode.HALF_EVEN);

    return roundedTotal;
}
```

现在，让我们为这个方法编写一个单元测试:

```java
@Test
public void givenPurchaseTxn_whenCalculatingTotalAmount_thenExpectedResult() {
    BigDecimal quantity = new BigDecimal("4.5");
    BigDecimal unitPrice = new BigDecimal("2.69");
    BigDecimal discountRate = new BigDecimal("0.10");
    BigDecimal taxRate = new BigDecimal("0.0725");

    BigDecimal amountToBePaid = BigDecimalDemo
      .calculateTotalAmount(quantity, unitPrice, discountRate, taxRate);

    assertEquals("11.68", amountToBePaid.toString());
}
```

## 5.`BigInteger`

**`BigInteger`代表不可变的任意精度整数**。它类似于原始整数类型，但允许任意大的值。

**当涉及的整数大于`long`类型的极限时使用。**例如，50 的阶乘是`30414093201713378043612608166064768844377641568960512000000000000\.` 这个值对于`int or long`数据类型来说太大了。它只能存储在一个`BigInteger`变量中。

它广泛应用于安全和密码应用中。

**我们可以从一个`byte`数组或者`String`** 中创建`BigInteger`:

```java
@Test
public void whenBigIntegerCreatedFromConstructor_thenExpectedResult() {
    BigInteger biFromString = new BigInteger("1234567890987654321");
    BigInteger biFromByteArray = new BigInteger(
       new byte[] { 64, 64, 64, 64, 64, 64 });
    BigInteger biFromSignMagnitude = new BigInteger(-1,
       new byte[] { 64, 64, 64, 64, 64, 64 });

    assertEquals("1234567890987654321", biFromString.toString());
    assertEquals("70644700037184", biFromByteArray.toString());
    assertEquals("-70644700037184", biFromSignMagnitude.toString());
}
```

另外，**我们可以使用静态方法** `**valueOf**:`将`long`转换为`BigInteger`

```java
@Test
public void whenLongConvertedToBigInteger_thenValueMatches() {
    BigInteger bi =  BigInteger.valueOf(2305843009213693951L);

    assertEquals("2305843009213693951", bi.toString());
}
```

## 6.`BigInteger`上的操作

与`int`和`long`类似，`BigInteger`实现所有的算术和逻辑运算。但是，它不会使操作员过载。

它还实现了来自`Math`类的相应方法:`abs`、`min`、`max`、`pow`、`signum`。

**我们使用`compareTo`方法比较两个大整数的值:**

```java
@Test
public void givenBigIntegers_whentCompared_thenExpectedResult() {
    BigInteger i = new BigInteger("123456789012345678901234567890");
    BigInteger j = new BigInteger("123456789012345678901234567891");
    BigInteger k = new BigInteger("123456789012345678901234567892");

    assertTrue(i.compareTo(i) == 0);
    assertTrue(j.compareTo(i) > 0);
    assertTrue(j.compareTo(k) < 0);
}
```

**我们通过调用相应的方法进行算术运算:**

```java
@Test
public void givenBigIntegers_whenPerformingArithmetic_thenExpectedResult() {
    BigInteger i = new BigInteger("4");
    BigInteger j = new BigInteger("2");

    BigInteger sum = i.add(j);
    BigInteger difference = i.subtract(j);
    BigInteger quotient = i.divide(j);
    BigInteger product = i.multiply(j);

    assertEquals(new BigInteger("6"), sum);
    assertEquals(new BigInteger("2"), difference);
    assertEquals(new BigInteger("2"), quotient);
    assertEquals(new BigInteger("8"), product);
}
```

由于`BigInteger`是不可变的，**这些操作不会修改现有的对象。与**、`int`和`long`、**不同，这些操作不会溢出。**

**`BigInteger`有类似于`int``long`**的位操作。但是，我们需要使用方法而不是运算符:

```java
@Test
public void givenBigIntegers_whenPerformingBitOperations_thenExpectedResult() {
    BigInteger i = new BigInteger("17");
    BigInteger j = new BigInteger("7");

    BigInteger and = i.and(j);
    BigInteger or = i.or(j);
    BigInteger not = j.not();
    BigInteger xor = i.xor(j);
    BigInteger andNot = i.andNot(j);
    BigInteger shiftLeft = i.shiftLeft(1);
    BigInteger shiftRight = i.shiftRight(1);

    assertEquals(new BigInteger("1"), and);
    assertEquals(new BigInteger("23"), or);
    assertEquals(new BigInteger("-8"), not);
    assertEquals(new BigInteger("22"), xor);
    assertEquals(new BigInteger("16"), andNot);
    assertEquals(new BigInteger("34"), shiftLeft);
    assertEquals(new BigInteger("8"), shiftRight);
}
```

**它有额外的位操作方法**:

```java
@Test
public void givenBigIntegers_whenPerformingBitManipulations_thenExpectedResult() {
    BigInteger i = new BigInteger("1018");

    int bitCount = i.bitCount();
    int bitLength = i.bitLength();
    int getLowestSetBit = i.getLowestSetBit();
    boolean testBit3 = i.testBit(3);
    BigInteger setBit12 = i.setBit(12);
    BigInteger flipBit0 = i.flipBit(0);
    BigInteger clearBit3 = i.clearBit(3);

    assertEquals(8, bitCount);
    assertEquals(10, bitLength);
    assertEquals(1, getLowestSetBit);
    assertEquals(true, testBit3);
    assertEquals(new BigInteger("5114"), setBit12);
    assertEquals(new BigInteger("1019"), flipBit0);
    assertEquals(new BigInteger("1010"), clearBit3);
}
```

**`BigInteger`提供 GCD 计算和模运算的方法**:

```java
@Test
public void givenBigIntegers_whenModularCalculation_thenExpectedResult() {
    BigInteger i = new BigInteger("31");
    BigInteger j = new BigInteger("24");
    BigInteger k = new BigInteger("16");

    BigInteger gcd = j.gcd(k);
    BigInteger multiplyAndmod = j.multiply(k).mod(i);
    BigInteger modInverse = j.modInverse(i);
    BigInteger modPow = j.modPow(k, i);

    assertEquals(new BigInteger("8"), gcd);
    assertEquals(new BigInteger("12"), multiplyAndmod);
    assertEquals(new BigInteger("22"), modInverse);
    assertEquals(new BigInteger("7"), modPow);
}
```

**它还具有与素数生成和素性测试相关的方法**:

```java
@Test
public void givenBigIntegers_whenPrimeOperations_thenExpectedResult() {
    BigInteger i = BigInteger.probablePrime(100, new Random());

    boolean isProbablePrime = i.isProbablePrime(1000);
    assertEquals(true, isProbablePrime);
}
```

## 7.结论

在这个快速教程中，我们探索了类`BigDecimal`和`BigInteger. `,它们对于原始整数类型不能满足要求的高级数值计算非常有用。

像往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221130044147/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-numbers)