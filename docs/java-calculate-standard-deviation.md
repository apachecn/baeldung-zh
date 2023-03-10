# 计算标准偏差的 Java 程序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-calculate-standard-deviation>

## 1.概观

[标准偏差](https://web.archive.org/web/20221208143845/https://en.wikipedia.org/wiki/Standard_deviation)(符号为 sigma–σ)是数据围绕平均值分布的度量。

在这个简短的教程中，我们将看到如何在 Java 中计算标准差。

## 2.计算标准偏差

标准差的计算公式是(∑(ų–Xi)^ ^ 2)/n 的平方根，其中:

*   ∑是每个元素的总和
*   Xi 是数组中的每个元素
*   ų是数组元素的平均值
*   n 是元素的数量

我们可以借助 Java 的 [`Math`](/web/20221208143845/https://www.baeldung.com/java-lang-math) 类轻松计算标准差:

```java
public static double calculateStandardDeviation(double[] array) {

    // get the sum of array
    double sum = 0.0;
    for (double i : array) {
        sum += i;
    }

    // get the mean of array
    int length = array.length;
    double mean = sum / length;

    // calculate the standard deviation
    double standardDeviation = 0.0;
    for (double num : array) {
        standardDeviation += Math.pow(num - mean, 2);
    }

    return Math.sqrt(standardDeviation / length);
}
```

让我们测试一下我们的方法:

```java
double[] array = {25, 5, 45, 68, 61, 46, 24, 95};
System.out.println("List of elements: " + Arrays.toString(array));

double standardDeviation = calculateStandardDeviation(array);
System.out.format("Standard Deviation = %.6f", standardDeviation);
```

结果将如下所示:

```java
List of elements: [25.0, 5.0, 45.0, 68.0, 61.0, 46.0, 24.0, 95.0]
Standard Deviation = 26.732179
```

## 3.结论

在这个快速教程中，我们学习了如何在 Java 中计算标准差。

和往常一样，本文的示例代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221208143845/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-math-3)