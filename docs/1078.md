# 在 Java 中生成随机数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-generating-random-numbers>

## 1.概观

在本教程中，我们将探索在 Java 中生成随机数的不同方法。

## 2.使用 Java API

Java API 为我们提供了几种方法来实现我们的目的。让我们看看其中的一些。

### 2.1.`java.lang.Math`

**`Math`类的`random`方法将返回一个范围从 0.0(含)到 1.0(不含)的`double`值。**让我们看看如何使用它来获得一个由`min`和`max`定义的给定范围内的随机数:

```
int randomWithMathRandom = (int) ((Math.random() * (max - min)) + min);
```

### 2.2.`java.util.Random`

**在 Java 1.7 之前，最流行的生成随机数的方式是使用`nextInt`。**该方法有两种使用方式，带参数和不带参数。无参数调用以近似相等的概率返回任何一个`int`值。所以，很有可能我们会得到负数:

```
Random random = new Random();
int randomWithNextInt = random.nextInt();
```

如果我们使用带有`bound`参数的`netxInt`调用，我们将得到一个范围内的数字:

```
int randomWintNextIntWithinARange = random.nextInt(max - min) + min;
```

这将给出一个介于 0(含)和参数(不含)之间的数字。**所以，绑定参数必须大于 0。**否则，我们会得到一个`java.lang.IllegalArgumentException`。

**Java 8 引入了新的`ints`方法，返回一个`java.util.stream.IntStream.`** 让我们看看如何使用它们。

不带参数的`ints`方法返回一个无限制的`int`值流:

```
IntStream unlimitedIntStream = random.ints();
```

我们还可以传入一个参数来限制流的大小:

```
IntStream limitedIntStream = random.ints(streamSize);
```

当然，我们可以设置生成范围的最大值和最小值:

```
IntStream limitedIntStreamWithinARange = random.ints(streamSize, min, max);
```

### 2.3.`java.util.concurrent.ThreadLocalRandom`

Java 1.7 版本给我们带来了一种新的更有效的方法，通过`ThreadLocalRandom`类生成随机数。这个与`Random`类有三个重要的区别:

*   我们不需要显式地启动一个新的`ThreadLocalRandom`实例。这有助于我们避免创建大量无用实例和浪费垃圾收集器时间的错误
*   我们不能为`ThreadLocalRandom`设定种子，这会导致一个真正的问题。如果我们需要设置种子，那么我们应该避免这种产生随机数的方式
*   `Random`类在多线程环境中表现不佳

现在，让我们看看它是如何工作的:

```
int randomWithThreadLocalRandomInARange = ThreadLocalRandom.current().nextInt(min, max);
```

有了 Java 8 或以上，我们有了新的可能。首先，`nextInt`方法有两种变体:

```
int randomWithThreadLocalRandom = ThreadLocalRandom.current().nextInt();
int randomWithThreadLocalRandomFromZero = ThreadLocalRandom.current().nextInt(max);
```

其次，也是更重要的，我们可以使用`ints`方法:

```
IntStream streamWithThreadLocalRandom = ThreadLocalRandom.current().ints();
```

### 2.4.`java.util.SplittableRandom`

Java 8 还为我们带来了一个真正快速的生成器——[`SplittableRandom`](https://web.archive.org/web/20221003184240/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/SplittableRandom.html)类。

正如我们在 JavaDoc 中看到的，这是一个用于并行计算的生成器。知道实例不是线程安全的很重要。所以，我们在使用这个类的时候要小心。

我们有`nextInt`和`ints`两种方法。通过`nextInt`,我们可以使用两个参数调用直接设置上限和下限范围:

```
SplittableRandom splittableRandom = new SplittableRandom();
int randomWithSplittableRandom = splittableRandom.nextInt(min, max);
```

这种使用方式检查`max`参数是否大于`min`。否则，我们会得到一个`IllegalArgumentException`。**然而，它并不检查我们使用的是正数还是负数。因此，任何参数都可以是负的。**此外，我们有可用的单参数和零参数调用。它们的工作方式和我们之前描述的一样。

我们也可以使用`ints`方法。这意味着我们可以很容易地得到一串`int`值。为了澄清，我们可以选择有一个有限的或无限的流。对于有限的流，我们可以设置数字生成范围的上限和下限:

```
IntStream limitedIntStreamWithinARangeWithSplittableRandom = splittableRandom.ints(streamSize, min, max);
```

### 2.5.`java.security.SecureRandom`

**如果我们有安全敏感的应用，应该考虑使用 [`SecureRandom`](/web/20221003184240/https://www.baeldung.com/java-secure-random) 。**这是一个强密码生成器。默认构造的实例不使用密码随机种子。因此，我们应该:

*   设定种子—因此，种子将是不可预测的
*   将`java.util.secureRandomSeed`系统属性设置为`true`

这个类继承自`java.util.Random`。因此，我们可以使用上面看到的所有方法。例如，如果我们需要获得任何一个`int`值，那么我们将不带参数地调用`nextInt`:

```
SecureRandom secureRandom = new SecureRandom();
int randomWithSecureRandom = secureRandom.nextInt();
```

另一方面，如果我们需要设置范围，我们可以用`bound`参数调用它:

```
int randomWithSecureRandomWithinARange = secureRandom.nextInt(max - min) + min;
```

我们必须记住，如果参数不大于零，这种使用方法会抛出`IllegalArgumentException`。

## 3.使用第三方 API

正如我们所看到的，Java 为我们提供了许多生成随机数的类和方法。但是，也有用于此目的的第三方 API。

我们将看看其中的一些。

### 3.1.`org.apache.commons.math3.random.RandomDataGenerator`

Apache commons 项目的 Commons 数学库中有很多生成器。最简单的，也可能是最有用的，是`RandomDataGenerator`。它采用 [`Well19937c`](https://web.archive.org/web/20221003184240/https://en.wikipedia.org/wiki/Well_equidistributed_long-period_linear) 算法进行随机生成。但是，我们可以提供我们的算法实现。

让我们看看如何使用它。首先，我们必须添加依赖性:

```
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-math3</artifactId>
    <version>3.6.1</version>
</dependency>
```

最新版本的`commons-math3`可以在 [Maven Central](https://web.archive.org/web/20221003184240/https://search.maven.org/search?q=g:commons-math%20AND%20a:commons-math) 上找到。

然后我们可以开始使用它:

```
RandomDataGenerator randomDataGenerator = new RandomDataGenerator();
int randomWithRandomDataGenerator = randomDataGenerator.nextInt(min, max);
```

### 3.2.`it.unimi.dsi.util.XoRoShiRo128PlusRandom`

当然，这是最快的随机数生成器实现之一。它是由米兰大学信息科学系开发的。

该库也可以在 Maven Central 仓库中获得。因此，让我们添加依赖关系:

```
<dependency>
    <groupId>it.unimi.dsi</groupId>
    <artifactId>dsiutils</artifactId>
    <version>2.6.0</version>
</dependency>
```

这个生成器继承自`java.util.Random`。然而，如果我们看一看 [JavaDoc](https://web.archive.org/web/20221003184240/http://dsiutils.di.unimi.it/docs/it/unimi/dsi/util/XoRoShiRo128PlusRandom.html) ，我们意识到只有一种方法可以使用它——通过`nextInt`方法。最重要的是，这个方法只适用于零参数和单参数调用。任何其他调用都将直接使用`java.util.Random`方法。

例如，如果我们想得到一个范围内的随机数，我们可以写:

```
XoRoShiRo128PlusRandom xoroRandom = new XoRoShiRo128PlusRandom();
int randomWithXoRoShiRo128PlusRandom = xoroRandom.nextInt(max - min) + min;
```

## 4.结论

有几种方法可以实现随机数生成。但是，没有最好的办法。因此，我们应该选择最适合我们需要的。

完整的例子可以在 GitHub 上找到[。](https://web.archive.org/web/20221003184240/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-numbers-3)