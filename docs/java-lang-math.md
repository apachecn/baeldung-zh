# Java 数学课程指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-lang-math>

## 1。简介

在本教程中，我们将描述`Math`类，它提供了有用的静态方法来执行数字运算，如指数、对数等。

## 2。基本数学函数

我们将介绍的第一组方法是基本的数学函数，例如绝对值、平方根、两个值之间的最大值或最小值。

### 2.1。`abs()`

`abs()`方法返回给定值的绝对值:

```java
Math.abs(-5); // returns 5
```

同样，在我们接下来要看到的其他函数中，`abs()`接受一个`int, long, float`或`double`作为参数，并返回相对值。

### 2.2。`pow()`

计算并返回第一个参数的第二次幂的值:

```java
Math.pow(5,2); // returns 25
```

我们在[这里的](/web/20220626204657/https://www.baeldung.com/java-math-pow)中更详细地讨论了这种方法。

### 2.3。`sqrt()`

返回一个`double`的舍入正平方根:

```java
Math.sqrt(25); // returns 5
```

如果参数为`[NaN](/web/20220626204657/https://www.baeldung.com/java-not-a-number)`或小于零，则结果为`[NaN](/web/20220626204657/https://www.baeldung.com/java-not-a-number).`

### 2.4。`cbrt()`

类似地，`cbrt()`返回一个`double`的立方根:

```java
Math.cbrt(125); // returns 5
```

### 2.5。`max()`

顾名思义，该方法返回两个值之间的最大值:

```java
Math.max(5,10); // returns 10
```

这里，该方法再次接受`int, long, float`或`double`。

### 2.6。`min() `

同样，`min()`返回两个值之间的最小值:

```java
Math.min(5,10); // returns 5
```

### 2.7。`random()`

返回大于或等于 0.0 且小于 1.0 的伪随机值`double `:

```java
double random = Math.random()
```

为此，**方法在第一次被调用时会创建一个`java.util.Random() `数字生成器的实例。**

此后，对于对此方法的所有调用，都使用同一个实例。请注意，该方法是同步的，因此可以由多个线程使用。

我们可以在本文的[中找到更多关于如何生成随机数的例子。](/web/20220626204657/https://www.baeldung.com/java-generate-random-long-float-integer-double)

### 2.8。`signum()`

当我们必须知道值的符号时，这是很有用的:

```java
Math.signum(-5) // returns -1
```

如果参数大于零，此方法返回 1.0，否则返回-1.0。如果参数为零正或零负，则结果与参数相同。

输入可以是`float `或`double.`

### 2.9。`copySign()`

接受两个参数，并返回第一个参数和第二个参数的符号:

```java
Math.copySign(5,-1); // returns -5
```

参数也可以是`float`或`double.`

## 3。指数和对数函数

除了基本的数学函数，**`Math`类包含了求解指数和对数函数的方法。**

### 3.1。`exp()`

`exp()`方法接收一个`double` 参数并返回该参数的欧拉数的幂( *e* ^(`x`) ):

```java
Math.exp(1); // returns 2.718281828459045
```

### 3.2。`expm1()`

与上面的方法类似，`expm1()`计算欧拉数的接收自变量的幂，但它增加了-1(exp-1):

```java
Math.expm1(1); // returns 1.718281828459045
```

### 3.3。`log()`

返回一个`double`值的自然对数:

```java
Math.log(Math.E); // returns 1
```

### 3.4。`log10()`

它返回参数以 10 为底的对数:

```java
Math.log10(10); // returns 1
```

### 3.5。`log1p()`

类似的还有`log(), `，但是它给参数 ln(1 + x)加 1:

```java
Math.log1p(Math.E); // returns 1.3132616875182228
```

## 4.三角函数

**当我们必须处理几何公式时，我们总是需要三角函数；`Math`类为我们提供了这些。**

### 4.1。`sin()`

接收表示角度(以弧度为单位)的单个`double`参数，并返回三角正弦值:

```java
Math.sin(Math.PI/2); // returns 1
```

### 4.2。`cos()`

同样，`cos()`返回一个角度的三角余弦值(以弧度为单位):

```java
Math.cos(0); // returns 1
```

### 4.3。`tan()`

返回角度的三角正切值(以弧度为单位):

```java
Math.tan(Math.PI/4); // returns 1
```

### 4.4。`sinh(), cosh(), tanh()`

**分别返回一个`double`值的双曲正弦、双曲余弦和双曲正切:**

```java
Math.sinh(Math.PI);

Math.cosh(Math.PI);

Math.tanh(Math.PI);
```

### 4.5。 `**asin()**`

返回收到的参数的反正弦值:

```java
Math.asin(1); // returns pi/2
```

结果是一个在-`pi`/2 到`pi` /2 范围内的角度。

### 4.6。`acos()`

返回收到的参数的反余弦值:

```java
Math.acos(0); // returns pi/2
```

结果是 0 到`pi`范围内的角度。

### 4.7。`atan()`

返回收到的参数的反正切值:

```java
Math.atan(1); // returns pi/4
```

结果是一个在-`pi`/2 到`pi` /2 范围内的角度。

### 4.8。`atan2()`

最后，`atan2()`接收纵坐标`y`和横坐标`x,`，返回直角坐标`(x,y)`到极坐标`(r, ϑ)`转换的角度`ϑ`:

```java
Math.atan2(1,1); // returns pi/4
```

### 4.9。`toDegrees()`

当我们需要将弧度转换为度数时，这种方法很有用:

```java
Math.toDegrees(Math.PI); // returns 180
```

### 4.10.`**toRadians()**`

另一方面，`toRadians()`有助于进行相反的转换:

```java
Math.toRadians(180); // returns pi
```

请记住，我们在本节中看到的大多数方法都接受以弧度为单位的参数，因此，当我们有一个以度为单位的角度时，这种方法应该在使用三角方法之前使用。

更多例子，请看这里的[。](/web/20220626204657/https://www.baeldung.com/java-math-sin-degrees)

## 5。舍入和其他功能

最后，我们来看看舍入方法。

### 5.1。`ceil()`

当我们必须将一个整数舍入到大于或等于参数的最小`double`值时，`ceil()`非常有用:

```java
Math.ceil(Math.PI); // returns 4
```

在[这篇](/web/20220626204657/https://www.baeldung.com/java-round-up-nearest-hundred) [文章](/web/20220626204657/https://www.baeldung.com/java-round-up-nearest-hundred)中，我们用这种方法将一个数四舍五入到最接近的百。

### 5.2。`floor()`

要将一个数字四舍五入到小于或等于自变量的最大值`double`，我们应该使用`floor()`:

```java
Math.floor(Math.PI); // returns 3
```

### 5.3。`getExponent()`

返回参数的无偏指数。

参数可以是一个`double`或一个`float`:

```java
Math.getExponent(333.3); // returns 8

Math.getExponent(222.2f); // returns 7
```

### 5.4。`IEEEremainder()`

计算第一个参数(被除数)和第二个参数(除数)之间的除法，并按照 IEEE 754 标准的规定返回余数:

```java
Math.IEEEremainder(5,2); // returns 1
```

### 5.5。`nextAfter()`

当我们需要知道`double`或`float` 值的相邻值时，这种方法很有用:

```java
Math.nextAfter(1.95f,1); // returns 1.9499999

Math.nextAfter(1.95f,2); // returns 1.9500002
```

它接受两个参数，第一个是要知道相邻数字的值，第二个是方向。

### 5.6。`nextUp()`

与前面的方法类似，但这种方法只返回正无穷大方向的相邻值:

```java
Math.nextUp(1.95f); // returns 1.9500002
```

### 5.7。`rint()`

返回一个`double `，它是参数的最接近的整数值:

```java
Math.rint(1.95f); // returns 2.0
```

### 5.8。`round()`

与上面的方法相同，但是如果参数是`float `这个方法返回一个`int`值，如果参数是`double:`这个方法返回一个`long`值

```java
int result = Math.round(1.95f); // returns 2

long result2 = Math.round(1.95) // returns 2
```

### 5.9。`scalb()`

Scalb 是“scale binary”的缩写。该函数执行一次移位、一次转换和一次乘法运算:

```java
Math.scalb(3, 4); // returns 3*2^4
```

### 5.10。`ulp()`

`ulp() `方法返回一个数字到其最近邻居的距离:

```java
Math.ulp(1); // returns 1.1920929E-7
Math.ulp(2); // returns 2.3841858E-7
Math.ulp(4); // returns 4.7683716E-7
Math.ulp(8); // returns 9.536743E-7
```

### 5.11。`hypot()`

返回其参数平方和的平方根:

```java
Math.hypot(4, 3); // returns 5
```

该方法在没有中间溢出或下溢的情况下计算平方根。

在[这篇文章](/web/20220626204657/https://www.baeldung.com/java-distance-between-two-points)中，我们用这个方法计算两点之间的距离。

## 6。Java 8 数学函数

**`Math `类在 Java 8 中被重新定义，加入了执行最常见算术运算的新方法。**

我们在另一篇文章中讨论了这些方法。

## 7。常量字段

除了这些方法，`Math `类还声明了两个常量字段:

```java
public static final double E

public static final double PI
```

它们分别表示更接近自然对数底的值和更接近`pi`的值。

## 8。结论

在本文中，我们描述了 Java 为数学运算提供的 API。

像往常一样，这里展示的所有代码片段都可以在 GitHub 上获得。