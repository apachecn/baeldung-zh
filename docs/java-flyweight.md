# Java 中的 Flyweight 模式

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-flyweight>

## 1。概述

在本文中，我们将看看 flyweight 设计模式。这种模式用于减少内存占用。它还可以提高对象实例化开销很大的应用程序的性能。

简单地说，flyweight 模式基于一个工厂，它通过在创建后存储对象来回收创建的对象。每次请求一个对象时，工厂都会查找该对象，以检查它是否已经被创建。如果有，则返回现有对象，否则，创建一个新对象，存储并返回。

flyweight 对象的状态由一个与其他类似对象共享的不变组件(**内在**)和一个可以由客户端代码操作的变量组件(**外在**)组成。

很重要的一点是，flyweight 对象是不可变的:对状态的任何操作都必须由工厂来执行。

## 2。实施

该模式的主要元素是:

*   一个接口，它定义了客户端代码可以在 flyweight 对象上执行的操作
*   我们接口的一个或多个具体实现
*   处理对象实例化和缓存的工厂

让我们看看如何实现每个组件。

### 2.1。车辆接口

首先，我们将创建一个`Vehicle`接口。由于该接口将是工厂方法的返回类型，我们需要确保公开所有相关的方法:

```java
public void start();
public void stop();
public Color getColor();
```

### 2.2。混凝土车

接下来，让我们制作一个`Car`类作为具体的`Vehicle.` 我们的汽车将实现车辆接口的所有方法。至于它的状态，它将有一个引擎和一个色域:

```java
private Engine engine;
private Color color;
```

### 2.3。车辆工厂

最后但同样重要的是，我们将创建`VehicleFactory`。制造一辆新车是一项非常昂贵的操作，因此工厂将只生产一种颜色的汽车。

为此，我们使用地图作为简单的缓存来跟踪创建的车辆:

```java
private static Map<Color, Vehicle> vehiclesCache
  = new HashMap<>();

public static Vehicle createVehicle(Color color) {
    Vehicle newVehicle = vehiclesCache.computeIfAbsent(color, newColor -> { 
        Engine newEngine = new Engine();
        return new Car(newEngine, newColor);
    });
    return newVehicle;
}
```

请注意，客户端代码只能影响对象的外在状态(我们车辆的颜色),并将其作为参数传递给`createVehicle`方法。

## 3。用例

### 3.1。数据压缩

flyweight 模式的目标是通过共享尽可能多的数据来减少内存使用，因此，它是无损压缩算法的良好基础。在这种情况下，每个 flyweight 对象充当一个指针，其外在状态是上下文相关的信息。

这种用法的一个典型例子是在文字处理器中。这里，每个角色都是一个共享渲染所需数据的 flyweight 对象。因此，只有文档中字符的位置会占用额外的内存。

### 3.2。数据缓存

许多现代应用程序使用缓存来改善响应时间。flyweight 模式类似于缓存的核心概念，可以很好地满足这个目的。

当然，这种模式和典型的通用缓存在复杂性和实现上有一些关键的区别。

## 4。结论

总而言之，这篇快速教程主要关注 Java 中的 flyweight 设计模式。我们还检查了一些涉及该模式的最常见的场景。

所有例子的代码都可以在 GitHub 项目的[上找到。](https://web.archive.org/web/20221205165540/https://github.com/eugenp/tutorials/tree/master/patterns-modules/design-patterns-creational)