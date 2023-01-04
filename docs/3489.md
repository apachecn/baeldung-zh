# Java 中面向对象的编程概念

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-oop>

## 1.概观

在本文中，我们将研究 Java 中面向对象编程(OOP)的概念。我们将讨论**类、对象、抽象、封装、继承和多态**。

## 2.班级

类是所有对象的起点，我们可以把它们看作是创建对象的模板。一个类通常包含成员字段、成员方法和一个特殊的构造函数方法。

我们将使用构造函数来创建该类的对象:

```
public class Car {

    // member fields
    private String type;
    private String model;
    private String color;
    private int speed;

    // constructor
    public Car(String type, String model, String color) {
        this.type = type;
        this.model = model;
        this.color = color;
    }

    // member methods
    public int increaseSpeed(int increment) {
        this.speed = this.speed + increment;
        return this.speed;
    }

    // ...
}
```

请注意，一个类可能有多个构造函数。我们可以在我们的 [classes 文章中读到更多关于这些类的内容。](/web/20221126233949/https://www.baeldung.com/java-classes-objects#classes)

## 3.目标

对象是从类创建的，称为类的实例。我们使用类的构造函数从类中创建对象:

```
Car veyron = new Car("Bugatti", "Veyron", "crimson");
Car corvette = new Car("Chevrolet", "Corvette", "black"); 
```

这里，我们已经创建了类`Car.`的两个实例，在我们的 [objects 文章中可以读到更多关于它们的内容。](/web/20221126233949/https://www.baeldung.com/java-classes-objects#objects)

## 4.抽象

抽象隐藏了实现的复杂性，暴露了更简单的接口。

如果我们考虑一台典型的计算机，我们只能看到外部接口，这是与它交互的最基本的接口，而内部芯片和电路对用户来说是隐藏的。

在 OOP 中，抽象意味着隐藏程序复杂的实现细节，只暴露使用实现所需的 API。在 Java 中，我们通过使用接口和抽象类来实现抽象。

我们可以在我们的[抽象类](/web/20221126233949/https://www.baeldung.com/java-abstract-class)和[接口](/web/20221126233949/https://www.baeldung.com/java-interfaces)文章中读到更多关于抽象的内容。

## 5.包装

**封装是对 API** 的消费者隐藏对象的状态或内部表示，并提供绑定到对象的可公开访问的方法以进行读写访问。这允许隐藏特定信息并控制对内部实现的访问。

例如，一个类中的成员字段对其他类是隐藏的，可以使用成员方法来访问它们。一种方法是使所有数据字段`private`只能通过使用`public`成员方法来访问:

```
public class Car {

    // ...
    private int speed;

    public int getSpeed() {
        return color;
    }

    public void setSpeed(int speed) {
        this.speed = speed;
    }
    // ...
}
```

这里，字段`speed`是用`private `访问修饰符封装的，只能用`public getSpeed()`和`setSpeed() `方法访问。我们可以在我们的[访问修饰符](/web/20221126233949/https://www.baeldung.com/java-access-modifiers)文章中读到更多关于访问修饰符的内容。

## 6.遗产

继承是一种机制，允许一个类通过继承另一个类来获得该类的所有属性。我们称继承类为子类，被继承的类为超类或父类。

在 Java 中，我们通过扩展父类来实现这一点。因此，子类从父类获得所有属性:

```
public class Car extends Vehicle { 
    //...
}
```

当我们扩展一个类时，我们形成了一个[关系](/web/20221126233949/https://www.baeldung.com/java-inheritance-composition)。**`Car`是-A `Vehicle`。**所以，它具备了`Vehicle`的所有特征。

我们可能会问这个问题，**为什么我们需要继承**？为了回答这个问题，让我们考虑一个制造不同类型车辆的车辆制造商，例如轿车、公共汽车、有轨电车和卡车。

为了简化工作，我们可以将所有车辆类型的公共特性和属性捆绑到一个模块中(对于 Java 来说是一个类)。我们可以让单个类型继承并重用这些属性:

```
public class Vehicle {
    private int wheels;
    private String model;
    public void start() {
        // the process of starting the vehicle
    }

    public void stop() {
        // process to stop the vehicle
    }

    public void honk() { 
        // produces a default honk 
    }

}
```

车辆类型`Car` 现在将从父类`Vehicle`继承:

```
public class Car extends Vehicle {
    private int numberOfGears;

    public void openDoors() {
        // process to open the doors
    }
}
```

Java 支持单一继承和多级继承。这意味着一个类不能直接从多个类扩展，但是它可以使用层次结构:

```
public class ArmoredCar extends Car {
    private boolean bulletProofWindows;

    public void remoteStartCar() {
        // this vehicle can be started by using a remote control
    }
}
```

这里，`ArmouredCar`延伸了`Car`，`Car`延伸了`Vehicle`。因此，`ArmouredCar`继承了`Car`和`Vehicle`的属性。

当我们从父类继承时，开发人员也可以覆盖父类的方法实现。**这就是所谓的[法压倒](/web/20221126233949/https://www.baeldung.com/java-method-overload-override#method-overriding)。**

在我们上面的`Vehicle`类的例子中，有一个`honk()`方法。扩展了`Vehicle `类的`Car `类可以覆盖这个方法，并以它想要的方式实现鸣声:

```
public class Car extends Vehicle {  
    //...

    @Override
    public void honk() { 
        // produces car-specific honk 
    }
 }
```

请注意，这也称为运行时多态性，下一节将对此进行解释。我们可以在我们的 [Java 继承](/web/20221126233949/https://www.baeldung.com/java-inheritance)和[继承和组合](/web/20221126233949/https://www.baeldung.com/java-inheritance-composition)文章中读到更多关于继承的内容。

## 7.多态性

[多态性](/web/20221126233949/https://www.baeldung.com/cs/polymorphism)是一种 OOP 语言根据其输入类型以不同方式处理数据的能力。在 Java 中，这可以是具有不同方法签名和执行不同功能的相同方法名:

```
public class TextFile extends GenericFile {
    //...

    public String read() {
        return this.getContent()
          .toString();
    }

    public String read(int limit) {
        return this.getContent()
          .toString()
          .substring(0, limit);
    }

    public String read(int start, int stop) {
        return this.getContent()
          .toString()
          .substring(start, stop);
    }
}
```

在这个例子中，我们可以看到方法`read() `有三种不同的形式，具有不同的功能。**这种类型的多态性是静态或编译时多态性，也称为[方法重载](/web/20221126233949/https://www.baeldung.com/java-method-overload-override#method-overloading)。**

还有运行时或**动态多态性，其中子类覆盖父类的方法**:

```
public class GenericFile {
    private String name;

    //...

    public String getFileInfo() {
        return "Generic File Impl";
    }
}
```

子类可以扩展`GenericFile `类并覆盖`getFileInfo() `方法:

```
public class ImageFile extends GenericFile {
    private int height;
    private int width;

    //... getters and setters

    public String getFileInfo() {
        return "Image File Impl";
    }
}
```

在我们的[Java 中的多态性](/web/20221126233949/https://www.baeldung.com/java-polymorphism)文章中阅读更多关于多态性的内容。

## 8.结论

在本文中，我们学习了面向对象和 Java 的基本概念。

Github 上的[提供了本文中的代码示例。](https://web.archive.org/web/20221126233949/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-oop-others)