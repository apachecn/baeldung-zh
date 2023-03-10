# 用 Java 计算百分比

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-calculate-percentage>

## 1.介绍

在这个快速教程中，我们将使用 Java 实现一个 CLI 程序来计算百分比。

但首先，让我们定义如何用数学方法计算百分比。

## 2.数学公式

在数学中，百分比是用 100 的分数表示的数字或比率。它通常用百分号“%”来表示。

让我们考虑一个学生，他获得了 y 分中的 x 分。计算该学生分数的公式是:

> `percentage = (x/y)*100`

## 3.Java 程序

现在我们已经清楚了如何用数学方法计算百分比，让我们用 Java 编写一个程序来计算它:

```java
public class PercentageCalculator {

    public double calculatePercentage(double obtained, double total) {
        return obtained * 100 / total;
    }

    public static void main(String[] args) {
        PercentageCalculator pc = new PercentageCalculator();
        Scanner in = new Scanner(System.in);
        System.out.println("Enter obtained marks:");
        double obtained = in.nextDouble();
        System.out.println("Enter total marks:");
        double total = in.nextDouble();
        System.out.println(
          "Percentage obtained: " + pc.calculatePercentage(obtained, total));
    }
}
```

这个程序从 CLI 中获取学生的分数(获得的分数和总分数),然后调用`calculatePercentage()`方法来计算分数。

这里我们选择 double 作为输入和输出的数据类型，因为它可以存储精度高达 16 位的十进制数。因此，它对于我们的用例应该是足够的。

## 4.输出

让我们运行这个程序，看看结果:

```java
Enter obtained marks:
87
Enter total marks:
100
Percentage obtained: 87.0

Process finished with exit code 0
```

## 5.结论

在本文中，我们研究了如何用数学方法计算百分比，然后编写了一个 Java CLI 程序来计算它。

最后，和往常一样，示例中使用的代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220901112516/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-math)