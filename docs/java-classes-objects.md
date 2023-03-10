# Java 类和对象

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-classes-objects>

## 1。概述

在这个快速教程中，我们将研究 Java 编程语言的两个基本构件——类和对象。它们是面向对象编程(OOP)的基本概念，我们用它来建模现实生活中的实体。

在 OOP 中，**类是对象的蓝图或模板。我们用它们来描述实体的类型。**

另一方面，对象是有生命的实体，由类创建。它们在其领域中包含某些状态，并通过它们的方法呈现某些行为。

## 2。类别

简单地说，一个类代表一个对象定义或类型。在 Java 中，类可以包含字段、构造函数和方法。

让我们看一个使用简单 Java 类表示`Car`的例子:

```java
class Car {

    // fields
    String type;
    String model;
    String color;
    int speed;

    // constructor
    Car(String type, String model, String color) {
        this.type = type;
        this.model = model;
        this.color = color;
    }

    // methods
    int increaseSpeed(int increment) {
        this.speed = this.speed + increment;
        return this.speed;
    }

    // ...
} 
```

这个 Java 类通常代表一辆汽车。我们可以从这个类创建任何类型的汽车。我们使用字段保存状态，并使用构造函数从该类创建对象。

默认情况下，每个 Java 类都有一个空的构造函数。如果我们不像上面那样提供特定的实现，我们就使用它。下面是默认构造函数如何寻找我们的`Car`类:

```java
Car(){} 
```

这个构造函数简单地用默认值初始化对象的所有字段。字符串被初始化为`null`，整数被初始化为零。

现在，我们的类有一个特定的构造函数，因为我们希望我们的对象在创建时就定义了它们的字段:

```java
Car(String type, String model) {
    // ...
} 
```

总而言之，我们写了一个定义汽车的类。它的属性由包含该类对象状态的字段描述，它的行为使用方法描述。

## 3。对象

当类在编译时被翻译时，**对象在运行时从类中创建**。

一个类的对象被称为实例，我们用构造函数创建并初始化它们:

```java
Car focus = new Car("Ford", "Focus", "red");
Car auris = new Car("Toyota", "Auris", "blue");
Car golf = new Car("Volkswagen", "Golf", "green"); 
```

现在，我们已经创建了不同的`Car`对象，全部来自同一个类。**这就是它的全部，在一个地方定义蓝图，然后，在许多地方多次重用它。**

到目前为止，我们有三个`Car`物体，它们都停着，因为它们的速度为零。我们可以通过调用我们的`increaseSpeed`方法来改变这一点:

```java
focus.increaseSpeed(10);
auris.increaseSpeed(20);
golf.increaseSpeed(30); 
```

现在，我们改变了汽车的状态——它们都以不同的速度行驶。

此外，我们可以并且应该定义对我们的类、它的构造函数、字段和方法的访问控制。我们可以通过使用访问修饰符来做到这一点，我们将在下一节中看到。

## 4。访问修饰符

在前面的例子中，为了简化代码，我们省略了访问修饰符。通过这样做，我们实际上使用了一个默认的包私有修饰符。该修饰符允许从同一个包中的任何其他类访问该类。

通常，我们会对构造函数使用一个`public`修饰符来允许来自所有其他对象的访问:

```java
public Car(String type, String model, String color) {
    // ...
} 
```

我们类中的每个字段和方法都应该通过特定的修饰符定义访问控制。**类通常有`public`修饰符，但是我们倾向于保留我们的字段`private`。**

字段保存对象的状态，因此我们希望控制对该状态的访问。我们可以保留其中一些`private`，其他的`public`。我们通过称为 getters 和 setters 的特定方法来实现这一点。

让我们看一下我们的具有完全指定的访问控制的类:

```java
public class Car {
    private String type;
    // ...

    public Car(String type, String model, String color) {
       // ...
    }

    public String getColor() {
        return color;
    }

    public void setColor(String color) {
        this.color = color;
    }

    public int getSpeed() {
        return speed;
    }

    // ...
} 
```

**我们的类被标记为`public`，这意味着我们可以在任何包中使用它。**此外，构造函数是`public`，这意味着我们可以在任何其他对象中从这个类创建一个对象。

**我们的字段被标记为`private`，这意味着它们不能从我们的对象**直接访问，但是我们通过 getters 和 setters 提供对它们的访问。

`type`和`model`字段没有 getters 和 setters，因为它们保存我们对象的内部数据。我们只能在初始化期间通过构造函数来定义它们。

此外，`color`可以被访问和更改，而`speed`只能被访问，不能被更改。我们通过专门的`public`方法`increaseSpeed()`和`decreaseSpeed()`进行速度调整。

换句话说，我们使用访问控制来封装对象的状态。

## 5。结论

在本文中，我们介绍了 Java 语言的两个基本元素:类和对象，并展示了如何以及为什么使用它们。我们还介绍了访问控制的基础知识，并演示了它的用法。

为了学习 Java 语言的其他概念，我们建议下一步阅读关于[继承](/web/20220625180138/https://www.baeldung.com/java-inheritance)、[超级关键字](/web/20220625180138/https://www.baeldung.com/java-super)和[抽象类](/web/20220625180138/https://www.baeldung.com/java-abstract-class)的内容。

这个例子的完整源代码在 GitHub 上可以找到。