# 用 Java 创建一个身体质量指数计算器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-body-mass-index-calculator>

## 1.概观

在本教程中，我们将用 Java 创建一个身体质量指数计算器。

在开始实现之前，首先让我们理解身体质量指数的概念。

## 2.什么是身体质量指数？

身体质量指数代表身体质量指数。这是从一个人的身高和体重得出的值。

在身体质量指数的帮助下，我们可以发现一个人的体重是否健康。

让我们来看看计算身体质量指数的公式:

**身体质量指数=体重公斤/(身高米*身高米)**

根据身体质量指数范围，一个人被分为体重不足、正常、超重或肥胖:

| 身体质量指数山脉 | 种类 |
| 

```java
< 18.5
```

 | 体重不足 |
| 

```java
18.5 - 25
```

 | 常态 |
| 

```java
25 - 30
```

 | 超重 |
| 

```java
> 30
```

 | 肥胖的 |

例如，让我们计算一个体重为 100 公斤、身高为 1.524 米的人的身体质量指数。

身体质量指数= 100 / (1.524 * 1.524)

身体质量指数= 43.056

由于身体质量指数大于 30，这个人被归类为“超重”。

## 3.计算身体质量指数的 Java 程序

Java 程序由计算身体质量指数的公式和简单的`if`–`else`语句组成。使用公式和上表，我们可以找出个人所在的类别:

```java
static String calculateBMI(double weight, double height) {

    double bmi = weight / (height * height);

    if (bmi < 18.5) {
        return "Underweight";
    }
    else if (bmi < 25) {
        return "Normal";
    }
    else if (bmi < 30) {
        return "Overweight";
    }
    else {
       return "Obese";
    }
}
```

## 4.测试

让我们通过提供“肥胖”个体的身高和体重来测试代码 **:**

```java
@Test
public void whenBMIIsGreaterThanThirty_thenObese() {
    double weight = 50;
    double height = 1.524;
    String actual = BMICalculator.calculateBMI(weight, height);
    String expected = "Obese";

    assertThat(actual).isEqualTo(expected);
}
```

运行测试后，我们可以看到实际结果与预期结果相同。

## 5.结论

在本文中，我们学习了用 Java 创建一个身体质量指数计算器。我们还通过编写 JUnit 测试来测试实现。

和往常一样，教程的完整代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221005192434/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-math-3)