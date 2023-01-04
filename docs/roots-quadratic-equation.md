# Java 程序来求一个二次方程的根

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/roots-quadratic-equation>

## 1.概观

在本文中，我们将看到如何用 Java 计算一个二次方程的解。我们将从定义什么是二次方程开始，然后我们将计算它的解，无论我们在实数系统还是复数系统中工作。

## 2.一个二次方程的解

给定实数 a ≠ 0，b，c，我们来考虑下面这个二次方程:`ax² + bx + c = 0`。

### 2.1.多项式的根

**这个方程的解也被称为多项式`ax² + bx + c`的根。**因此，让我们定义一个`Polynom`类。如果`a`系数等于 0，我们将抛出一个 [`IllegalArgumentException`](/web/20221102130416/https://www.baeldung.com/java-exceptions) :

```
public class Polynom {

    private double a;
    private double b;
    private double c;

    public Polynom(double a, double b, double c) {
        if (a==0) {
            throw new IllegalArgumentException("a can not be equal to 0");
        }
        this.a = a;
        this.b = b;
        this.c = c;
    }

    // getters and setters
}
```

我们将在实数系统中解这个方程:为此，我们将寻找一些`Double`解。

### 2.2.复数系统

我们还将展示如何在复数系统中求解这个方程。**Java**中没有复数的默认表示，所以我们将创建自己的表示。下面就给它一个 [`static`](/web/20221102130416/https://www.baeldung.com/java-static) 的方法`ofReal`来轻松转换实数。这将有助于以下步骤:

```
public class Complex {

    private double realPart;
    private double imaginaryPart;

    public Complex(double realPart, double imaginaryPart) {
        this.realPart = realPart;
        this.imaginaryPart = imaginaryPart;
    }

    public static Complex ofReal(double realPart) {
        return new Complex(realPart, 0);
    }

    // getters and setters
}
```

## 3.计算判别式

**量δ= b–4ac 称为二次方程的判别式。**要在 java 中计算 b 的平方，我们有两种解决方案:

*   将 b 自身相乘
*   使用 [`Math.pow`](/web/20221102130416/https://www.baeldung.com/java-math-pow) 将其提升到 2 的幂

让我们坚持使用第一种方法，并向`Polynom`类添加一个`getDiscriminant`方法:

```
public double getDiscriminant() {
    return b*b - 4*a*c;
}
```

## 4.获取解决方案

根据判别式的值，我们能够知道存在多少个解并计算它们。

### 4.1.通过严格的正判别式

**如果判别式是严格正的，方程有两个实解，(-b-√δ)/2a 和(-b+√δ)/2a:**

```
Double solution1 = (-polynom.getB() - Math.sqrt(polynom.getDiscriminant())) / (2 * polynom.getA());
Double solution2 = (-polynom.getB() + Math.sqrt(polynom.getDiscriminant())) / (2 * polynom.getA());
```

如果我们在复数系统中工作，那么我们只需要进行转换:

```
Complex solution1 = Complex.ofReal((-polynom.getB() - Math.sqrt(polynom.getDiscriminant())) / (2 * polynom.getA()));
Complex solution2 = Complex.ofReal((-polynom.getB() + Math.sqrt(polynom.getDiscriminant())) / (2 * polynom.getA()));
```

### 4.2.判别式等于零

**若判别式等于零，则方程有唯一的实解-b / 2a:**

```
Double solution = (double) -polynom.getB() / (2 * polynom.getA());
```

同样，如果我们在一个复数系统中工作，我们将以如下方式转换解决方案:

```
Complex solution = Complex.ofReal(-polynom.getB() / (2 * polynom.getA()));
```

### 4.3.用严格的否定判别式

**如果判别式是严格负的，则该方程在实数系中无解。但在复数系中可以求解:解为(-b–I √-δ)/2a 及其共轭(-b+I √-δ)/2a:**

```
Complex solution1 = new Complex(-polynom.getB() / (2* polynom.getA()), -Math.sqrt(-polynom.getDiscriminant()) / 2* polynom.getA());
Complex solution2 = new Complex(-polynom.getB() / (2* polynom.getA()), Math.sqrt(-polynom.getDiscriminant()) / 2* polynom.getA());
```

### 4.4.收集结果

**综上所述，让我们构建一个方法，当方程的解存在时，用方程的解填充一个`[List](/web/20221102130416/https://www.baeldung.com/java-collections)`。**在实数系统中，这种方法看起来是这样的:

```
public static List<Double> getPolynomRoots(Polynom polynom) {
    List<Double> roots = new ArrayList<>();
    double discriminant = polynom.getDiscriminant();
    if (discriminant > 0) {
        roots.add((-polynom.getB() - Math.sqrt(discriminant)) / (2 * polynom.getA()));
        roots.add((-polynom.getB() + Math.sqrt(discriminant)) / (2 * polynom.getA()));
    } else if (discriminant == 0) {
        roots.add(-polynom.getB() / (2 * polynom.getA()));
    }
    return roots;
}
```

如果我们在一个复数系统中工作，我们宁愿写:

```
public static List<Complex> getPolynomRoots(Polynom polynom) {
    List<Complex> roots = new ArrayList<>();
    double discriminant = polynom.getDiscriminant();
    if (discriminant > 0) {
        roots.add(Complex.ofReal((-polynom.getB() - Math.sqrt(discriminant)) / (2 * polynom.getA())));
        roots.add(Complex.ofReal((-polynom.getB() + Math.sqrt(discriminant)) / (2 * polynom.getA())));
    } else if (discriminant == 0) {
        roots.add(Complex.ofReal(-polynom.getB() / (2 * polynom.getA())));
    } else {
        roots.add(new Complex(-polynom.getB() / (2* polynom.getA()), -Math.sqrt(-discriminant) / 2* polynom.getA()));
        roots.add(new Complex(-polynom.getB() / (2* polynom.getA()), Math.sqrt(-discriminant) / 2* polynom.getA()));
    }
    return roots;
}
```

## 5.结论

在本教程中，我们看到了如何在 Java 中求解一个二次方程，无论我们使用的是实数还是复数。

和往常一样，代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221102130416/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-math-3)