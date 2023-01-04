# Java 接口中的静态和默认方法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-static-default-methods>

## 1。概述

Java 8 带来了一些全新的特性，包括 [lambda 表达式](https://web.archive.org/web/20221129002420/https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html)、[函数接口](/web/20221129002420/https://www.baeldung.com/java-8-functional-interfaces)、[方法引用](/web/20221129002420/https://www.baeldung.com/java-8-double-colon-operator)、[流](/web/20221129002420/https://www.baeldung.com/java-8-streams)、[可选](/web/20221129002420/https://www.baeldung.com/java-optional)，以及接口中的`static`和`default`方法。

我们已经在[的另一篇文章](/web/20221129002420/https://www.baeldung.com/java-8-new-features)中讨论了其中的一些特性。尽管如此，接口中的`static`和`default`方法本身值得深入研究。

在本教程中，我们将学习**如何在接口**中使用`static`和`default`方法，并讨论它们可能有用的一些情况。

## 延伸阅读:

## [Java 接口中的私有方法](/web/20221129002420/https://www.baeldung.com/java-interface-private-methods)

Learn how to define private methods within an interface and how we can use them from both static and non-static contexts.[Read more](/web/20221129002420/https://www.baeldung.com/java-interface-private-methods) →

## [在 Java 中使用接口与抽象类](/web/20221129002420/https://www.baeldung.com/java-interface-vs-abstract-class)

Learn when to use an interface and when to use an abstract class in Java.[Read more](/web/20221129002420/https://www.baeldung.com/java-interface-vs-abstract-class) →

## [Java 中的静态关键字指南](/web/20221129002420/https://www.baeldung.com/java-static)

Learn about Java static fields, static methods, static blocks and static inner classes.[Read more](/web/20221129002420/https://www.baeldung.com/java-static) →

## 2。为什么接口需要默认方法

与常规接口方法一样，**默认方法是隐式公共的；**没有必要指定`public`修饰符。

与常规的接口方法不同，我们**在方法签名**的开头用`default` 关键字声明它们，它们**提供一个实现**。

让我们看一个简单的例子:

```
public interface MyInterface {

    // regular interface methods

    default void defaultMethod() {
        // default method implementation
    }
}
```

Java 8 发行版包含`default`方法的原因很明显。

在基于抽象的典型设计中，一个接口有一个或多个实现，如果一个或多个方法被添加到接口中，所有的实现都将被迫实现它们。否则，设计就会失败。

默认接口方法是处理这个问题的有效方法。它们**允许我们向接口添加新方法，这些方法在实现中自动可用**。因此，我们不需要修改实现类。

通过这种方式，**向后兼容性被巧妙地保留了下来**,而不必重构实现者。

## 3。运行中的默认接口方法

为了更好地理解`default`接口方法的功能，让我们创建一个简单的例子。

假设我们有一个简单的接口和一个实现。可能还有更多，但让我们保持简单:

```
public interface Vehicle {

    String getBrand();

    String speedUp();

    String slowDown();

    default String turnAlarmOn() {
        return "Turning the vehicle alarm on.";
    }

    default String turnAlarmOff() {
        return "Turning the vehicle alarm off.";
    }
}
```

现在让我们编写实现类:

```
public class Car implements Vehicle {

    private String brand;

    // constructors/getters

    @Override
    public String getBrand() {
        return brand;
    }

    @Override
    public String speedUp() {
        return "The car is speeding up.";
    }

    @Override
    public String slowDown() {
        return "The car is slowing down.";
    }
} 
```

最后，让我们定义一个典型的`main`类，它创建一个 `Car`的实例并调用它的方法:

```
public static void main(String[] args) { 
    Vehicle car = new Car("BMW");
    System.out.println(car.getBrand());
    System.out.println(car.speedUp());
    System.out.println(car.slowDown());
    System.out.println(car.turnAlarmOn());
    System.out.println(car.turnAlarmOff());
}
```

请注意我们的`Vehicle`接口中的`default`方法、`turnAlarmOn()`和`turnAlarmOff(),`是如何在`Car`类中自动可用的**。**

此外，如果在某个时候我们决定向`Vehicle`接口添加更多的`default`方法，应用程序仍将继续工作，我们不必强迫类提供新方法的实现。

接口默认方法最常见的用途是**在不分解实现类的情况下，为给定类型提供额外的功能。**

此外，我们可以使用它们来为现有的抽象方法提供额外的功能:

```
public interface Vehicle {

    // additional interface methods 

    double getSpeed();

    default double getSpeedInKMH(double speed) {
       // conversion      
    }
}
```

## 4。多接口继承规则

默认接口方法是一个非常好的特性，但是有一些警告值得一提。因为 Java 允许类实现多个接口，所以重要的是要知道当一个类实现几个定义相同`default`方法的接口时**会发生什么。**

为了更好地理解这个场景，让我们定义一个新的 `Alarm`接口并重构`Car`类:

```
public interface Alarm {

    default String turnAlarmOn() {
        return "Turning the alarm on.";
    }

    default String turnAlarmOff() {
        return "Turning the alarm off.";
    }
}
```

这个新接口定义了自己的一组`default`方法，`Car`类将实现`Vehicle`和`Alarm`:

```
public class Car implements Vehicle, Alarm {
    // ...
}
```

在这种情况下，**代码根本无法编译，因为多重接口继承导致了冲突**(也称为[钻石问题](https://web.archive.org/web/20221129002420/https://en.wikipedia.org/wiki/Multiple_inheritance))。`Car`类将继承两组`default`方法。那么我们应该打电话给哪些人呢？

**为了解决这种模糊性，我们必须明确地提供方法的实现:**

```
@Override
public String turnAlarmOn() {
    // custom implementation
}

@Override
public String turnAlarmOff() {
    // custom implementation
}
```

我们也可以**让我们的类使用其中一个接口**的`default`方法。

让我们看一个使用来自`Vehicle`接口的`default`方法的例子:

```
@Override
public String turnAlarmOn() {
    return Vehicle.super.turnAlarmOn();
}

@Override
public String turnAlarmOff() {
    return Vehicle.super.turnAlarmOff();
} 
```

类似地，我们可以让类使用在`Alarm`接口中定义的`default`方法:

```
@Override
public String turnAlarmOn() {
    return Alarm.super.turnAlarmOn();
}

@Override
public String turnAlarmOff() {
    return Alarm.super.turnAlarmOff();
} 
```

甚至有可能让`Car`类使用两套默认方法:

```
@Override
public String turnAlarmOn() {
    return Vehicle.super.turnAlarmOn() + " " + Alarm.super.turnAlarmOn();
}

@Override
public String turnAlarmOff() {
    return Vehicle.super.turnAlarmOff() + " " + Alarm.super.turnAlarmOff();
} 
```

## 5。静态接口方法

除了在接口中声明`default`方法， **Java 8 还允许我们在接口**中定义和实现`static`方法。

由于`static`方法不属于特定的对象，它们不是实现接口的类的 API 的一部分；因此，必须使用方法名前面的接口名来调用它们**。**

为了理解`static`方法在接口中是如何工作的，让我们重构`Vehicle`接口并向其添加一个`static`实用方法:

```
public interface Vehicle {

    // regular / default interface methods

    static int getHorsePower(int rpm, int torque) {
        return (rpm * torque) / 5252;
    }
} 
```

**在一个接口中定义一个`static`方法等同于在一个类中定义一个方法。**此外，一个`static`方法可以在其他`static`和`default`方法中被调用。

假设我们想要计算给定车辆发动机的[马力](https://web.archive.org/web/20221129002420/https://en.wikipedia.org/wiki/Horsepower)。我们只是调用了`getHorsePower()`方法:

```
Vehicle.getHorsePower(2500, 480)); 
```

接口方法背后的想法是提供一个简单的机制，允许我们通过把相关的方法放在一个地方来增加设计的 T2 内聚 T4 的程度，而不需要创建一个对象。

抽象类也可以做到这一点。主要区别在于**抽象类可以有构造函数、状态和行为**。

此外，接口中的静态方法使得对相关的实用程序方法进行分组成为可能，而不必创建只是静态方法占位符的人工实用程序类。

## 6。结论

在本文中，我们深入探讨了 Java 8 中`static`和`default`接口方法的使用。乍一看，这个特性可能有点草率，特别是从面向对象的纯粹主义者的角度来看。理想情况下，接口不应该封装行为，我们应该只使用它们来定义某种类型的公共 API。

然而，当涉及到维护与现有代码的向后兼容性时，`static`和`default`方法是一个很好的权衡。

像往常一样，本文中显示的所有代码示例都可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221129002420/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-oop-modifiers)