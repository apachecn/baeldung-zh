# 在 Java 中使用接口与抽象类

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-interface-vs-abstract-class>

## 1.介绍

抽象是面向对象编程的关键特性之一。它允许我们**通过简单的接口提供功能来隐藏实现的复杂性。在 Java 中，我们通过使用[接口](/web/20220625080345/https://www.baeldung.com/java-interfaces)或[抽象类](/web/20220625080345/https://www.baeldung.com/java-abstract-class)来实现抽象。**

在本文中，我们将讨论在设计应用程序时，何时使用接口，何时使用抽象类。此外，它们之间的主要区别以及根据我们要实现的目标选择哪一个。

## 2.类与接口

首先，让我们看看普通的具体类和接口之间的区别。

类是用户定义的类型，充当对象创建的蓝图。它可以有属性和方法，分别表示对象的状态和行为。

接口也是用户定义的类型，在语法上类似于类。它可以有一个字段常量和方法签名的集合，将被接口实现类覆盖。

除了这些， [Java 8 新特性](/web/20220625080345/https://www.baeldung.com/java-8-new-features)支持接口中的[静态和默认方法](/web/20220625080345/https://www.baeldung.com/java-static-default-methods)以支持向后兼容。如果接口中的方法不是`static`或`default`并且都是`public`，那么它们就是隐式抽象的。

但是，从 Java 9 开始，我们也可以在接口中添加[私有方法](/web/20220625080345/https://www.baeldung.com/new-java-9#3-interface-private-method)。

## 3.接口与抽象类

抽象类只不过是使用`abstract`关键字声明的类。它还允许我们使用`abstract`关键字(抽象方法)声明方法签名，并强制其子类实现所有声明的方法。假设一个类有一个抽象的方法，那么这个类本身一定是抽象的。

抽象类对字段和方法修饰符没有限制，而在接口中，默认情况下它们都是公共的。我们可以在抽象类中拥有实例和静态初始化块，但是我们永远不能在接口中拥有它们。抽象类也可能有构造函数，这些构造函数将在子对象的实例化过程中执行。

Java 8 引入了[函数接口](/web/20220625080345/https://www.baeldung.com/java-8-functional-interfaces)，这个接口的限制是只能声明一个抽象方法。除了静态和默认方法之外，任何具有单一抽象方法的接口都被认为是函数接口。我们可以使用这个特性来限制要声明的抽象方法的数量。而在抽象类中，我们永远不能对抽象方法声明的数量有这种限制。

抽象类在某些方面类似于接口:

*   **我们不能实例化它们中的任何一个。**也就是说，我们不能直接使用语句`new TypeName()`来实例化一个对象。如果我们使用前面提到的语句，我们必须使用一个[匿名类](/web/20220625080345/https://www.baeldung.com/java-anonymous-classes)覆盖所有的方法
*   它们都可能包含一组声明和定义的方法，有或没有它们的实现。即静态&默认方法(定义在一个接口中)，实例方法(定义在抽象类中)，抽象方法(声明在两者中)

## 4.何时使用界面

让我们来看一些应该使用接口的场景:

*   如果问题需要使用多个继承来解决，并且由不同的类层次结构组成
*   当不相关的类实现我们的接口时。例如， [Comparable](/web/20220625080345/https://www.baeldung.com/java-comparator-comparable#comparable) 提供了`compareTo()`方法，可以重写该方法来比较两个对象
*   当应用程序功能必须被定义为契约，但不关心谁实现行为时。即第三方供应商需要完全实现它

**当我们的问题做出陈述“A 有能力[这样做]”**时，考虑使用接口。例如，“Clonable 能够克隆一个对象”，“Drawable 能够绘制一个形状”等。

让我们考虑一个使用接口的例子:

```
public interface Sender {
    void send(File fileToBeSent);
}
```

```
public class ImageSender implements Sender {
    @Override
    public void send(File fileToBeSent) {
        // image sending implementation code.
    }
}
```

这里，`Sender `是带有方法`send()`的接口。因此，“发送者能够发送文件”我们将其实现为一个接口。`ImageSender` 实现向目标发送图像的接口。我们可以进一步使用上面的接口实现`VideoSender`、`DocumentSender` 来完成各种工作。

考虑一个使用上述接口及其实现类的单元测试案例:

```
@Test
void givenImageUploaded_whenButtonClicked_thenSendImage() { 

    File imageFile = new File(IMAGE_FILE_PATH);

    Sender sender = new ImageSender();
    sender.send(imageFile);
}
```

## 5.何时使用抽象类

现在，让我们看一些应该使用抽象类的场景:

*   当试图在代码中使用继承概念时(在许多相关类之间共享代码)，通过提供子类覆盖的公共基类方法
*   如果我们已经指定了需求并且只有部分实现细节
*   而扩展抽象类的类有几个公共字段或方法(需要非公共修饰符)
*   如果想用非最终或非静态的方法来修改对象的状态

当我们的问题证明“A 是 B”时，考虑使用抽象类和继承。比如“狗是动物”、“兰博基尼是汽车”等。

让我们看一个使用抽象类的例子:

```
public abstract class Vehicle {

    protected abstract void start();
    protected abstract void stop();
    protected abstract void drive();
    protected abstract void changeGear();
    protected abstract void reverse();

    // standard getters and setters
} 
```

```
public class Car extends Vehicle {

    @Override
    protected void start() {
        // code implementation details on starting a car.
    }

    @Override
    protected void stop() {
        // code implementation details on stopping a car.
    }

    @Override
    protected void drive() {
        // code implementation details on start driving a car.
    }

    @Override
    protected void changeGear() {
        // code implementation details on changing the car gear.
    }

    @Override
    protected void reverse() {
        // code implementation details on reverse driving a car.
    }
}
```

在上面的代码中，`Vehicle`类和其他抽象方法一起被定义为抽象的。它提供任何真实世界车辆的通用操作，并且还具有几个通用功能。扩展了`Vehicle`类的`Car`类通过提供汽车的实现细节(“汽车是一辆汽车”)覆盖了所有的方法。

因此，我们将`Vehicle`类定义为抽象类，其中的功能可以由任何单独的真实交通工具实现，如汽车和公共汽车。例如，在现实世界中，启动汽车和公共汽车永远不会相同(它们各自需要不同的实现细节)。

现在，让我们考虑一个使用上述代码的简单单元测试:

```
@Test
void givenVehicle_whenNeedToDrive_thenStart() {

    Vehicle car = new Car("BMW");

    car.start();
    car.drive();
    car.changeGear();
    car.stop();
}
```

## 6.结论

本文讨论了接口和抽象类的概述以及它们之间的主要区别。此外，我们还研究了在我们的工作中何时使用它们来完成编写灵活和干净的代码。

本文给出的例子的完整源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220625080345/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-oop-patterns)