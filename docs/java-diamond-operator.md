# Java 中的菱形运算符指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-diamond-operator>

## 1。概述

在本文中，我们将看看 Java 中的 **diamond 操作符，以及[泛型](/web/20221208143839/https://www.baeldung.com/java-generics)和集合 API 如何影响它的演变**。

## 2。原始类型

**在 Java 1.5 之前，集合 API 仅支持原始类型**——在构造集合时，类型参数无法参数化:

```java
List cars = new ArrayList();
cars.add(new Object());
cars.add("car");
cars.add(new Integer(1));
```

这允许添加任何类型，**导致运行时的潜在转换异常**。

## 3。仿制药

在 Java 1.5 中，引入了泛型——**，它允许我们在声明和构造对象时对类**的类型参数进行参数化，包括集合 API 中的类型参数:

```java
List<String> cars = new ArrayList<String>();
```

此时，我们必须**在构造函数**中指定参数化类型，这可能有些难以理解:

```java
Map<String, List<Map<String, Map<String, Integer>>>> cars 
 = new HashMap<String, List<Map<String, Map<String, Integer>>>>();
```

这种方法的原因是为了向后兼容，原始类型仍然存在，所以编译器需要区分这些原始类型和泛型:

```java
List<String> generics = new ArrayList<String>();
List<String> raws = new ArrayList();
```

尽管编译器仍然允许我们在构造函数中使用原始类型，但它会提示我们一条警告消息:

```java
ArrayList is a raw type. References to generic type ArrayList<E> should be parameterized
```

## 4。钻石操作员

Java 1.7 中引入的 diamond 操作符增加了类型推断，减少了赋值的冗长性——当使用泛型时:

```java
List<String> cars = new ArrayList<>();
```

Java 1.7 编译器的类型推断特性**决定了与调用**匹配的最合适的构造函数声明。

考虑以下用于车辆和引擎的接口和类层次结构:

```java
public interface Engine { }
public class Diesel implements Engine { }
public interface Vehicle<T extends Engine> { }
public class Car<T extends Engine> implements Vehicle<T> { }
```

让我们使用菱形操作符创建一个`Car`的新实例:

```java
Car<Diesel> myCar = new Car<>();
```

在内部，编译器知道`Diesel`实现了`Engine`接口，然后能够通过推断类型来确定合适的构造函数。

## 5。结论

简单地说，diamond 操作符为编译器增加了类型推断功能，减少了泛型赋值的冗长性。

本教程的一些例子可以在 GitHub 项目上找到[，可以随意下载并使用它。](https://web.archive.org/web/20221208143839/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-operators)