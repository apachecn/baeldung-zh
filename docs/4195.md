# Java 中的具体类

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-concrete-class>

## 1。简介

在这个快速指南中，**我们将讨论 Java** 中的术语“具体类”。

首先，我们来定义这个术语。然后，我们将看到它与接口和抽象类有什么不同。

## 2。什么是具体类？

**具体类是我们可以使用`new`关键字**创建实例的类。

换句话说，这是其蓝图的**完全实现。一个具体的类完成了。**

例如，想象一个`Car `类:

```
public class Car {
    public String honk() {
        return "beep!";
    }

    public String drive() {
        return "vroom";
    }
}
```

因为它的所有方法都是实现的，所以我们称它为具体类，我们可以实例化它:

```
Car car = new Car();
```

JDK 中具体类的一些例子有 **`HashMap`、`HashSet`、`ArrayList`和`LinkedList`。**

## 3。Java 抽象与具体类

然而，并不是所有的 Java 类型都实现了它们所有的方法。这种灵活性，也称为`abstraction`，允许我们从更一般的角度考虑我们试图建模的领域。

在 Java 中，我们可以使用接口和抽象类来实现抽象。

让我们通过与其他类的比较来更好地了解具体类。

### 3.1。接口

接口是一个类的蓝图。或者，换句话说，它是未实现的方法签名的集合:

```
interface Driveable {
    void honk();
    void drive();
}
```

**注意，它使用了`interface`关键字，而不是`class.`**

因为`Driveable `有未实现的方法，我们不能用`new `关键字实例化它。

但是，**像`Car `这样的具体类可以实现这些方法。**

JDK 提供了许多接口，如**`Map``List``Set`。**

### 3.2。抽象类

**抽象类是具有未实现方法的类，**尽管它实际上可以同时具有这两种方法:

```
public abstract class Vehicle {
    public abstract String honk();

    public String drive() {
        return "zoom";
    }
}
```

**注意，我们用关键字`abstract`标记抽象类。**

同样，由于`Vehicle` 有一个未实现的方法`honk`，我们将不能使用`new `关键字。

来自 JDK 的抽象类的一些例子是 **`AbstractMap`和`AbstractList`。**

### 3.3。具体类别

相比之下，**具体类没有任何未实现的方法。**不管实现是否被继承，只要每个方法都有一个实现，这个类就是具体的。

具体的类可以像我们前面的例子一样简单。它们还可以实现接口和扩展抽象类:

```
public class FancyCar extends Vehicle implements Driveable {
    public String honk() { 
        return "beep";
    }
}
```

`FancyCar `类为`honk`提供了一个实现，它从`Vehicle.`继承了`drive`的实现

**因此，它没有未实现的方法**。因此，我们可以用`new`关键字创建一个`FancyCar`类实例。

```
FancyCar car = new FancyCar();
```

或者，简单地说，所有非抽象的类，我们都可以称之为具体类。

## 4。总结

在这个简短的教程中，我们学习了具体的类及其规范。

此外，我们还展示了接口、具体类和抽象类之间的区别。