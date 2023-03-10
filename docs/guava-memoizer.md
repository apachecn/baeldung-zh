# 番石榴发酵剂简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/guava-memoizer>

## 1.概观

在本教程中，我们将探索谷歌的番石榴图书馆的记忆功能。

记忆是一种通过缓存函数第一次执行的结果来避免重复执行计算量大的函数的技术。

### 1.1。记忆与缓存

就记忆存储而言，记忆化类似于缓存。这两种技术都试图通过减少对计算量大的代码的调用来提高效率。

然而，缓存是一个更通用的术语，在类实例化、对象检索或内容检索的层次上解决问题，而记忆化在方法/函数执行的层次上解决问题。

### 1.2。番石榴记忆器和番石榴缓存

Guava 支持记忆和缓存。**记忆适用于没有自变量的函数(`Supplier`)和只有一个自变量的函数(`Function`)。** `Supplier`和`Function`这里指的是 Guava 函数接口，是同名 Java 8 函数 API 接口的直接子类。

从 23.6 版本开始，Guava 不支持带有多个参数的函数的记忆化。

我们可以按需调用内存化 API，并指定一个逐出策略，该策略控制内存中保存的条目数量，并通过在条目符合策略条件时从缓存中逐出/删除条目来防止使用中的内存不受控制地增长。

记忆化利用番石榴缓存；有关番石榴缓存的更多详细信息，请参考我们的[番石榴缓存文章](/web/20220630142555/https://www.baeldung.com/guava-cache)。

## 2.`Supplier`记忆

在`Suppliers`类中有两个方法支持记忆化:`memoize`和`memoizeWithExpiration`。

当我们想要执行 memoized 方法时，我们可以简单地调用返回的`Supplier`的`get`方法。**根据方法的返回值是否存在于内存中，`get`方法要么返回内存中的值，要么执行内存化的方法并将返回值传递给调用者。**

让我们探索一下`Supplier`记忆的每一种方法。

### 2.1。`Supplier`不驱逐的记忆

我们可以使用`Suppliers` ' `memoize`方法，并指定委托的`Supplier` 作为方法引用:

```java
Supplier<String> memoizedSupplier = Suppliers.memoize(
  CostlySupplier::generateBigNumber);
```

由于我们还没有指定驱逐策略，**一旦调用了`get`方法，当 Java 应用程序还在运行时，返回值将保存在内存中。**初始调用后对`get` 的任何调用都将返回记忆值。

### 2.2。`Supplier`通过生存时间(TTL)驱逐的记忆化

假设我们只想将备忘录中从`Supplier`返回的值保留一段时间。

我们可以使用`Suppliers` ' `memoizeWithExpiration`的方法，除了委托的`Supplier`之外，还可以用相应的时间单位(如秒、分)来指定到期时间:

```java
Supplier<String> memoizedSupplier = Suppliers.memoizeWithExpiration(
  CostlySupplier::generateBigNumber, 5, TimeUnit.SECONDS);
```

**经过指定的时间(5 秒)后，缓存将从内存**中清除`Supplier`的返回值，并且任何对`get`方法的后续调用将重新执行 *generateBigNumber* 。

更多详细信息，请参考 [Javadoc](https://web.archive.org/web/20220630142555/https://google.github.io/guava/releases/23.0/api/docs/com/google/common/base/Suppliers.html#memoizeWithExpiration-com.google.common.base.Supplier-long-java.util.concurrent.TimeUnit-) 。

### 2.3。示例

让我们模拟一个名为`generateBigNumber`的计算开销很大的方法:

```java
public class CostlySupplier {
    private static BigInteger generateBigNumber() {
        try {
            TimeUnit.SECONDS.sleep(2);
        } catch (InterruptedException e) {}
        return new BigInteger("12345");
    }
}
```

我们的示例方法将花费 2 秒钟来执行，然后返回一个`BigInteger`结果。我们可以使用`memoize`或`memoizeWithExpiration`API`.` 来记忆它

为了简单起见，我们将省略驱逐策略:

```java
@Test
public void givenMemoizedSupplier_whenGet_thenSubsequentGetsAreFast() {
    Supplier<BigInteger> memoizedSupplier; 
    memoizedSupplier = Suppliers.memoize(CostlySupplier::generateBigNumber);

    BigInteger expectedValue = new BigInteger("12345");
    assertSupplierGetExecutionResultAndDuration(
      memoizedSupplier, expectedValue, 2000D);
    assertSupplierGetExecutionResultAndDuration(
      memoizedSupplier, expectedValue, 0D);
    assertSupplierGetExecutionResultAndDuration(
      memoizedSupplier, expectedValue, 0D);
}

private <T> void assertSupplierGetExecutionResultAndDuration(
  Supplier<T> supplier, T expectedValue, double expectedDurationInMs) {
    Instant start = Instant.now();
    T value = supplier.get();
    Long durationInMs = Duration.between(start, Instant.now()).toMillis();
    double marginOfErrorInMs = 100D;

    assertThat(value, is(equalTo(expectedValue)));
    assertThat(
      durationInMs.doubleValue(), 
      is(closeTo(expectedDurationInMs, marginOfErrorInMs)));
}
```

第一个`get`方法调用需要两秒钟，就像在`generateBigNumber`方法中模拟的那样；然而，**对`get()`的后续调用会执行得更快，因为`generateBigNumber`结果已经被记忆了。**

## 3.`Function`记忆

为了记住一个采用单个参数的方法，我们**使用`CacheLoader`的`from`方法构建了一个`LoadingCache`映射，将我们的方法作为一个番石榴`Function.`** 提供给构建器

`LoadingCache`是并发映射，值由`CacheLoader`自动加载。 **`CacheLoader`通过计算`from`方法中指定的`Function`、**并将返回值放入`LoadingCache`来填充映射。更多详细信息，请参考 [Javadoc](https://web.archive.org/web/20220630142555/https://google.github.io/guava/releases/23.0/api/docs/com/google/common/cache/CacheLoader.html#from-com.google.common.base.Function-) 。

`LoadingCache`的键是`Function`的参数/输入，而 map 的值是`Function`的返回值:

```java
LoadingCache<Integer, BigInteger> memo = CacheBuilder.newBuilder()
  .build(CacheLoader.from(FibonacciSequence::getFibonacciNumber));
```

由于`LoadingCache`是一个并发映射，**不允许空键或空值。**因此，我们需要确保`Function`不支持 null 作为参数或返回 null 值。

### 3.1。`Function`采用驱逐政策的记忆化

正如[番石榴缓存文章](/web/20220630142555/https://www.baeldung.com/guava-cache)的第 3 节所提到的，当我们记忆一个`Function` 时，我们可以应用不同的番石榴缓存的驱逐策略。

例如，我们可以驱逐空闲了 2 秒钟的条目:

```java
LoadingCache<Integer, BigInteger> memo = CacheBuilder.newBuilder()
  .expireAfterAccess(2, TimeUnit.SECONDS)
  .build(CacheLoader.from(Fibonacci::getFibonacciNumber));
```

接下来，我们来看看`Function`记忆化的两个用例:斐波那契数列和阶乘。

### 3.2。斐波那契数列例子

我们可以从一个给定的数字`n`递归计算一个斐波那契数:

```java
public static BigInteger getFibonacciNumber(int n) {
    if (n == 0) {
        return BigInteger.ZERO;
    } else if (n == 1) {
        return BigInteger.ONE;
    } else {
        return getFibonacciNumber(n - 1).add(getFibonacciNumber(n - 2));
    }
}
```

**没有记忆化，输入值比较高的时候，功能执行会比较慢。**

为了提高效率和性能，如果有必要，我们可以使用指定驱逐策略的`CacheLoader`和`CacheBuilder,`来记忆`getFibonacciNumber`。

在下面的示例中，一旦 memo 大小达到 100 个条目，我们将删除最旧的条目:

```java
public class FibonacciSequence {
    private static LoadingCache<Integer, BigInteger> memo = CacheBuilder.newBuilder()
      .maximumSize(100)
      .build(CacheLoader.from(FibonacciSequence::getFibonacciNumber));

    public static BigInteger getFibonacciNumber(int n) {
        if (n == 0) {
            return BigInteger.ZERO;
        } else if (n == 1) {
            return BigInteger.ONE;
        } else {
            return memo.getUnchecked(n - 1).add(memo.getUnchecked(n - 2));
        }
    }
}
```

这里，我们使用`getUnchecked`方法，如果存在返回值，它不会抛出检查过的异常。

在这种情况下，当在`CacheLoader`的`from`方法调用中指定`getFibonacciNumber`方法引用时，我们不需要显式处理异常。

更多详细信息，请参考 [Javadoc](https://web.archive.org/web/20220630142555/https://google.github.io/guava/releases/23.0/api/docs/com/google/common/cache/LoadingCache.html#getUnchecked-K-) 。

### 3.3。阶乘示例

接下来，我们有另一个递归方法来计算给定输入值 n 的阶乘:

```java
public static BigInteger getFactorial(int n) {
    if (n == 0) {
        return BigInteger.ONE;
    } else {
        return BigInteger.valueOf(n).multiply(getFactorial(n - 1));
    }
}
```

我们可以通过应用记忆来提高实现的效率:

```java
public class Factorial {
    private static LoadingCache<Integer, BigInteger> memo = CacheBuilder.newBuilder()
      .build(CacheLoader.from(Factorial::getFactorial));

    public static BigInteger getFactorial(int n) {
        if (n == 0) {
            return BigInteger.ONE;
        } else {
            return BigInteger.valueOf(n).multiply(memo.getUnchecked(n - 1));
        }
    }
}
```

## 4.结论

在本文中，我们看到了 Guava 如何提供 API 来执行对`Supplier`和`Function` 方法的记忆。我们还展示了如何指定内存中存储函数结果的回收策略。

和往常一样，源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220630142555/https://github.com/eugenp/tutorials/tree/master/guava-modules/guava-core)