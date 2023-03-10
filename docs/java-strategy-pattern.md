# Java 8 中的策略设计模式

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-strategy-pattern>

## 1。简介

在本文中，我们将看看如何在 Java 8 中实现策略设计模式。

首先，我们将给出该模式的概述，并解释它在 Java 的旧版本中是如何实现的。

接下来，我们将再次尝试该模式，只是这次使用 Java 8 lambdas，减少代码的冗长。

## 2。战略模式

本质上，策略模式允许我们在运行时改变算法的行为。

通常，我们会从一个用于应用算法的接口开始，然后为每个可能的算法多次实现它。

假设我们有一个需求，根据是圣诞节、复活节还是新年，对采购应用不同类型的折扣。首先，让我们创建一个将由我们的每个策略实现的`Discounter` 接口:

```java
public interface Discounter {
    BigDecimal applyDiscount(BigDecimal amount);
} 
```

然后，假设我们想在复活节打五折，在圣诞节打九折。让我们为每个策略实现我们的接口:

```java
public static class EasterDiscounter implements Discounter {
    @Override
    public BigDecimal applyDiscount(final BigDecimal amount) {
        return amount.multiply(BigDecimal.valueOf(0.5));
    }
}

public static class ChristmasDiscounter implements Discounter {
   @Override
   public BigDecimal applyDiscount(final BigDecimal amount) {
       return amount.multiply(BigDecimal.valueOf(0.9));
   }
} 
```

最后，让我们在测试中尝试一个策略:

```java
Discounter easterDiscounter = new EasterDiscounter();

BigDecimal discountedValue = easterDiscounter
  .applyDiscount(BigDecimal.valueOf(100));

assertThat(discountedValue)
  .isEqualByComparingTo(BigDecimal.valueOf(50));
```

这工作得很好，但是问题是必须为每个策略创建一个具体的类可能有点痛苦。另一种方法是使用匿名内部类型，但这仍然很冗长，并且不比前面的解决方案方便多少:

```java
Discounter easterDiscounter = new Discounter() {
    @Override
    public BigDecimal applyDiscount(final BigDecimal amount) {
        return amount.multiply(BigDecimal.valueOf(0.5));
    }
}; 
```

## 3。利用 Java 8

自从 Java 8 发布以来，lambdas 的引入让匿名内部类型或多或少变得多余。这意味着在线创建策略变得更加简洁和容易。

此外，函数式编程的声明式风格让我们可以实现以前不可能实现的模式。

### 3.1.减少代码冗长

让我们尝试使用 lambda 表达式创建一个内联`EasterDiscounter,` :

```java
Discounter easterDiscounter = amount -> amount.multiply(BigDecimal.valueOf(0.5)); 
```

正如我们所看到的，我们的代码现在更干净，更易维护，实现了和以前一样的功能，但是只用了一行代码。本质上，**lambda 可以被视为匿名内部类型**的替代。

当我们想要在队列中声明更多的`Discounters` 时，这个优势变得更加明显:

```java
List<Discounter> discounters = newArrayList(
  amount -> amount.multiply(BigDecimal.valueOf(0.9)),
  amount -> amount.multiply(BigDecimal.valueOf(0.8)),
  amount -> amount.multiply(BigDecimal.valueOf(0.5))
);
```

当我们想要定义大量的`Discounters,` 时，我们可以在一个地方静态地声明它们。如果我们愿意，Java 8 甚至允许我们在接口中定义静态方法。

因此，与其在具体类或匿名内部类型之间进行选择，不如让我们尝试在单个类中创建 lambdas:

```java
public interface Discounter {
    BigDecimal applyDiscount(BigDecimal amount);

    static Discounter christmasDiscounter() {
        return amount -> amount.multiply(BigDecimal.valueOf(0.9));
    }

    static Discounter newYearDiscounter() {
        return amount -> amount.multiply(BigDecimal.valueOf(0.8));
    }

    static Discounter easterDiscounter() {
        return amount -> amount.multiply(BigDecimal.valueOf(0.5));
    }
} 
```

正如我们所看到的，我们在不太多的代码中取得了很大的成就。

### 3.2。杠杆功能组合

让我们修改我们的`Discounter` 接口，使其扩展`UnaryOperator` 接口，然后添加一个`combine()` 方法:

```java
public interface Discounter extends UnaryOperator<BigDecimal> {
    default Discounter combine(Discounter after) {
        return value -> after.apply(this.apply(value));
    }
}
```

本质上，我们正在重构我们的`Discounter`,并利用这样一个事实，即应用折扣是一个将`BigDecimal`实例转换成另一个`BigDecimal`实例`,` 的函数，允许我们访问预定义的方法*。* **由于`UnaryOperator`带有一个`apply()` 方法，我们可以用它来代替`applyDiscount` 。**

`combine()`方法只是对`this.`的结果应用一个`Discounter`的抽象，它使用内置的函数`apply()` 来实现这一点。

现在，让我们尝试将倍数`Discounters` 累计应用于一个金额。我们将通过使用函数`reduce()` 和我们的`combine():`来做到这一点

```java
Discounter combinedDiscounter = discounters
  .stream()
  .reduce(v -> v, Discounter::combine);

combinedDiscounter.apply(...);
```

特别注意第一个`reduce`参数。当没有提供折扣时，我们需要返回不变的值。这可以通过提供一个 identity 函数作为默认折扣器来实现。

这是执行标准迭代的一个有用且不太冗长的替代方法。如果我们考虑功能组合的现成方法，它也给了我们更多的免费功能。

## 4。结论

在本文中，我们解释了策略模式，并展示了如何使用 lambda 表达式以一种不太冗长的方式实现它。

这些例子的实现可以在 GitHub 的[中找到。这是一个基于 Maven 的项目，所以应该很容易运行。](https://web.archive.org/web/20220920161352/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-8)