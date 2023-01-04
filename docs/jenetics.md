# 遗传学文库介绍

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jenetics>

## 1。简介

本系列的目的是解释遗传算法的概念，并展示最常见的实现方式。

在本教程中，我们将描述一个非常强大的 Jenetics Java 库，它可以用来解决各种优化问题。

如果你觉得需要学习更多关于遗传算法的知识，我们建议从[这篇文章](/web/20221208143856/https://www.baeldung.com/java-genetic-algorithm)开始。

## 2。它是如何工作的？

根据其[官方文档](https://web.archive.org/web/20221208143856/http://jenetics.io/)，Jenetics 是一个基于 Java 编写的进化算法的库。进化算法源于生物学，因为它们使用受生物进化启发的机制，如繁殖、突变、重组和选择。

**Jenetics 是使用 Java `Stream`接口实现的，因此它可以与 Java `Stream` API 的其余部分顺利协作。**

主要特点是:

*   **无摩擦最小化**–无需改变或调整适应度函数；我们只需更改`Engine`类的配置，就可以开始我们的第一个应用程序了
*   **无依赖性**–不需要运行时第三方库来使用 Jenetics
*   **Java 8 就绪**——完全支持`Stream`和 lambda 表达式
*   **多线程**–进化步骤可以并行执行

为了使用 Jenetics，我们需要将以下依赖项添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>io.jenetics</groupId>
    <artifactId>jenetics</artifactId>
    <version>3.7.0</version>
</dependency>
```

最新版本可以在 Maven Central 找到[。](https://web.archive.org/web/20221208143856/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22jenetics%22)

## 3。用例

为了测试 Jenetics 的所有特性，我们将尝试解决各种众所周知的优化问题，从简单的二进制算法开始，以背包问题结束。

### 3.1。简单遗传算法

假设我们需要解决最简单的二进制问题，其中我们需要优化由 0 和 1 组成的染色体中 1 位的位置。首先，我们需要定义适合该问题的工厂:

```java
Factory<Genotype<BitGene>> gtf = Genotype.of(BitChromosome.of(10, 0.5));
```

我们创建了长度为 10 的`BitChromosome`，染色体中有 1 的概率等于 0.5。

现在，让我们创建执行环境:

```java
Engine<BitGene, Integer> engine
  = Engine.builder(SimpleGeneticAlgorithm::eval, gtf).build();
```

`eval()`方法返回位数:

```java
private Integer eval(Genotype<BitGene> gt) {
    return gt.getChromosome().as(BitChromosome.class).bitCount();
}
```

在最后一步，我们开始进化并收集结果:

```java
Genotype<BitGene> result = engine.stream()
  .limit(500)
  .collect(EvolutionResult.toBestGenotype());
```

最终结果将类似于此:

```java
Before the evolution:
[00000010|11111100]
After the evolution:
[00000000|11111111]
```

我们设法优化了基因中 1 的位置。

### 3.2。子集和问题

Jenetics 的另一个用例是解决[子集和问题](https://web.archive.org/web/20221208143856/https://en.wikipedia.org/wiki/Subset_sum_problem)。简而言之，优化的挑战在于，给定一组整数，我们需要找到一个总和为零的非空子集。

Jenetics 中有预定义的接口来解决此类问题:

```java
public class SubsetSum implements Problem<ISeq<Integer>, EnumGene<Integer>, Integer> {
    // implementation
}
```

如我们所见，我们实现了`Problem<T, G, C>`，它有三个参数:

*   `**<T>**` –问题适应度函数的参数类型，在我们的例子中是不可变的、有序的、固定大小的`Integer`序列`ISeq<Integer>`
*   `**<G>**`–进化引擎正在处理的基因类型，在这种情况下，可数的`Integer`基因`EnumGene<Integer>`
*   `**<C>**`–适应度函数的结果类型；这是一个`Integer`

为了使用`Problem<T, G, C>` 接口，我们需要覆盖两个方法:

```java
@Override
public Function<ISeq<Integer>, Integer> fitness() {
    return subset -> Math.abs(subset.stream()
      .mapToInt(Integer::intValue).sum());
}

@Override
public Codec<ISeq<Integer>, EnumGene<Integer>> codec() {
    return codecs.ofSubSet(basicSet, size);
}
```

在第一个示例中，我们定义了我们的适应度函数，而第二个示例是一个包含工厂方法的类，用于创建常见的问题编码，例如，从给定的基本集合中找到最佳的固定大小子集，就像我们的例子一样。

现在我们可以进入主要部分了。开始时，我们需要创建一个子集用于问题:

```java
SubsetSum problem = of(500, 15, new LCG64ShiftRandom(101010));
```

请注意，我们使用的是 Jenetics 提供的`LCG64ShiftRandom`生成器。下一步，我们将构建解决方案的引擎:

下一步，我们将构建解决方案的引擎:

```java
Engine<EnumGene<Integer>, Integer> engine = Engine.builder(problem)
  .minimizing()
  .maximalPhenotypeAge(5)
  .alterers(new PartiallyMatchedCrossover<>(0.4), new Mutator<>(0.3))
  .build();
```

我们试图通过设置用于改变后代的表现型年龄和改变者来最小化结果(最佳结果将是 0)。在下一步中，我们可以获得结果:

```java
Phenotype<EnumGene<Integer>, Integer> result = engine.stream()
  .limit(limit.bySteadyFitness(55))
  .collect(EvolutionResult.toBestPhenotype());
```

请注意，我们使用的是返回谓词的`bySteadyFitness()`,如果在给定数量的代之后找不到更好的表型，它将截断进化流，并收集最佳结果。如果我们幸运的话，对于随机创建的集合有一个解决方案，我们会看到类似这样的东西:

如果我们幸运的话，对于随机创建的集合有一个解决方案，我们会看到类似这样的东西:

```java
[85|-76|178|-197|91|-106|-70|-243|-41|-98|94|-213|139|238|219] --> 0
```

否则，子集的总和将不同于 0。

### 3.3。背包首次拟合问题

Jenetics 库允许我们解决更复杂的问题，比如背包问题。简单来说，在这个问题中，我们的背包空间有限，我们需要决定将哪些物品放在里面。

让我们从定义箱包尺寸和物品数量开始:

```java
int nItems = 15;
double ksSize = nItems * 100.0 / 3.0;
```

在下一步中，我们将生成一个包含`KnapsackItem`个对象(由`size`和`value`字段定义)的随机数组，我们将使用首次适合方法将这些项目随机放入背包中:

```java
KnapsackFF ff = new KnapsackFF(Stream.generate(KnapsackItem::random)
  .limit(nItems)
  .toArray(KnapsackItem[]::new), ksSize);
```

接下来，我们需要创建`Engine`:

```java
Engine<BitGene, Double> engine
  = Engine.builder(ff, BitChromosome.of(nItems, 0.5))
  .populationSize(500)
  .survivorsSelector(new TournamentSelector<>(5))
  .offspringSelector(new RouletteWheelSelector<>())
  .alterers(new Mutator<>(0.115), new SinglePointCrossover<>(0.16))
  .build();
```

这里有几点需要注意:

*   人口规模为 500 人
*   后代将通过锦标赛和[轮盘](/web/20221208143856/https://www.baeldung.com/cs/genetic-algorithms-roulette-selection)的选择被选出
*   正如我们在前一小节中所做的那样，我们还需要为新创建的后代定义更改者

**jene tics 还有一个很重要的特点。我们可以轻松地从整个模拟过程中收集所有统计数据和见解。**我们将通过使用 `EvolutionStatistics` 类来实现这一点:

```java
EvolutionStatistics<Double, ?> statistics = EvolutionStatistics.ofNumber();
```

最后，让我们运行模拟:

```java
Phenotype<BitGene, Double> best = engine.stream()
  .limit(bySteadyFitness(7))
  .limit(100)
  .peek(statistics)
  .collect(toBestPhenotype());
```

请注意，我们将在每一代后更新评估统计数据，仅限于 7 个稳定代，总共最多 100 代。更详细地说，有两种可能的情况:

*   我们达到 7 个稳定的世代，然后模拟停止
*   我们不能在少于 100 代的时间内得到 7 个稳定的代，所以模拟由于第二个`limit()`而停止

设定最大代数限制很重要，否则模拟可能无法在合理的时间内停止。

最终结果包含大量信息:

```java
+---------------------------------------------------------------------------+
|  Time statistics                                                          |
+---------------------------------------------------------------------------+
|             Selection: sum=0,039207931000 s; mean=0,003267327583 s        |
|              Altering: sum=0,065145069000 s; mean=0,005428755750 s        |
|   Fitness calculation: sum=0,029678433000 s; mean=0,002473202750 s        |
|     Overall execution: sum=0,111383965000 s; mean=0,009281997083 s        |
+---------------------------------------------------------------------------+
|  Evolution statistics                                                     |
+---------------------------------------------------------------------------+
|           Generations: 12                                                 |
|               Altered: sum=7 664; mean=638,666666667                      |
|                Killed: sum=0; mean=0,000000000                            |
|              Invalids: sum=0; mean=0,000000000                            |
+---------------------------------------------------------------------------+
|  Population statistics                                                    |
+---------------------------------------------------------------------------+
|                   Age: max=10; mean=1,792167; var=4,657748                |
|               Fitness:                                                    |
|                      min  = 0,000000000000                                |
|                      max  = 716,684883338605                              |
|                      mean = 587,012666759785                              |
|                      var  = 17309,892287851708                            |
|                      std  = 131,567063841418                              |
+---------------------------------------------------------------------------+
```

这一次，我们能够将总价值为 716，68 的项目放在最佳方案中。我们还可以看到进化和时间的详细统计数据。

**怎么考？**

这是一个相当简单的过程——只需打开与问题相关的主文件，首先运行算法。一旦我们有了一个大概的想法，那么我们就可以开始研究参数了。

## 4。结论

在本文中，我们介绍了基于实际优化问题的 Jenetics 库特性。

该代码在 GitHub 上作为 Maven 项目提供。请注意，我们提供了更多优化挑战的代码示例，例如[斯普林斯汀记录](https://web.archive.org/web/20221208143856/https://softwareengineering.stackexchange.com/questions/326378/finding-the-best-combination-of-sets-that-gives-the-maximum-number-of-unique-ite)(是的，它存在！)和旅行推销员问题。

对于本系列的所有文章，包括遗传算法的其他例子，请查看以下链接:

*   [如何用 Java 设计遗传算法](/web/20221208143856/https://www.baeldung.com/java-genetic-algorithm)
*   [Java 中的旅行推销员问题](/web/20221208143856/https://www.baeldung.com/java-simulated-annealing-for-traveling-salesman)
*   [蚁群优化](/web/20221208143856/https://www.baeldung.com/java-ant-colony-optimization)
*   Jenetics 库介绍(this)