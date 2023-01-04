# Java 继承指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-inheritance>

## 1。概述

面向对象编程的核心原则之一——继承——使我们能够重用现有的代码或扩展现有的类型。

简单来说，在 Java 中，一个类可以继承另一个类和多个接口，而一个接口可以继承其他接口。

在本文中，我们将从继承的需要开始，转到继承如何与类和接口一起工作。

然后，我们将讨论变量/方法名和访问修饰符如何影响被继承的成员。

最后，我们会看到继承一个类型意味着什么。

## 2。继承的需要

想象一下，作为一个汽车制造商，你向你的客户提供多种车型。尽管不同的汽车型号可能提供不同的功能，如天窗或防弹窗，但它们都包括通用的部件和功能，如发动机和车轮。

有意义的是**创建一个基本设计，并扩展它以创建他们的专业版本，**而不是从头开始分别设计每一款汽车模型。

以类似的方式，通过继承，我们可以创建一个具有基本特性和行为的类，并通过创建继承这个基类的类来创建它的专用版本。同样，接口可以扩展现有的接口。

我们会注意到使用了多个术语来指代被另一个类型继承的类型，特别是:

*   **基本类型也称为超类型或父类型**
*   **派生类型被称为扩展类型、子类型或子类型**

## 3。类继承

### 3.1。扩展一个类

一个类可以继承另一个类并定义附加成员。

让我们从定义一个基类`Car`开始:

```java
public class Car {
    int wheels;
    String model;
    void start() {
        // Check essential parts
    }
}
```

类`ArmoredCar`可以通过**在其声明**中使用关键字`extends`来继承`Car`类的成员:

```java
public class ArmoredCar extends Car {
    int bulletProofWindows;
    void remoteStartCar() {
	// this vehicle can be started by using a remote control
    }
}
```

我们现在可以说,`ArmoredCar`类是`Car,` 的子类，而后者是 `ArmoredCar.`的超类

**Java 中的类支持单继承**；`ArmoredCar`类不能扩展多个类。

还要注意，在没有关键字`extends`的情况下，一个类隐式地继承了类`java.lang.Object`。

**子类从超类继承非静态的`protected`和`public`成员。**此外，如果两个类在同一个包中，那么具有`default` ( `package-private)`访问权限的成员将被继承。

另一方面，类的`private`和`static`成员不会被继承。

### 3.2。从子类中访问父成员

要访问继承的属性或方法，我们可以简单地直接使用它们:

```java
public class ArmoredCar extends Car {
    public String registerModel() {
        return model;
    }
}
```

注意，我们不需要引用超类来访问它的成员。

## 4。接口继承

### 4.1。实现多个接口

虽然类只能继承一个类，但是它们可以实现多个接口。

想象一下，一个超级间谍需要我们在上一节中定义的`ArmoredCar`。于是`Car`制造公司想到了增加飞行和漂浮功能:

```java
public interface Floatable {
    void floatOnWater();
}
```

```java
public interface Flyable {
    void fly();
}
```

```java
public class ArmoredCar extends Car implements Floatable, Flyable{
    public void floatOnWater() {
        System.out.println("I can float!");
    }

    public void fly() {
        System.out.println("I can fly!");
    }
}
```

在上面的例子中，我们注意到使用了关键字`implements`来继承一个接口。

### 4.2。多重继承的问题

Java 允许使用接口的多重继承。

在 Java 7 之前，这不是问题。接口只能定义`abstract`方法，即没有任何实现的方法。因此，如果一个类用相同的方法签名实现了多个接口，这不是问题。实现类最终只有一个方法要实现。

让我们看看这个简单的等式是如何随着 Java 8 在接口中引入`default`方法而改变的。

**从 Java 8 开始，接口可以选择为其方法**定义默认实现(一个接口仍然可以定义`abstract`方法)。这意味着，如果一个类实现多个接口，这些接口用相同的签名定义方法，那么子类将继承不同的实现。这听起来很复杂，是不允许的。

Java 不允许继承在不同接口中定义的相同方法的多个实现。

这里有一个例子:

```java
public interface Floatable {
    default void repair() {
    	System.out.println("Repairing Floatable object");	
    }
}
```

```java
public interface Flyable {
    default void repair() {
    	System.out.println("Repairing Flyable object");	
    }
}
```

```java
public class ArmoredCar extends Car implements Floatable, Flyable {
    // this won't compile
}
```

如果我们确实想实现这两个接口，我们必须覆盖`repair()`方法。

如果前面例子中的接口定义了同名的变量，比如说`duration`，我们不能在变量名前面加上接口名来访问它们:

```java
public interface Floatable {
    int duration = 10;
}
```

```java
public interface Flyable {
    int duration = 20;
}
```

```java
public class ArmoredCar extends Car implements Floatable, Flyable {

    public void aMethod() {
    	System.out.println(duration); // won't compile
    	System.out.println(Floatable.duration); // outputs 10
    	System.out.println(Flyable.duration); // outputs 20
    }
}
```

### 4.3。扩展其他接口的接口

一个接口可以扩展多个接口。这里有一个例子:

```java
public interface Floatable {
    void floatOnWater();
}
```

```java
interface interface Flyable {
    void fly();
}
```

```java
public interface SpaceTraveller extends Floatable, Flyable {
    void remoteControl();
}
```

一个接口通过使用关键字`extends`继承其他接口。类使用关键字`implements`来继承接口。

## 5。继承类型

当一个类继承另一个类或接口时，除了继承它们的成员，它还继承它们的类型。这也适用于继承其他接口的接口。

这是一个非常强大的概念，它允许开发者**编程到一个接口(基类或接口)**，而不是编程到他们的实现。

例如，假设有这样一种情况，某个组织维护着其员工拥有的汽车列表。当然，所有员工可能拥有不同的汽车型号。那么我们如何引用不同的汽车实例呢？解决方案如下:

```java
public class Employee {
    private String name;
    private Car car;

    // standard constructor
}
```

因为`Car`的所有派生类都继承了类型`Car`，所以可以通过使用类`Car`的变量来引用派生类实例:

```java
Employee e1 = new Employee("Shreya", new ArmoredCar());
Employee e2 = new Employee("Paul", new SpaceCar());
Employee e3 = new Employee("Pavni", new BMW());
```

## 6.隐藏类成员

### 6.1。隐藏实例成员

如果**超类和子类都定义了一个同名的变量或方法**，会发生什么？不用担心；我们仍然可以访问它们。然而，我们必须通过在变量或方法前面加上关键字`this`或`super`来让 Java 明白我们的意图。

`this`关键字指的是使用它的实例。`super`关键字(显而易见)指的是父类实例:

```java
public class ArmoredCar extends Car {
    private String model;
    public String getAValue() {
    	return super.model;   // returns value of model defined in base class Car
    	// return this.model;   // will return value of model defined in ArmoredCar
    	// return model;   // will return value of model defined in ArmoredCar
    }
}
```

许多开发人员使用`this`和`super`关键字来明确说明他们所指的是哪个变量或方法。然而，对所有成员使用它们会使我们的代码看起来混乱。

### 6.2。隐藏静态成员

当我们的基类和子类用相同的名字定义静态变量和方法时**会发生什么？我们可以像访问实例变量一样，在派生类中访问基类的成员吗？**

让我们用一个例子来找出答案:

```java
public class Car {
    public static String msg() {
        return "Car";
    }
}
```

```java
public class ArmoredCar extends Car {
    public static String msg() {
        return super.msg(); // this won't compile.
    }
}
```

不，我们不能。静态成员属于一个类，而不属于实例。所以我们不能在`msg()`中使用非静态的`super`关键字。

由于静态成员属于一个类，我们可以如下修改前面的调用:

```java
return Car.msg();
```

考虑下面的例子，其中基类和派生类都用相同的签名定义了一个静态方法`msg()`:

```java
public class Car {
    public static String msg() {
        return "Car";
    }
}
```

```java
public class ArmoredCar extends Car {
    public static String msg() {
        return "ArmoredCar";
    }
}
```

我们可以这样称呼它们:

```java
Car first = new ArmoredCar();
ArmoredCar second = new ArmoredCar();
```

对于前面的代码，`first.msg()`将输出“汽车`“`,`second.msg()`将输出“装甲汽车”。被调用的静态消息取决于用于引用`ArmoredCar`实例的变量的类型。

## 7。结论

在本文中，我们讨论了 Java 语言的一个核心方面——继承。

我们看到了 Java 如何支持类的单一继承和接口的多重继承，并讨论了该机制在语言中如何工作的复杂性。

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20220930150703/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-oop-inheritance)