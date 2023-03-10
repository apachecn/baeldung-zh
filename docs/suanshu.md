# 《算书概论》

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/suanshu>

## 1。简介

是一个 Java 数学库，用于数值分析、统计、求根、线性代数、最优化等等。它提供了实数和复数的功能。

该库有一个开源版本，也有一个需要许可证的版本——有不同形式的许可证:学术、商业和贡献者。

请注意，以下示例通过`pom.xml`使用许可版本。开源版本目前在 Maven 知识库中不可用；许可版本需要运行许可服务器。因此，GitHub 中没有针对这个包的任何测试。

## 2。算术设置

让我们从将 Maven 依赖项添加到`pom.xml`开始:

```java
<dependencies>
    <dependency>
        <groupId>com.numericalmethod</groupId>
        <artifactId>suanshu</artifactId>
        <version>4.0.0</version>
    </dependency>
</dependencies>
<repositories>
    <repository>
        <id>nm-repo</id>
        <name>Numerical Method's Maven Repository</name>
        <url>http://repo.numericalmethod.com/maven/</url>
        <layout>default</layout>
    </repository>
</repositories>
```

## 3。使用向量

**算术库为`dense`向量和`sparse`向量提供了类。**`dense`向量是大多数元素具有非零值的向量，与`sparse`向量相反，后者大多数值为零。

一个`dense`向量的实现简单地使用一个实数/复数的 Java 数组，而一个`sparse`向量的实现使用一个`entries`的 Java 数组，其中每个`entry`都有一个索引和一个实数/复数的值。

我们可以看到，当我们有一个大向量，其中大多数值为零时，这将如何在存储上产生巨大的差异。当需要支持大尺寸的向量时，大多数数学库都使用这样的方法。

让我们看看一些基本的向量运算。

### 3.1。添加向量

使用`add()`方法将两个向量相加非常简单:

```java
public void addingVectors() throws Exception {
    Vector v1 = new DenseVector(new double[] {1, 2, 3, 4, 5});
    Vector v2 = new DenseVector(new double[] {5, 4, 3, 2, 1});
    Vector v3 = v1.add(v2);
    log.info("Adding vectors: {}", v3);
}
```

我们将看到的输出是:

```java
[6.000000, 6.000000, 6.000000, 6.000000, 6.000000]
```

我们还可以使用`add(double)`方法将相同的数字添加到所有元素中。

### 3.2。缩放矢量

缩放矢量(即乘以常数)也非常简单:

```java
public void scaleVector() throws Exception {
    Vector v1 = new DenseVector(new double[]{1, 2, 3, 4, 5});
    Vector v2 = v1.scaled(2.0);
    log.info("Scaling a vector: {}", v2);
}
```

输出:

```java
[2.000000, 4.000000, 6.000000, 8.000000, 10.000000]
```

### 3.3。向量内积

计算两个向量的内积需要调用`innerProduct(Vector)`方法:

```java
public void innerProductVectors() throws Exception {
    Vector v1 = new DenseVector(new double[]{1, 2, 3, 4, 5});
    Vector v2 = new DenseVector(new double[]{5, 4, 3, 2, 1});
    double inner = v1.innerProduct(v2);
    log.info("Vector inner product: {}", inner);
}
```

### 3.4。处理错误

该库验证我们正在操作的向量与我们正在执行的操作是否兼容。例如，将大小为 2 的向量添加到大小为 3 的向量应该是不可能的。所以下面的代码应该会导致一个异常:

```java
public void addingIncorrectVectors() throws Exception {
    Vector v1 = new DenseVector(new double[] {1, 2, 3});
    Vector v2 = new DenseVector(new double[] {5, 4});
    Vector v3 = v1.add(v2);
}
```

的确如此——运行这段代码会导致:

```java
Exception in thread "main" com.numericalmethod.suanshu.vector.doubles.IsVector$SizeMismatch: vectors do not have the same size: 3 and 2
    at com.numericalmethod.suanshu.vector.doubles.IsVector.throwIfNotEqualSize(IsVector.java:101)
    at com.numericalmethod.suanshu.vector.doubles.dense.DenseVector.add(DenseVector.java:174)
    at com.baeldung.suanshu.SuanShuMath.addingIncorrectVectors(SuanShuMath.java:21)
    at com.baeldung.suanshu.SuanShuMath.main(SuanShuMath.java:8)
```

## 4。使用矩阵

**除了矢量，该库还提供了对矩阵运算的支持。**与向量类似，矩阵支持`dense`和`sparse`格式，适用于实数和复数。

### 4.1。添加矩阵

添加矩阵就像处理向量一样简单:

```java
public void addingMatrices() throws Exception {
    Matrix m1 = new DenseMatrix(new double[][]{
        {1, 2, 3},
        {4, 5, 6}
    });

    Matrix m2 = new DenseMatrix(new double[][]{
        {3, 2, 1},
        {6, 5, 4}
    });

    Matrix m3 = m1.add(m2);
    log.info("Adding matrices: {}", m3);
}
```

### 4.2。乘法矩阵

数学库可用于矩阵乘法:

```java
public void multiplyMatrices() throws Exception {
    Matrix m1 = new DenseMatrix(new double[][]{
        {1, 2, 3},
        {4, 5, 6}
    });

    Matrix m2 = new DenseMatrix(new double[][]{
        {1, 4},
        {2, 5},
        {3, 6}
    });

    Matrix m3 = m1.multiply(m2);
    log.info("Multiplying matrices: {}", m3);
}
```

2×3 矩阵乘以 3×2 矩阵将得到 2×2 矩阵。

为了证明该库对矩阵大小进行了正确的检查，让我们尝试做一个可能会失败的乘法:

```java
public void multiplyIncorrectMatrices() throws Exception {
    Matrix m1 = new DenseMatrix(new double[][]{
        {1, 2, 3},
        {4, 5, 6}
    });

    Matrix m2 = new DenseMatrix(new double[][]{
        {3, 2, 1},
        {6, 5, 4}
    });

    Matrix m3 = m1.multiply(m2);
}
```

执行该命令将产生以下输出。

```java
Exception in thread "main" com.numericalmethod.suanshu.matrix.MatrixMismatchException:
    matrix with 3 columns and matrix with 2 rows cannot multiply due to mis-matched dimension
    at com.numericalmethod.suanshu.datastructure.DimensionCheck.throwIfIncompatible4Multiplication(DimensionCheck.java:164)
    at com.numericalmethod.suanshu.matrix.doubles.matrixtype.dense.DenseMatrix.multiply(DenseMatrix.java:374)
    at com.baeldung.suanshu.SuanShuMath.multiplyIncorrectMatrices(SuanShuMath.java:98)
    at com.baeldung.suanshu.SuanShuMath.main(SuanShuMath.java:22)
```

### 4.3。计算矩阵的逆矩阵

手动计算矩阵的逆矩阵可能是一个漫长的过程，但算术数学库使它变得很容易:

```java
public void inverseMatrix() {
    Matrix m1 = new DenseMatrix(new double[][]{
        {1, 2},
        {3, 4}
    });

    Inverse m2 = new Inverse(m1);
    log.info("Inverting a matrix: {}", m2);
}
```

我们可以使用算术库来验证这一点，但是将矩阵与其逆矩阵相乘:结果应该是单位矩阵。我们可以通过在上面的方法中添加以下内容来做到这一点:

```java
log.info("Verifying a matrix inverse: {}", m1.multiply(m2));
```

## 5。求解多项式

算盘支持的另一个领域是多项式。它提供了计算多项式的方法，也提供了求其根的方法(多项式计算结果为 0 时的输入值)。

### 5.1。创建多项式

多项式可以通过指定其系数来创建。因此，类似于`3x²-5x+1`的多项式可以用以下公式创建:

```java
public Polynomial createPolynomial() {
    return new Polynomial(new double[]{3, -5, 1});
}
```

正如我们所看到的，我们首先从最高度数的系数开始。

### 5.2。评估多项式

`evaluate()`方法可用于计算多项式。对于实数和复数输入，可以这样做。

```java
public void evaluatePolynomial(Polynomial p) {
    log.info("Evaluating a polynomial using a real number: {}", p.evaluate(5));
    log.info("Evaluating a polynomial using a complex number: {}", p.evaluate(new Complex(1, 2)));
}
```

我们将看到的输出是:

```java
51.0
-13.000000+2.000000i
```

### 5.3。求多项式的根

算术数学库使寻找多项式的根变得很容易。它提供了众所周知的算法来确定各种次数的多项式的根，并且基于多项式的最高次数，PolyRoot 类选择最佳方法:

```java
public void solvePolynomial() {
    Polynomial p = new Polynomial(new double[]{2, 2, -4});
    PolyRootSolver solver = new PolyRoot();
    List<? extends Number> roots = solver.solve(p);
    log.info("Finding polynomial roots: {}", roots);
}
```

输出:

```java
[-2.0, 1.0]
```

所以这个样本多项式有 2 个实根:-2 和 1。自然，也支持复杂的根。

## 6。结论

本文只是对算书数学库的一个简单介绍。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20221208143919/https://github.com/eugenp/tutorials/tree/master/libraries-data-2)