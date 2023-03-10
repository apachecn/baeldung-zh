# 代理、装饰、适配器和桥模式

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-structural-design-patterns>

## 1。简介

在本文中，我们将关注 Java 中的结构设计模式——并讨论它们是什么以及它们之间的一些基本区别。

## 2。结构设计模式

根据四人帮(g Of)的说法，设计模式可以分为三种类型:

1.  创造型的
2.  结构的
3.  行为的

简单地说，结构模式处理类和对象的组成。它们提供了使用对象组合和继承来创建某种抽象的不同方式。

## 3。代理模式

使用这种模式，**我们创建一个中介，作为另一个资源**的接口，例如，一个文件，一个连接。这种二级访问为真实组件提供了代理，并保护它免受底层复杂性的影响。

关于该模式的详细示例，请看专门的帖子:[Java 中的代理模式](/web/20220628235439/https://www.baeldung.com/java-proxy-pattern)。

**差异化的关键点:**

*   代理提供了与它保存引用的对象相同的接口，它不以任何方式修改数据；它与适配器和装饰器模式形成对比，适配器和装饰器模式分别改变和装饰预先存在的实例的功能
*   代理通常在编译时就有关于真正主体的信息，而装饰器和适配器在运行时被注入，只知道实际对象的接口

## 4.装饰图案

此模式对于增强对象的行为非常有用。详细的概述，请看这里的重点教程:[Java 中的装饰模式](/web/20220628235439/https://www.baeldung.com/java-decorator-pattern)

**差异化的关键点:**

*   尽管代理模式和装饰模式有相似的结构，但它们的意图不同；虽然代理的主要目的是便于使用或控制访问，但是装饰者附加了额外的责任
*   代理和适配器模式都包含对原始对象的引用
*   这种模式的所有装饰器都可以递归使用，次数不限，这在其他模型中是不可能的

## 5。适配器模式

适配器模式用于连接两个不能直接连接的不兼容接口。一个适配器用一个新的接口包装一个现有的类，这样它就变得与所需的接口兼容。

关于详细的描述和实现，请看 Java 中的专用 post:[Adapter Pattern](/web/20220628235439/https://www.baeldung.com/java-adapter-pattern)

**适配器和代理模式之间的主要区别是:**

*   代理提供相同的接口，而适配器提供与其客户端兼容的不同接口
*   适配器模式是在应用程序组件设计完成后使用的，这样我们就可以在不修改源代码的情况下使用它们。这与桥接模式形成对比，桥接模式在组件设计之前使用。

## 6。桥接模式

**桥模式用于将抽象从其实现中分离出来**,这样两者可以独立变化。

这意味着创建一个桥接接口，它使用 OOP 原理将职责分离到不同的抽象类中。

关于详细的描述和实现，看一下专门的帖子:Java 中的[桥模式](/web/20220628235439/https://www.baeldung.com/java-bridge-pattern)

**差异化的关键点**:

*   桥模式只能在设计应用程序之前实现。
*   允许抽象和实现独立地改变，而适配器模式使得不兼容的类可以一起工作

## 7。结论

在本文中，我们重点讨论了结构设计模式及其一些类型之间的差异。

一如既往，本教程的完整实现可以在 Github 上找到[。](https://web.archive.org/web/20220628235439/https://github.com/eugenp/tutorials/tree/master/patterns/design-patterns-structural)