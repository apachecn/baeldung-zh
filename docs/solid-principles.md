# 坚实原则的坚实指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/solid-principles>

## 1。概述

在本教程中，我们将讨论面向对象设计的坚实原则。

首先，我们将从**探索它们产生的原因以及为什么我们在设计软件时应该考虑它们**开始。然后我们将列出每个原则和一些示例代码。

## 延伸阅读:

## [Java 中的单一责任原则](/web/20220929174444/https://www.baeldung.com/java-single-responsibility-principle)

A quick and practical guide to the Single Responsibility Principle in Java[Read more](/web/20220929174444/https://www.baeldung.com/java-single-responsibility-principle) →

## [Java 中的开/闭原理](/web/20220929174444/https://www.baeldung.com/java-open-closed-principle)

Explore the Open/Closed Principle (OCP) as one of the SOLID principles of object-oriented programming in Java.[Read more](/web/20220929174444/https://www.baeldung.com/java-open-closed-principle) →

## [Java 中的利斯科夫替换原理](/web/20220929174444/https://www.baeldung.com/java-liskov-substitution-principle)

The L in SOLID, the Liskov Substitution Principle helps structure object oriented design. We also explore how it supports the Open/Closed Principle.[Read more](/web/20220929174444/https://www.baeldung.com/java-liskov-substitution-principle) →

## 2。坚实原则的理由

Robert C. Martin 在他 2000 年的论文[“设计原则和设计模式”中介绍了这些坚实的原则](https://web.archive.org/web/20220929174444/https://fi.ort.edu.uy/innovaportal/file/2032/1/design_principles.pdf)这些概念后来由 Michael Feathers 建立，他向我们介绍了 SOLID 缩写词。在过去的 20 年里，这五个原则彻底改变了面向对象编程的世界，改变了我们编写软件的方式。

那么，什么是 SOLID，它如何帮助我们写出更好的代码？简而言之，Martin 和 Feathers 的设计原则鼓励我们创建更易维护、更易理解、更灵活的软件。因此，**随着我们的应用程序规模的增长，我们可以降低它们的复杂性**并在未来为我们自己省去许多麻烦！

以下五个概念构成了我们坚实的原则:

1.  单一责任
2.  **O** 笔/关
3.  伊斯科夫替代
4.  界面偏析
5.  依赖倒置

虽然这些概念看起来令人望而生畏，但通过一些简单的代码示例就可以很容易理解。在接下来的几节中，我们将深入研究这些原则，并用一个简单的 Java 例子来说明每一个原则。

## 3。单一责任

让我们从单一责任原则开始。正如我们所料，这个原则声明一个类应该只有一个责任。此外，它应该只有一个改变的理由。

这个原则如何帮助我们构建更好的软件？让我们来看看它的一些好处:

1.  测试–一个有一个职责的类将会有更少的测试用例。
2.  **低耦合度**–单个类中的功能越少，依赖关系就越少。
3.  **组织**–较小的、组织良好的类比单一的类更容易搜索。

例如，让我们看一个代表一本书的类:

```java
public class Book {

    private String name;
    private String author;
    private String text;

    //constructor, getters and setters
}
```

在这段代码中，我们存储了与一个`Book`实例相关的名称、作者和文本。

现在让我们添加几个方法来查询文本:

```java
public class Book {

    private String name;
    private String author;
    private String text;

    //constructor, getters and setters

    // methods that directly relate to the book properties
    public String replaceWordInText(String word, String replacementWord){
        return text.replaceAll(word, replacementWord);
    }

    public boolean isWordInText(String word){
        return text.contains(word);
    }
}
```

现在我们的`Book`类运行良好，我们可以在应用程序中存储任意多的书。

但是，如果我们不能将文本输出到我们的控制台并阅读它，那么存储信息又有什么用呢？

我们豁出去了，加一个打印方法:

```java
public class Book {
    //...

    void printTextToConsole(){
        // our code for formatting and printing the text
    }
}
```

然而，该代码违反了我们前面概述的单一责任原则。

为了解决我们的混乱，我们应该实现一个单独的类，只处理打印我们的文本:

```java
public class BookPrinter {

    // methods for outputting text
    void printTextToConsole(String text){
        //our code for formatting and printing the text
    }

    void printTextToAnotherMedium(String text){
        // code for writing to any other location..
    }
}
```

太棒了。我们不仅开发了一个类来减轻`Book `的打印任务，而且我们还可以利用我们的`BookPrinter `类将我们的文本发送到其他媒体。

无论是电子邮件、日志还是其他，我们都有一个单独的类专门处理这一问题。

## 4。打开进行扩展，关闭进行修改

现在是 O 在固体中的时候了，被称为**开闭原理。简单地说，**类应该对扩展开放，但对修改关闭。** **这样做，我们** **阻止自己修改现有代码，并在一个原本愉快的应用程序中引起潜在的新错误**。**

当然，该规则的一个例外是修复现有代码中的错误。

让我们通过一个简单的代码示例来探索这个概念。作为新项目的一部分，假设我们已经实现了一个`Guitar `类。

它功能齐全，甚至还有音量旋钮:

```java
public class Guitar {

    private String make;
    private String model;
    private int volume;

    //Constructors, getters & setters
}
```

我们推出应用程序，每个人都喜欢它。但是几个月后，我们决定`Guitar `有点无聊，可以用一个很酷的火焰图案让它看起来更摇滚。

在这一点上，打开`Guitar `类并添加火焰图案可能很诱人——但是谁知道在我们的应用程序中会出现什么错误。

相反，让我们**坚持开闭原则，简单地扩展我们的`Guitar `类**:

```java
public class SuperCoolGuitarWithFlames extends Guitar {

    private String flameColor;

    //constructor, getters + setters
}
```

通过扩展`Guitar `类，我们可以确保我们现有的应用程序不会受到影响。

## 5。利斯科夫换人

我们名单上的下一个是[利斯科夫替换](/web/20220929174444/https://www.baeldung.com/cs/liskov-substitution-principle)，这可以说是五个原则中最复杂的。简单地说，**如果类`A`是类`B`的一个子类型，我们应该能够用`A `替换`B `，而不会扰乱我们程序的行为。**

让我们直接跳到代码来帮助我们理解这个概念:

```java
public interface Car {

    void turnOnEngine();
    void accelerate();
}
```

上面，我们定义了一个简单的`Car `接口，其中有几个所有汽车都应该能够实现的方法:打开引擎，向前加速。

让我们实现我们的接口，并为这些方法提供一些代码:

```java
public class MotorCar implements Car {

    private Engine engine;

    //Constructors, getters + setters

    public void turnOnEngine() {
        //turn on the engine!
        engine.on();
    }

    public void accelerate() {
        //move forward!
        engine.powerOn(1000);
    }
}
```

正如我们的代码所描述的，我们有一个可以启动的引擎，我们可以增加功率。

但是等等——我们现在生活在电动汽车时代:

```java
public class ElectricCar implements Car {

    public void turnOnEngine() {
        throw new AssertionError("I don't have an engine!");
    }

    public void accelerate() {
        //this acceleration is crazy!
    }
}
```

通过把一辆没有引擎的汽车扔进组合中，我们在本质上改变了我们程序的行为。这公然违反了里斯科夫替代原则，比我们之前的两条原则更难解决。

一个可能的解决方案是将我们的模型重新设计成考虑到我们的`Car`的无引擎状态的接口。

## 6。界面分离

固体中的 I 代表界面分离，它简单地意味着**较大的界面应该分裂成较小的界面。通过这样做，我们可以确保实现类只需要关心它们感兴趣的方法。**

在这个例子中，我们将试着扮演动物园管理员。更确切地说，我们将在熊圈里工作。

让我们从一个界面开始，这个界面概括了我们作为一个养熊人的角色:

```java
public interface BearKeeper {
    void washTheBear();
    void feedTheBear();
    void petTheBear();
}
```

作为热心的动物园管理员，我们非常乐意为我们心爱的熊洗澡和喂食。但是我们都很清楚抚摸它们的危险。不幸的是，我们的接口相当大，我们别无选择，只能实现代码来宠爱熊。

让我们**通过将我们的大接口分成三个独立的接口来解决这个问题**:

```java
public interface BearCleaner {
    void washTheBear();
}

public interface BearFeeder {
    void feedTheBear();
}

public interface BearPetter {
    void petTheBear();
}
```

现在，由于接口分离，我们可以自由地只实现对我们重要的方法:

```java
public class BearCarer implements BearCleaner, BearFeeder {

    public void washTheBear() {
        //I think we missed a spot...
    }

    public void feedTheBear() {
        //Tuna Tuesdays...
    }
}
```

最后，我们可以把危险的事情留给鲁莽的人:

```java
public class CrazyPerson implements BearPetter {

    public void petTheBear() {
        //Good luck with that!
    }
}
```

更进一步，我们甚至可以从前面的例子中分离出我们的`[BookPrinter](#s) `类，以同样的方式使用接口隔离。通过用一个`print `方法实现一个`Printer`接口，我们可以实例化单独的`ConsoleBookPrinter `和`OtherMediaBookPrinter `类。

## 7。依赖性反转

**依赖反转的原理是指软件模块的解耦。这样，两者都将依赖于抽象，而不是高级模块依赖于低级模块。**

为了证明这一点，让我们用老一套的方法，用代码将 Windows 98 计算机变成现实:

```java
public class Windows98Machine {}
```

但是没有显示器和键盘的电脑有什么用呢？让我们给我们的构造函数各添加一个，这样我们实例化的每个`Windows98Computer `都预装了一个`Monitor `和一个`StandardKeyboard`:

```java
public class Windows98Machine {

    private final StandardKeyboard keyboard;
    private final Monitor monitor;

    public Windows98Machine() {
        monitor = new Monitor();
        keyboard = new StandardKeyboard();
    }

}
```

这段代码将会工作，我们将能够在我们的`Windows98Computer `类中自由地使用`StandardKeyboard`和`Monitor`。

问题解决了？不完全是。**通过用`new `关键字声明`StandardKeyboard `和`Monitor `，我们已经将这三个类紧密耦合在一起。**

这不仅使我们的`Windows98Computer `难以测试，而且我们也失去了在需要的时候用不同的类替换我们的`StandardKeyboard `类的能力。我们也被我们的`Monitor`类困住了。

让我们通过添加一个更通用的`Keyboard`接口并在我们的类中使用它来将我们的机器与`StandardKeyboard`解耦:

```java
public interface Keyboard { }
```

```java
public class Windows98Machine{

    private final Keyboard keyboard;
    private final Monitor monitor;

    public Windows98Machine(Keyboard keyboard, Monitor monitor) {
        this.keyboard = keyboard;
        this.monitor = monitor;
    }
}
```

这里，我们使用依赖注入模式来帮助将`Keyboard`依赖添加到`Windows98Machine`类中。

让我们也修改我们的`StandardKeyboard`类来实现`Keyboard`接口，以便它适合注入到`Windows98Machine`类中:

```java
public class StandardKeyboard implements Keyboard { }
```

现在我们的类被解耦并通过`Keyboard`抽象进行通信。如果我们愿意，我们可以很容易地用不同的接口实现来切换我们机器中的键盘类型。对于`Monitor`类，我们可以遵循同样的原则。

太棒了。我们已经解耦了依赖性，并且可以自由地用我们选择的任何测试框架来测试我们的`Windows98Machine `。

## 8。结论

在本文中，我们深入探讨了面向对象设计的坚实原则。

我们从一小段历史和这些原则存在的原因开始。

我们一个字母接一个字母地用一个违反原则的快速代码例子来分解每个原则的含义。然后，我们看到了如何修复我们的代码，并使它遵循坚实的原则。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20220929174444/https://github.com/eugenp/tutorials/tree/master/patterns-modules/solid)