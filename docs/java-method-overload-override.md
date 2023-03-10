# Java 中的方法重载和覆盖

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-method-overload-override>

## 1。概述

方法重载和重写是 Java 编程语言的关键概念，因此值得深入研究。

在本文中，我们将学习这些概念的基础知识，并看看它们在什么情况下有用。

## 2。方法重载

方法重载是一种强大的机制，它允许我们定义内聚的类 API。为了更好地理解为什么方法重载是如此有价值的特性，让我们看一个简单的例子。

假设我们已经编写了一个简单的实用程序类，它实现了两个数字相乘、三个数字相乘等等的不同方法。

如果我们给这些方法起了误导的或者模糊的名字，比如`multiply2()`、`multiply3()`、`multiply4(),` ，那么这将是一个设计糟糕的类 API。这就是方法重载发挥作用的地方。

**简单来说，我们可以用两种不同的方式实现方法重载:**

*   实现两个或更多的**方法，它们具有相同的名称，但是采用不同数量的参数**
*   实现两个或多个**方法，这些方法具有相同的名称，但采用不同类型的参数**

### 2.1。不同数量的参数

简单地说，`Multiplier`类展示了如何通过简单地定义两个采用不同数量参数的实现来重载`multiply()`方法:

```java
public class Multiplier {

    public int multiply(int a, int b) {
        return a * b;
    }

    public int multiply(int a, int b, int c) {
        return a * b * c;
    }
}
```

### 2.2。不同类型的论点

类似地，我们可以通过让 `multiply()`方法接受不同类型的参数来重载它:

```java
public class Multiplier {

    public int multiply(int a, int b) {
        return a * b;
    }

    public double multiply(double a, double b) {
        return a * b;
    }
} 
```

此外，用两种类型的方法重载来定义`Multiplier`类是合理的:

```java
public class Multiplier {

    public int multiply(int a, int b) {
        return a * b;
    }

    public int multiply(int a, int b, int c) {
        return a * b * c;
    }

    public double multiply(double a, double b) {
        return a * b;
    }
} 
```

然而，值得注意的是，**不可能有两个方法实现只在返回类型**上不同。

为了理解原因，让我们考虑下面的例子:

```java
public int multiply(int a, int b) { 
    return a * b; 
}

public double multiply(int a, int b) { 
    return a * b; 
}
```

在这种情况下，**由于方法调用不明确**，代码根本无法编译——编译器不知道要调用`multiply()`的哪个实现。

### 2.3。类型提升

方法重载提供的一个简洁的特性是所谓的`type promotion, a.k.a. widening primitive conversion`。

简单地说，当传递给重载方法和特定方法实现的参数类型不匹配时，一个给定的类型会隐式提升为另一个类型。

为了更清楚地理解类型提升是如何工作的，考虑下面的`multiply()`方法的实现:

```java
public double multiply(int a, long b) {
    return a * b;
}

public int multiply(int a, int b, int c) {
    return a * b * c;
} 
```

现在，调用带有两个`int`参数的方法将导致第二个参数被提升为`long`，因为在这种情况下，不存在带有两个`int`参数的方法的匹配实现。

让我们看一个演示类型提升的快速单元测试:

```java
@Test
public void whenCalledMultiplyAndNoMatching_thenTypePromotion() {
    assertThat(multiplier.multiply(10, 10)).isEqualTo(100.0);
}
```

相反，如果我们调用具有匹配实现的方法，类型提升就不会发生:

```java
@Test
public void whenCalledMultiplyAndMatching_thenNoTypePromotion() {
    assertThat(multiplier.multiply(10, 10, 10)).isEqualTo(1000);
}
```

下面是适用于方法重载的类型提升规则的总结:

*   `byte`可以晋升为`short, int, long, float,` 或者 `double`
*   `short`可以晋升为`int, long, float,` 或者 `double`
*   `char`可以晋升为`int, long, float,` 或者 `double`
*   `int` 可以晋升为 `long, float,` 或者 `double`
*   `long` 可以晋升为 `float` 或者 `double`
*   `float` 可以晋升为 `double`

### 2.4。静态绑定

将特定的方法调用关联到方法体的能力称为绑定。

在方法重载的情况下，绑定是在编译时静态执行的，因此它被称为静态绑定。

编译器可以在编译时通过简单地检查方法的签名来有效地设置绑定。

## 3。方法覆盖

方法覆盖允许我们在子类中为基类中定义的方法提供细粒度的实现。

虽然方法覆盖是一个强大的特性——考虑到这是使用[继承](https://web.archive.org/web/20220717181909/https://en.wikipedia.org/wiki/Inheritance_(object-oriented_programming))的逻辑结果，但是[OOP](https://web.archive.org/web/20220717181909/https://en.wikipedia.org/wiki/Object-oriented_programming)—**的最大支柱之一应该在每个用例的基础上仔细分析何时何地使用它**。

现在让我们看看如何通过创建一个简单的、基于继承的(“is-a”)关系来使用方法覆盖。

下面是基类:

```java
public class Vehicle {

    public String accelerate(long mph) {
        return "The vehicle accelerates at : " + mph + " MPH.";
    }

    public String stop() {
        return "The vehicle has stopped.";
    }

    public String run() {
        return "The vehicle is running.";
    }
}
```

这里有一个人为的子类:

```java
public class Car extends Vehicle {

    @Override
    public String accelerate(long mph) {
        return "The car accelerates at : " + mph + " MPH.";
    }
}
```

在上面的层次结构中，我们简单地覆盖了`accelerate()`方法，以便为子类型`Car.`提供更精确的实现

这里可以清楚地看到，**如果应用程序使用了`Vehicle`类的实例，那么它也可以使用`Car` 和**的实例，因为`accelerate()`方法的两个实现具有相同的签名和相同的返回类型。

让我们编写一些单元测试来检查`Vehicle`和`Car`类:

```java
@Test
public void whenCalledAccelerate_thenOneAssertion() {
    assertThat(vehicle.accelerate(100))
      .isEqualTo("The vehicle accelerates at : 100 MPH.");
}

@Test
public void whenCalledRun_thenOneAssertion() {
    assertThat(vehicle.run())
      .isEqualTo("The vehicle is running.");
}

@Test
public void whenCalledStop_thenOneAssertion() {
    assertThat(vehicle.stop())
      .isEqualTo("The vehicle has stopped.");
}

@Test
public void whenCalledAccelerate_thenOneAssertion() {
    assertThat(car.accelerate(80))
      .isEqualTo("The car accelerates at : 80 MPH.");
}

@Test
public void whenCalledRun_thenOneAssertion() {
    assertThat(car.run())
      .isEqualTo("The vehicle is running.");
}

@Test
public void whenCalledStop_thenOneAssertion() {
    assertThat(car.stop())
      .isEqualTo("The vehicle has stopped.");
} 
```

现在，让我们来看一些单元测试，展示未被覆盖的`run()`和`stop()`方法如何为`Car`和`Vehicle`返回相等的值:

```java
@Test
public void givenVehicleCarInstances_whenCalledRun_thenEqual() {
    assertThat(vehicle.run()).isEqualTo(car.run());
}

@Test
public void givenVehicleCarInstances_whenCalledStop_thenEqual() {
   assertThat(vehicle.stop()).isEqualTo(car.stop());
}
```

在我们的例子中，我们可以访问这两个类的源代码，所以我们可以清楚地看到，在一个基本的`Vehicle`实例上调用`accelerate()`方法和在一个`Car`实例上调用`accelerate()`将会为同一个参数返回不同的值。

因此，下面的测试演示了为`Car`的实例调用被覆盖的方法:

```java
@Test
public void whenCalledAccelerateWithSameArgument_thenNotEqual() {
    assertThat(vehicle.accelerate(100))
      .isNotEqualTo(car.accelerate(100));
}
```

### 3.1。类型可替代性

OOP 中的一个核心原则是类型可替代性，这与 [Liskov 替代原则(LSP)](https://web.archive.org/web/20220717181909/https://en.wikipedia.org/wiki/Liskov_substitution_principle) 密切相关。

简单地说，LSP 声明**如果应用程序使用给定的基本类型，那么它也应该使用它的任何子类型**。这样，类型可替换性得到了适当的保留。

方法覆盖的最大问题是派生类中的一些特定方法实现可能不完全符合 LSP，因此无法保持类型可替换性。

当然，让一个被覆盖的方法接受不同类型的参数并返回不同的类型也是有效的，但是要完全遵守这些规则:

*   如果基类中的方法采用给定类型的参数，被覆盖的方法应该采用相同的类型或超类型(也称为`contravariant`方法参数)
*   如果基类中的方法返回`void`，被覆盖的方法应该返回`void`
*   如果基类中的方法返回一个基元，被重写的方法应该返回相同的基元
*   如果基类中的方法返回某种类型，被覆盖的方法应该返回相同的类型或子类型(也称为`covariant`返回类型)
*   如果基类中的方法引发异常，被重写的方法必须引发相同的异常或基类异常的子类型

### 3.2。动态绑定

考虑到方法重写只能用继承来实现，其中有一个基类型和子类型的层次结构，编译器不能在编译时确定调用什么方法，因为基类和子类都定义了相同的方法。

因此，编译器需要检查对象的类型，以了解应该调用什么方法。

由于这种检查发生在运行时，方法重写是动态绑定的一个典型例子。

## 4。结论

在本教程中，我们学习了如何实现方法重载和方法覆盖，并且探索了它们有用的一些典型情况。

像往常一样，本文中显示的所有代码示例都可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220717181909/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-oop-methods)