# Java 中的概率

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-probability>

## 1.概观

在本教程中，我们将看几个如何用 Java 实现概率的例子。

## 2.模拟基本概率

**在 Java 中模拟概率，我们要做的第一件事就是生成[随机数](/web/20220626104429/https://www.baeldung.com/cs/randomness)。**幸运的是，Java 为我们提供了大量的 [`random numbers generators`](/web/20220626104429/https://www.baeldung.com/java-generating-random-numbers) 。

在这种情况下，我们将使用 [`SplittableRandom`](https://web.archive.org/web/20220626104429/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/SplittableRandom.html) 类，因为它提供了高质量的随机性并且相对较快:

```java
SplittableRandom random = new SplittableRandom();
```

然后，我们需要在一个范围内生成一个数字，并将其与从该范围中选择的另一个数字进行比较。范围内的每个数字都有均等的机会被抽到。因为我们知道范围，我们知道抽到我们选择的号码的概率。这样我们就控制了概率:

```java
boolean probablyFalse = random.nextInt(10) == 0
```

在这个例子中，我们抽取了从 0 到 9 的数字。所以抽 0 的概率等于 10%。现在，让我们获取一个随机数，并测试选择的数字是否低于抽取的数字:

```java
boolean whoKnows = random.nextInt(1, 101) <= 50
```

这里，我们画了从 1 到 100 的数字。我们的随机数小于或等于 50 的概率正好是 50%。

## 3.均匀分布

到目前为止生成的值属于均匀分布。这意味着每一个事件，例如掷骰子，都有均等的机会发生。

### 3.1.以给定的概率调用函数

现在，假设我们想时不时地执行一项任务，并控制其概率。例如，我们运营一个电子商务网站，我们希望给 10%的用户提供折扣。

为此，让我们实现一个带三个参数的方法:在某些情况下调用一个供应商，在其余情况下调用另一个供应商，以及概率。

首先，我们使用 [Vavr](/web/20220626104429/https://www.baeldung.com/vavr) 将我们的`SplittableRandom`声明为`[Lazy](https://web.archive.org/web/20220626104429/https://javadoc.io/doc/io.vavr/vavr/0.9.2/io/vavr/Lazy.html)` 。这样，我们将只在第一次请求时实例化它一次:

```java
private final Lazy<SplittableRandom> random = Lazy.of(SplittableRandom::new); 
```

然后，我们将实现概率管理函数:

```java
public <T> withProbability(Supplier<T> positiveCase, Supplier<T> negativeCase, int probability) {
    SplittableRandom random = this.random.get();
    if (random.nextInt(1, 101) <= probability) {
        return positiveCase.get();
    } else {
        return negativeCase.get();
    }
}
```

### 3.2.蒙特卡罗方法的抽样概率

让我们颠倒一下我们在上一节中看到的过程。为此，我们将使用[蒙特卡罗](https://web.archive.org/web/20220626104429/https://en.wikipedia.org/wiki/Monte_Carlo_method)方法来测量概率。它生成大量的随机事件，并计算其中有多少满足所提供的条件。**当概率很难或不可能通过分析计算时，这很有用。**

例如，如果我们看六面骰子，我们知道掷出某个数字的概率是 1/6。但是，如果我们有一个未知边数的神秘骰子，就很难说概率会是多少。我们可以不去分析骰子，而只是将它滚动无数次，并计算某些事件发生了多少次。

让我们看看如何实现这种方法。首先，我们将尝试一百万次生成概率为 10%的数字 1，并对它们进行计数:

```java
int numberOfSamples = 1_000_000;
int probability = 10;
int howManyTimesInvoked = 
  Stream.generate(() -> randomInvoker.withProbability(() -> 1, () -> 0, probability))
    .limit(numberOfSamples)
    .mapToInt(e -> e)
    .sum();
```

然后，生成数的总和除以样本数将是事件概率的近似值:

```java
int monteCarloProbability = (howManyTimesInvoked * 100) / numberOfSamples; 
```

请注意，计算出的概率是近似值。样本数量越多，近似值就越好。

## 4.其他分布

均匀分布对于像游戏这样的建模工作很好。为了让游戏公平，所有的事件通常需要有相同的发生概率。

然而，在现实生活中，分布通常更复杂。不同的事情发生的机会并不相等。

比如极矮的人很少，极高的人也很少。大部分人都是中等身高，也就是说人的身高遵循[正态分布](https://web.archive.org/web/20220626104429/https://en.wikipedia.org/wiki/Normal_distribution)。如果我们需要生成随机的人类身高，那么生成随机的英尺数是不够的。

幸运的是，我们不需要自己实现底层的数学模型。**我们需要知道使用哪个发行版以及如何配置它**，例如，使用统计数据。

Apache Commons 库为我们提供了几个发行版的实现。让我们用它来实现正态分布:

```java
private static final double MEAN_HEIGHT = 176.02;
private static final double STANDARD_DEVIATION = 7.11;
private static NormalDistribution distribution =  new NormalDistribution(MEAN_HEIGHT, STANDARD_DEVIATION); 
```

使用这个 API 非常简单——[sample](https://web.archive.org/web/20220626104429/https://commons.apache.org/proper/commons-math/javadocs/api-3.2/org/apache/commons/math3/distribution/NormalDistribution.html#sample())方法从分布中抽取一个随机数:

```java
public static double generateNormalHeight() {
    return distribution.sample();
}
```

最后，让我们颠倒一下这个过程:

```java
public static double probabilityOfHeightBetween(double heightLowerExclusive, double heightUpperInclusive) {
    return distribution.probability(heightLowerExclusive, heightUpperInclusive);
}
```

结果，我们会得到一个人的身高在两个界限之间的概率。在这种情况下，较低和较高的高度。

## 5.结论

在本文中，我们学习了如何生成随机事件以及如何计算它们发生的概率。我们使用均匀分布和正态分布来模拟不同的情况。

完整的例子可以在 GitHub 上找到[。](https://web.archive.org/web/20220626104429/https://github.com/eugenp/tutorials/tree/master/core-java-modules/java-numbers-4)