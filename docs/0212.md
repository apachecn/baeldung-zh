# 打印帕斯卡三角形的 Java 程序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-pascal-triangle>

## 1.概观

[帕斯卡三角形](https://web.archive.org/web/20221218181728/https://en.wikipedia.org/wiki/Pascal%27s_triangle)是二项式系数以三角形形式的排列。帕斯卡三角形的数字是这样排列的，每个数字都是它上面两个数字的和。

在本教程中，我们将看到如何用 Java 打印帕斯卡的三角形。

## 2.使用递归

我们可以使用递归打印帕斯卡三角形，公式为`nCr` : n！/((n–r)！r！)

首先，让我们创建一个递归函数:

```
public int factorial(int i) {
    if (i == 0) {
        return 1;
    }
    return i * factorial(i - 1);
}
```

然后我们可以使用该函数打印三角形:

```
private void printUseRecursion(int n) {
    for (int i = 0; i <= n; i++) {
        for (int j = 0; j <= n - i; j++) {
            System.out.print(" ");
        }

        for (int k = 0; k <= i; k++) {
            System.out.print(" " + factorial(i) / (factorial(i - k) * factorial(k)));
        }

        System.out.println();
    }
}
```

n = 5 的结果如下所示:

```
 1
      1 1
     1 2 1
    1 3 3 1
   1 4 6 4 1
  1 5 10 10 5 1
```

## 3.避免使用递归

另一种不用递归打印帕斯卡三角形的方法是使用二项式展开。

我们总是在每一行的开始有值 1，那么在第(n)行和第(I)位置的 k 值将被计算为:

```
k = ( k * (n - i) / i ) 
```

让我们用这个公式创建我们的函数:

```
public void printUseBinomialExpansion(int n) {
    for (int line = 1; line <= n; line++) {
        for (int j = 0; j <= n - line; j++) {
            System.out.print(" ");
        }

        int k = 1;
        for (int i = 1; i <= line; i++) {
            System.out.print(k + " ");
            k = k * (line - i) / i;
        }

        System.out.println();
    }
}
```

## 4.结论

在这个快速教程中，我们学习了用 Java 打印帕斯卡三角形的两种方法。

本文的示例代码可以在 GitHub 的[中找到。](https://web.archive.org/web/20221218181728/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-math-3)