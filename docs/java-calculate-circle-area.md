# 用 Java 计算圆的面积

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-calculate-circle-area>

## 1。概述

在这个快速教程中，我们将演示如何用 Java 计算圆的面积。

我们将使用众所周知的数学公式:`r^2 * PI`。

## 2.一种圆面积的计算方法

让我们首先创建一个执行计算的方法:

```java
private void calculateArea(double radius) {
    double area = radius * radius * Math.PI;
    System.out.println("The area of the circle [radius = " + radius + "]: " + area);
}
```

### 2.1.将半径作为命令行参数传递

现在我们可以读取命令行参数并计算面积:

```java
double radius = Double.parseDouble(args[0]);
calculateArea(radius);
```

当我们编译和运行程序时:

```java
java CircleArea.java
javac CircleArea 7
```

我们将得到以下输出:

```java
The area of the circle [radius = 7.0]: 153.93804002589985
```

### 2.2.从键盘上读取半径

获得半径值的另一种方法是使用来自用户的输入数据:

```java
Scanner sc = new Scanner(System.in);
System.out.println("Please enter radius value: ");
double radius = sc.nextDouble();
calculateArea(radius);
```

输出与前面的示例相同。

## 3.一个循环类

除了调用我们在第 2 节中看到的计算面积的方法之外，我们还可以创建一个表示圆的类:

```java
public class Circle {

    private double radius;

    public Circle(double radius) {
        this.radius = radius;
    }

    // standard getter and setter

    private double calculateArea() {
        return radius * radius * Math.PI;
    }

    public String toString() {
        return "The area of the circle [radius = " + radius + "]: " + calculateArea();
    }
}
```

我们应该注意一些事情。首先，我们不把面积保存为变量，因为它直接依赖于半径，所以我们可以很容易地计算它。其次，计算面积的方法是私有的，因为我们在`toString()`方法中使用它。**`toString()`方法不应该调用类中的任何公共方法，因为这些方法可能会被覆盖，它们的行为会与预期的不同。**

我们现在可以实例化我们的 Circle 对象:

```java
Circle circle = new Circle(7);
```

当然，输出将和以前一样。

## 4.结论

在这篇简短扼要的文章中，我们展示了使用 Java 计算圆的面积的不同方法。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220630020210/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-numbers-2)