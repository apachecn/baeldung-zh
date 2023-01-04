# 在 Java 中向上舍入到最接近的百

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-round-up-nearest-hundred>

## 1.概观

在这个快速教程中，我们将演示如何将一个给定的数字四舍五入到最接近的一百位。

比如:
`99`变成`100`
`200.2`变成`300`
`400`变成`400`

## 2.履行

首先，我们将对输入参数调用`Math.ceil()`。 **`Math.ceil()`返回大于或等于自变量的最小整数。例如，**如果输入的是 200.2 `Math.ceil()`就会返回 201。

接下来，我们将结果加上 99，然后除以 100。我们利用整数[除法](https://web.archive.org/web/20220526060311/https://docs.oracle.com/javase/specs/jls/se7/html/jls-15.html#jls-15.17.2) **来截断商的小数部分。**最后，我们将商乘以 100，得到我们想要的输出。

下面是我们的实现:

```java
static long round(double input) {
    long i = (long) Math.ceil(input);
    return ((i + 99) / 100) * 100;
};
```

## 3.测试

让我们测试一下实现:

```java
@Test
public void givenInput_whenRound_thenRoundUpToTheNearestHundred() {
    assertEquals("Rounded up to hundred", 100, RoundUpToHundred.round(99));
    assertEquals("Rounded up to three hundred ", 300, RoundUpToHundred.round(200.2));
    assertEquals("Returns same rounded value", 400, RoundUpToHundred.round(400));
}
```

## 4.结论

在这篇简短的文章中，我们展示了如何将一个数字四舍五入到最接近的百。

和往常一样，完整的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220526060311/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-math-2)