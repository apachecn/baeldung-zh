# Apache Commons 数学简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-commons-math>

## 1。概述

我们经常需要使用数学工具，有时 *java.lang.Math* 根本不够用。幸运的是，Apache Commons 的目标是用 [Apache Commons Math](https://web.archive.org/web/20221208143856/https://commons.apache.org/proper/commons-math/) 填补标准库的漏洞。

Apache Commons Math 是最大的 Java 数学函数和实用程序开源库。鉴于这篇文章只是一个介绍，我们将只给出这个库的一个概述，并展示最引人注目的用例。

## 2。从 Apache Commons Math 开始

### 2.1。Apache Commons Math 的用法

Apache Commons Math 由数学函数( *erf* 举例)、表示数学概念的结构(如复数、多项式、向量等)组成。)，以及我们可以应用于这些结构的算法(求根、优化、曲线拟合、几何图形交点的计算等)。).

### 2.2。Maven 配置

如果您使用的是 Maven，只需添加[这个依赖关系](https://web.archive.org/web/20221208143856/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22commons-math3%22):

```
<dependency>
  <groupId>org.apache.commons</groupId>
  <artifactId>commons-math3</artifactId>
  <version>3.6.1</version>
</dependency> 
```

### 2.3。包装概述

Apache Commons Math 分为几个包:

*   `**org.apache.commons.math3.stat –**`统计和统计检验
*   概率分布
*   `**org.apache.commons.math3.random –**`随机数、字符串和数据生成
*   `**org.apache.commons.math3.analysis –**`求根、积分、插值、多项式等。
*   `**org.apache.commons.math3.linear –**`矩阵，求解线性系统
*   `**org.apache.commons.math3.geometry –**`几何学(欧几里得空间和二元空间分割)
*   `**org.apache.commons.math3.transform –**`变换方法(快速傅立叶变换)
*   `**org.apache.commons.math3.ode –**`常微分方程积分
*   `**org.apache.commons.math3.fitting –**`曲线拟合
*   `**org.apache.commons.math3.optim –**`函数最大化或最小化
*   `**org.apache.commons.math3.genetics –**`遗传算法
*   `**org.apache.commons.math3.ml –**`机器学习(聚类和神经网络)
*   `**org.apache.commons.math3.util –**`扩展 java.lang.Math 的常用 math/stat 函数
*   `**org.apache.commons.math3.special –**`特殊功能(伽马，贝塔)
*   `**org.apache.commons.math3.complex –**`复数
*   `**org.apache.commons.math3.fraction –**`有理数

## 3。统计、概率和随机性

### 3.1。统计数据

包*org . Apache . commons . math 3 . stat*为统计计算提供了几个工具。例如，为了计算平均值、标准差等等，我们可以使用*描述统计*:

```
double[] values = new double[] {65, 51 , 16, 11 , 6519, 191 ,0 , 98, 19854, 1, 32};
DescriptiveStatistics descriptiveStatistics = new DescriptiveStatistics();
for (double v : values) {
    descriptiveStatistics.addValue(v);
}

double mean = descriptiveStatistics.getMean();
double median = descriptiveStatistics.getPercentile(50);
double standardDeviation = descriptiveStatistics.getStandardDeviation(); 
```

在这个包中，我们可以找到计算协方差、相关性或执行统计测试的工具(使用 *TestUtils* )。

### 3.2。概率和分布

在核心 Java 中， *Math.random()* 可用于生成随机值，但这些值在 0 和 1 之间均匀分布。

有时，我们希望使用更复杂的分布来产生随机值。对此，我们可以使用*org . Apache . commons . math 3 . distribution*提供的框架。

下面是如何根据均值为 10、标准差为 3 的正态分布生成随机值:

```
NormalDistribution normalDistribution = new NormalDistribution(10, 3);
double randomValue = normalDistribution.sample(); 
```

或者我们可以获得离散分布的概率`P(X = x)`，或者连续分布的累积概率`P(X <= x)`。

## 4。分析

分析相关的函数和算法可以在*org . Apache . commons . math 3 . analysis*中找到。

### 4.1。求根

根是一个函数的值为 0 的值。Commons-Math 包括几个[求根算法](https://web.archive.org/web/20221208143856/https://commons.apache.org/proper/commons-math/userguide/analysis.html#a4.3_Root-finding)的实现。

在这里，我们试图找到*v->(v * v)-2*的根:

```
UnivariateFunction function = v -> Math.pow(v, 2) - 2;
UnivariateSolver solver = new BracketingNthOrderBrentSolver(1.0e-12, 1.0e-8, 5);
double c = solver.solve(100, function, -10.0, 10.0, 0); 
```

首先，我们从定义函数开始，然后定义求解器，并设置所需的精度。最后，我们调用`solve()` API。

求根操作将使用若干次迭代来执行，因此这是一个在执行时间和准确性之间寻找折衷的问题。

### 4.2。计算积分

集成的工作方式几乎和求根一样:

```
UnivariateFunction function = v -> v;
UnivariateIntegrator integrator = new SimpsonIntegrator(1.0e-12, 1.0e-8, 1, 32);
double i = integrator.integrate(100, function, 0, 10); 
```

我们首先定义一个函数，在现有的[个可用积分解](https://web.archive.org/web/20221208143856/https://commons.apache.org/proper/commons-math/userguide/analysis.html#a4.5_Integration)中选择一个积分器，设置所需的精度，最后进行积分。

## 5。线性代数

如果我们有一个 AX = B 形式的线性方程组，其中 A 是一个实数矩阵，B 是一个实数向量——Commons Math 提供了表示矩阵和向量的结构，还提供了求解器来计算 X 的值:

```
RealMatrix a = new Array2DRowRealMatrix(
  new double[][] { { 2, 3, -2 }, { -1, 7, 6 }, { 4, -3, -5 } },
  false);
RealVector b = new ArrayRealVector(n
  ew double[] { 1, -2, 1 }, 
  false);

DecompositionSolver solver = new LUDecomposition(a).getSolver();

RealVector solution = solver.solve(b); 
```

情况非常简单:我们从一个双精度数组定义一个矩阵 *a* ，从一个向量数组定义一个向量 *b* 。

然后，我们创建一个 *LUDecomposition* ，它为 AX = B 形式下的方程提供一个解算器。顾名思义， *LUDecomposition* 依赖于 [LU 分解](https://web.archive.org/web/20221208143856/https://en.wikipedia.org/wiki/LU_decomposition)，因此仅适用于方阵。

对于其他矩阵，存在不同的求解器，通常使用最小二乘法求解方程。

## 6。几何图形

包*org . Apache . commons . math 3 . geometry*提供了几个表示几何对象的类和几个操作它们的工具。需要注意的是，根据我们想要使用的几何形状，这个包被分成不同的子包:

需要注意的是，根据我们想要使用的几何形状，这个包被分成不同的子包:

*   `**org.apache.commons.math3.geometry.euclidean.oned –**` 1D 欧几里得几何
*   `**org.apache.commons.math3.geometry.euclidean.twod –**` 2D 欧几里得几何
*   `**org.apache.commons.math3.geometry.euclidean.threed –**`三维欧几里得几何
*   `**org.apache.commons.math3.geometry.spherical.oned –**` 1D 球面几何学
*   `**org.apache.commons.math3.geometry.spherical.twod –**` 2D 球面几何学

最有用的类大概是 *Vector2D* 、 *Vector3D* 、 *Line* 和 *Segment* 。它们分别用于表示 2D 矢量(或点)、三维矢量、直线和线段。

当使用上面提到的类时，执行一些计算是可能的。例如，下面的代码执行两条 2D 线交点的计算:

```
Line l1 = new Line(new Vector2D(0, 0), new Vector2D(1, 1), 0);
Line l2 = new Line(new Vector2D(0, 1), new Vector2D(1, 1.5), 0);

Vector2D intersection = l1.intersection(l2); 
```

使用这些结构来获取点到线的距离，或者一条线到另一条线的最近点也是可行的(在 3D 中)。

## 7。最优化、遗传算法和机器学习

Commons-Math 还为与优化和机器学习相关的更复杂的任务提供了一些工具和算法。

### 7.1。优化

最优化通常包括最小化或最大化成本函数。优化的算法可以在*org . Apache . commons . math 3 . optim*和*org . Apache . commons . math 3 . optimization*中找到。它包括线性和非线性优化算法。

我们可以注意到在 *optim* 和*优化*包中有重复的类:*优化*包大部分被弃用，将在 Commons Math 4 中被移除。

### 7.2。遗传算法

遗传算法是一种元启发式算法:当确定性算法太慢时，它们是找到问题的可接受解决方案的解决方案。遗传算法的概述可以在[这里](/web/20221208143856/https://www.baeldung.com/java-genetic-algorithm)找到。

包*org . Apache . commons . math 3 . genetics*提供了一个使用遗传算法进行计算的框架。它包含可用于表示群体和染色体的结构，以及执行变异、交叉和选择操作的标准算法。

以下类提供了一个很好的起点:

*   *[遗传算法](https://web.archive.org/web/20221208143856/https://commons.apache.org/proper/commons-math/javadocs/api-3.6.1/org/apache/commons/math3/genetics/GeneticAlgorithm.html)–*遗传算法框架
*   *[人口](https://web.archive.org/web/20221208143856/https://commons.apache.org/proper/commons-math/javadocs/api-3.6.1/org/apache/commons/math3/genetics/Population.html)–*表示人口的界面
*   *[染色体](https://web.archive.org/web/20221208143856/https://commons.apache.org/proper/commons-math/javadocs/api-3.6.1/org/apache/commons/math3/genetics/Chromosome.html)–*代表一条染色体的界面

### 7.3。机器学习

Commons-Math 中的机器学习分为两部分:聚类和神经网络。

聚类部分包括根据向量关于距离度量的相似性给向量加上标签。所提供的聚类算法基于 K-means 算法。

神经网络部分给出类来表示网络( [*网络*](https://web.archive.org/web/20221208143856/https://commons.apache.org/proper/commons-math/javadocs/api-3.6.1/index.html?org/apache/commons/math3/ml/neuralnet/Network.html) )和神经元( [*神经元*](https://web.archive.org/web/20221208143856/https://commons.apache.org/proper/commons-math/javadocs/api-3.6.1/index.html?org/apache/commons/math3/ml/neuralnet/Neuron.html) )。人们可能会注意到，与最常见的神经网络框架相比，所提供的功能是有限的，但它对于要求较低的小型应用程序仍然有用。

## 8。公用事业

### 8.1 .fastmath〔t1〕

*FastMath* 是一个静态类，位于*org . Apache . commons . math 3 . util*中，工作方式与 *java.lang.Math* 完全一样。

它的目的是提供至少与我们在 *java.lang.Math* 中找到的相同的功能，但是实现速度更快。因此，当一个程序严重依赖数学计算时，将对 *Math.sin()* (例如)的调用替换为对 *FastMath.sin()* 的调用来提高应用程序的性能是一个好主意。另一方面，请注意`FastMath`不如*准确*

### 8.2。普通和特殊功能

Commons-Math 提供了在 *java.lang.Math* 中没有实现的标准数学函数(比如阶乘)。这些函数中的大多数都可以在包*org . Apache . commons . math 3 . special*和*org . Apache . commons . math 3 . util*中找到。

例如，如果我们想计算 10 的阶乘，我们可以简单地这样做:

```
long factorial = CombinatorialUtils.factorial(10); 
```

与算术相关的函数( *gcd* 、 *lcm* 等)。)可以在*算术工具*中找到，与组合相关的函数可以在*组合工具*中找到。其他一些特殊函数，像 *erf* ，可以在*org . Apache . commons . math 3 . special*中访问。

### 8.3。分数和复数

还可以使用 commons-math: fraction 和复数来处理更复杂的类型。这些结构允许我们对这类数字进行特定的计算。

然后，我们可以计算两个分数的和，并将结果显示为分数的字符串表示形式(即，在“a / b”形式下):

```
Fraction lhs = new Fraction(1, 3);
Fraction rhs = new Fraction(2, 5);
Fraction sum = lhs.add(rhs);

String str = new FractionFormat().format(sum); 
```

或者，我们可以快速计算复数的幂:

```
Complex first = new Complex(1.0, 3.0);
Complex second = new Complex(2.0, 5.0);

Complex power = first.pow(second); 
```

## 9。结论

在本教程中，我们介绍了一些使用 Apache Commons Math 可以做的有趣的事情。

不幸的是，本文不能涵盖整个分析或线性代数领域，因此，只提供了最常见情况的例子。

然而，要了解更多信息，我们可以阅读写得很好的[文档](https://web.archive.org/web/20221208143856/https://commons.apache.org/proper/commons-math/userguide/)，它提供了该库各方面的大量细节。

和往常一样，代码样本可以在 GitHub 上找到[。](https://web.archive.org/web/20221208143856/https://github.com/eugenp/tutorials/tree/master/libraries-apache-commons)