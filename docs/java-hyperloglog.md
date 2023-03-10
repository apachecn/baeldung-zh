# Java 中超对数算法指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-hyperloglog>

## 1。概述

`HyperLogLog (HLL)` 数据结构是一种概率数据结构，用于**估计数据集**的基数。

假设我们有数百万用户，我们想计算我们的网页的不同访问量。一个简单的实现是将每个唯一的用户 id 存储在一个集合中，然后这个集合的大小就是我们的基数。

当我们处理非常大量的数据时，用这种方式计算基数是非常低效的，因为数据集将占用大量内存。

但是如果我们可以接受百分之几的估计值，并且不需要特定访问的准确数量，那么我们可以使用`HLL`，因为它正是为这样一个用例而设计的——**估计数百万甚至数十亿不同值的计数**。

## 2。Maven 依赖关系

首先，我们需要为 [`hll`](https://web.archive.org/web/20221012192025/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22net.agkn%22%20AND%20a%3A%22hll%22) 库添加 Maven 依赖项:

```java
<dependency>
    <groupId>net.agkn</groupId>
    <artifactId>hll</artifactId>
    <version>1.6.0</version>
</dependency>
```

## 3。使用`HLL` 估算基数

直接进入——`HLL`构造函数有两个参数，我们可以根据需要进行调整:

*   `log2m (log base 2) –` 这是`HLL` 内部使用的寄存器数量(注意:我们指定的是`m`)
*   `regwidth –` 这是每个寄存器使用的位数

如果我们想要更高的精度，我们需要设置更高的值。这样的配置会有额外的开销，因为我们的`HLL` 会占用更多的内存。如果我们可以接受较低的精度，我们可以降低这些参数，我们的`HLL` 将占用更少的内存。

让我们创建一个`HLL` 来计算具有 1 亿个条目的数据集的不同值。我们将把`log2m`参数设置为等于`14`，将`regwidth` 参数设置为等于`5`——对于这种规模的数据集来说，这是合理的值。

**当每个新元素被插入到`HLL`中时，它需要被预先散列。**我们将使用来自番石榴库的`Hashing.murmur3_128()` (包含在`hll`依赖项中)，因为它既准确又快速。

```java
HashFunction hashFunction = Hashing.murmur3_128();
long numberOfElements = 100_000_000;
long toleratedDifference = 1_000_000;
HLL hll = new HLL(14, 5);
```

选择这些参数将使我们的**误差率低于百分之一** (1，000，000 个元素)。我们一会儿将对此进行测试。

接下来，让我们插入 1 亿个元素:

```java
LongStream.range(0, numberOfElements).forEach(element -> {
    long hashedValue = hashFunction.newHasher().putLong(element).hash().asLong();
    hll.addRaw(hashedValue);
  }
);
```

最后，我们可以测试由`HLL` 返回的基数是否在我们期望的错误阈值之内:

```java
long cardinality = hll.cardinality();
assertThat(cardinality)
  .isCloseTo(numberOfElements, Offset.offset(toleratedDifference));
```

## 4。`HLL`的内存大小

我们可以使用下面的公式来计算上一节的`HLL`将占用多少内存:`numberOfBits = 2 ^ log2m * regwidth`。

在我们的例子中，这将是`2 ^ 14 * 5` 位(大约是 `81000` 位或 `8100` 字节)。所以使用`HLL` 估计 1 亿个成员集的基数只占用了 8100 字节的内存。

让我们将它与一个简单的 set 实现进行比较。在这样的实现中，我们需要 1 亿个`Long`值的`Set`，这将占用`100,000,000 * 8` 字节 `= 800,000,000` 字节`.`

我们可以看到差异惊人的高。使用`HLL`，我们只需要 8100 字节，而使用简单的`Set`实现，我们大约需要 800 兆字节。

当我们考虑更大的数据集时，`HLL`和简单的`Set`实现之间的差异变得更大。

## 5。两个`HLLs`的结合

`HLL`在执行联合*时有一个有利属性。*当我们获取从不同数据集创建的两个`HLLs`的并集并测量其基数**时，我们将获得与使用单个`HLL`并从头开始计算两个数据集的所有元素的哈希值**时相同的错误阈值。

请注意，当我们联合两个 hll 时，它们应该具有相同的`log2m` 和`regwidth` 参数，以产生正确的结果。

让我们通过创建两个`HLLs –`来测试该属性，一个填充 0 到 1 亿的值，第二个填充 1 亿到 2 亿的值:

```java
HashFunction hashFunction = Hashing.murmur3_128();
long numberOfElements = 100_000_000;
long toleratedDifference = 1_000_000;
HLL firstHll = new HLL(15, 5);
HLL secondHLL = new HLL(15, 5);

LongStream.range(0, numberOfElements).forEach(element -> {
    long hashedValue = hashFunction.newHasher()
      .putLong(element)
      .hash()
      .asLong();
    firstHll.addRaw(hashedValue);
    }
);

LongStream.range(numberOfElements, numberOfElements * 2).forEach(element -> {
    long hashedValue = hashFunction.newHasher()
      .putLong(element)
      .hash()
      .asLong();
    secondHLL.addRaw(hashedValue);
    }
);
```

请注意，我们调整了`HLLs`的配置参数，将`log2m`参数从上一节中看到的 14 增加到本例中的 15，因为产生的`HLL`联合将包含两倍多的元素。

接下来，让我们使用`union()` 方法联合`firstHll` 和`secondHll` 。正如您所看到的，估计的基数在一个错误阈值内，就好像我们从一个有 2 亿个元素的`HLL` 中获取基数一样:

```java
firstHll.union(secondHLL);
long cardinality = firstHll.cardinality();
assertThat(cardinality)
  .isCloseTo(numberOfElements * 2, Offset.offset(toleratedDifference * 2)); 
```

## 6。结论

在本教程中，我们看了一下`HyperLogLog`算法。

我们看到了如何使用`HLL` 来估计一个集合的基数。我们还看到，与简单的解决方案相比，`HLL`非常节省空间。我们对两个`HLLs`执行了联合操作，并验证了联合的行为方式与单个`HLL`相同。

所有这些例子和代码片段的实现都可以在 [GitHub 项目](https://web.archive.org/web/20221012192025/https://github.com/eugenp/tutorials/tree/master/libraries-data-2)中找到；这是一个 Maven 项目，因此应该很容易导入和运行。