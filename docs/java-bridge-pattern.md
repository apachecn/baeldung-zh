# Java 中的桥模式

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-bridge-pattern>

## 1。概述

由 Gang of Four (GoF)引入的桥设计模式的官方定义是将抽象与其实现分离，以便两者可以独立变化。

这意味着创建一个桥接接口，它使用 OOP 原理将职责分离到不同的抽象类中。

## 2。桥接模式示例

对于桥模式，我们将考虑两个抽象层；一个是用不同颜色填充的几何形状(如三角形和正方形)(我们的第二个抽象层):

[![](img/6e4a3b284cbb9e0b42f97854aa7c3c99.png)](/web/20220526051721/https://www.baeldung.com/wp-content/uploads/2017/09/zfq_OUu_M.jpg)

首先，我们将定义一个颜色界面:

```java
public interface Color {
    String fill();
}
```

现在我们将为这个接口创建一个具体的类:

```java
public class Blue implements Color {
    @Override
    public String fill() {
        return "Color is Blue";
    }
}
```

现在让我们创建一个抽象的`Shape`类，它包含一个对`Color` 对象的引用(桥):

```java
public abstract class Shape {
    protected Color color;

    //standard constructors

    abstract public String draw();
}
```

我们现在将创建一个`Shape` 接口的具体类，它也将利用来自`Color` 接口的方法:

```java
public class Square extends Shape {

    public Square(Color color) {
        super(color);
    }

    @Override
    public String draw() {
        return "Square drawn. " + color.fill();
    }
}
```

对于这种模式，以下断言将是正确的:

```java
@Test
public void whenBridgePatternInvoked_thenConfigSuccess() {
    //a square with red color
    Shape square = new Square(new Red());

    assertEquals(square.draw(), "Square drawn. Color is Red");
}
```

这里，我们使用桥模式并传递所需的颜色对象。正如我们在输出中可以注意到的，该形状以所需的颜色绘制:

```java
Square drawn. Color: Red
Triangle drawn. Color: Blue
```

## 3。结论

在本文中，我们看了一下桥设计模式。在下列情况下，这是一个很好的选择:

*   当我们希望一个父抽象类定义一组基本规则，而具体类添加附加规则时
*   当我们有一个抽象类，它有一个对对象的引用，并且它有抽象方法，这些方法将在每个具体的类中定义

这个例子的完整源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220526051721/https://github.com/eugenp/tutorials/tree/master/patterns/design-patterns-structural)