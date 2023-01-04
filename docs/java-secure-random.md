# Java SecureRandom 类

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-secure-random>

## 1.介绍

在这个简短的教程中，我们将学习`java.security.SecureRandom,` 这个类提供了一个加密的强随机数生成器。

## 2.与`java.util.Random`的比较

`java.util.Random`的标准 JDK 实现使用[线性同余生成器](https://web.archive.org/web/20221019211536/https://en.wikipedia.org/wiki/Linear_congruential_generator) (LCG)算法来提供随机数。这个算法的问题是它在加密方面不够强大。换句话说，生成的值更容易预测，因此攻击者可以利用它来危害我们的系统。

为了解决这个问题，我们应该**在任何安全决策**中使用`java.security.SecureRandom`。它通过使用**密码强伪随机数发生器** ( [CSPRNG](https://web.archive.org/web/20221019211536/https://en.wikipedia.org/wiki/Cryptographically_secure_pseudorandom_number_generator) )产生密码强随机值。

为了更好地理解 LCG 和 CSPRNG 之间的差异，请查看下图，其中显示了两种算法的值分布:

[![secure random algorithms](img/4b31517aa39bd483b2696fd8d36374a3.png)](/web/20221019211536/https://www.baeldung.com/wp-content/uploads/2019/07/secure_random_algorithms.png)

## 3.生成随机值

使用`SecureRandom`最常见的方式是**生成`int`、`long`、`float`、`double`或`boolean`值**:

```
int randomInt = secureRandom.nextInt();
long randomLong = secureRandom.nextLong();
float randomFloat = secureRandom.nextFloat();
double randomDouble = secureRandom.nextDouble();
boolean randomBoolean = secureRandom.nextBoolean();
```

为了生成`int`值，我们可以传递一个上限作为参数:

```
int randomInt = secureRandom.nextInt(upperBound);
```

此外，我们可以**为`int,` `double`和`long`生成一个值流**:

```
IntStream randomIntStream = secureRandom.ints();
LongStream randomLongStream = secureRandom.longs();
DoubleStream randomDoubleStream = secureRandom.doubles();
```

对于所有流，我们可以显式设置流大小:

```
IntStream intStream = secureRandom.ints(streamSize);
```

以及原点(包括)和边界(不包括)值:

```
IntStream intStream = secureRandom.ints(streamSize, originValue, boundValue);
```

我们也可以生成一个随机字节序列。`nextBytes()`函数获取用户提供的`byte`数组并用随机的 `byte`填充它:

```
byte[] values = new byte[124];
secureRandom.nextBytes(values);
```

## 4.选择算法

**默认情况下，`SecureRandom`使用 SHA1PRNG 算法**生成随机值。我们可以通过调用`getInstance()`方法显式地让它使用另一种算法:

```
SecureRandom secureRandom = SecureRandom.getInstance("NativePRNG");
```

用`new`操作符创建`SecureRandom`相当于`SecureRandom.getInstance(“SHA1PRNG”)`。

Java 中所有可用的随机数生成器都可以在官方文档页面上找到[。](https://web.archive.org/web/20221019211536/https://docs.oracle.com/en/java/javase/11/docs/specs/security/standard-names.html#securerandom-number-generation-algorithms)

## 5.种子

每个`SecureRandom`实例都是用一个初始种子创建的。它作为提供随机值的基础，并在我们每次生成新值时发生变化。

使用`new`操作符或调用`SecureRandom.getInstance()`将[从`/dev/urandom`](https://web.archive.org/web/20221019211536/https://tersesystems.com/blog/2015/12/17/the-right-way-to-use-securerandom/) 获取默认种子。

我们可以通过将种子作为构造函数参数传递来更改它:

```
byte[] seed = getSecureRandomSeed();
SecureRandom secureRandom = new SecureRandom(seed);
```

或者通过在已经创建的对象上调用 setter 方法:

```
byte[] seed = getSecureRandomSeed();
secureRandom.setSeed(seed);
```

请记住，如果我们用相同的种子创建两个`SecureRandom`实例，并且对每个实例进行相同的方法调用序列，那么它们将**生成并返回相同的数字序列。**

## 6.结论

在本教程中，我们已经了解了`SecureRandom`是如何工作的，以及如何使用它来生成随机值。

和往常一样，本教程中的所有代码都可以在 GitHub 上找到[。](https://web.archive.org/web/20221019211536/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-security)