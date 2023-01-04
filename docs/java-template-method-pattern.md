# 用 Java 实现模板方法模式

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-template-method-pattern>

## 1。概述

在这个快速教程中，我们将看到如何利用[模板方法模式](https://web.archive.org/web/20221031114425/https://en.wikipedia.org/wiki/Template_method_pattern)——最流行的 [GoF](https://web.archive.org/web/20221031114425/https://en.wikipedia.org/wiki/Design_Patterns) 模式之一。

通过将逻辑封装在一个方法中，可以更容易地实现复杂的算法。

## 2。实施

为了演示模板方法模式是如何工作的，让我们创建一个简单的例子来表示构建一个计算机站。

给定模式的定义，**算法的结构将在定义模板`build()`方法**的基类中定义:

```java
public abstract class ComputerBuilder {

    // ...

    public final Computer buildComputer() {
        addMotherboard();
        setupMotherboard();
        addProcessor();
        return new Computer(computerParts);
    }

    public abstract void addMotherboard();
    public abstract void setupMotherboard();
    public abstract void addProcessor();

    // ...
}
```

**`ComputerBuilder`类负责通过声明添加和设置不同组件**的方法来概述构建计算机所需的步骤，例如主板和处理器。

这里，**`build()`方法是模板方法**，它定义了组装计算机部件的算法步骤，并返回完全初始化的`Computer`实例。

注意，i **t 被声明为`final`以防止它被覆盖。**

## 3。行动中

已经设置了基类，让我们试着通过创建两个子类来使用它。一个制造“标准”计算机，另一个制造“高端”计算机:

```java
public class StandardComputerBuilder extends ComputerBuilder {

    @Override
    public void addMotherboard() {
        computerParts.put("Motherboard", "Standard Motherboard");
    }

    @Override
    public void setupMotherboard() {
        motherboardSetupStatus.add(
          "Screwing the standard motherboard to the case.");
        motherboardSetupStatus.add(
          "Pluging in the power supply connectors.");
        motherboardSetupStatus.forEach(
          step -> System.out.println(step));
    }

    @Override
    public void addProcessor() {
        computerParts.put("Processor", "Standard Processor");
    }
}
```

这里是`HighEndComputerBuilder`的变体:

```java
public class HighEndComputerBuilder extends ComputerBuilder {

    @Override
    public void addMotherboard() {
        computerParts.put("Motherboard", "High-end Motherboard");
    }

    @Override
    public void setupMotherboard() {
        motherboardSetupStatus.add(
          "Screwing the high-end motherboard to the case.");
        motherboardSetupStatus.add(
          "Pluging in the power supply connectors.");
        motherboardSetupStatus.forEach(
          step -> System.out.println(step));
    }

    @Override
    public void addProcessor() {
         computerParts.put("Processor", "High-end Processor");
    }
}
```

正如我们所看到的，我们不需要担心整个组装过程，只需要为单独的方法提供实现。

现在，让我们来看看它的运行情况:

```java
new StandardComputerBuilder()
  .buildComputer();
  .getComputerParts()
  .forEach((k, v) -> System.out.println("Part : " + k + " Value : " + v));

new HighEndComputerBuilder()
  .buildComputer();
  .getComputerParts()
  .forEach((k, v) -> System.out.println("Part : " + k + " Value : " + v));
```

## 4。Java 核心库中的模板方法

这种模式在 Java 核心库中被广泛使用，例如通过 [java.util.AbstractList](https://web.archive.org/web/20221031114425/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/AbstractList.html) ，或者 [java.util.AbstractSet.](https://web.archive.org/web/20221031114425/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/AbstractSet.html)

例如，`Abstract List` 提供了 [`List`](https://web.archive.org/web/20221031114425/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/List.html) 接口的框架实现。

模板方法的一个例子是`addAll()`方法，尽管它没有被明确定义为`final:`

```java
public boolean addAll(int index, Collection<? extends E> c) {
    rangeCheckForAdd(index);
    boolean modified = false;
    for (E e : c) {
        add(index++, e);
        modified = true;
    }
    return modified;
}
```

用户只需要实现`add()`方法:

```java
public void add(int index, E element) {
    throw new UnsupportedOperationException();
}
```

在这里，程序员的责任是提供一个实现，将一个元素添加到列表的给定索引处(列表算法的变体部分)。

## 5。结论

在本文中，我们展示了模板方法模式以及如何用 Java 实现它。

模板方法模式促进了代码重用和解耦，但是以使用继承为代价。

和往常一样，本文中展示的所有代码示例都可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221031114425/https://github.com/eugenp/tutorials/tree/master/patterns-modules/design-patterns-behavioral)