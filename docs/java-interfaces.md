# Java 接口

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-interfaces>

## 1.概观

在本教程中，我们将讨论 Java 中的接口。我们还将看到 Java 如何使用它们来实现多态和多重继承。

## 2.Java 中的接口是什么？

在 Java 中，接口是一种抽象类型，包含一组方法和常量变量。它是 Java 的核心概念之一，用于**实现抽象、[多态](/web/20220930151459/https://www.baeldung.com/java-polymorphism)和[多重继承](/web/20220930151459/https://www.baeldung.com/java-inheritance)** 。

让我们看一个简单的 Java 接口示例:

```java
public interface Electronic {

    // Constant variable
    String LED = "LED";

    // Abstract method
    int getElectricityUse();

    // Static method
    static boolean isEnergyEfficient(String electtronicType) {
        if (electtronicType.equals(LED)) {
            return true;
        }
        return false;
    }

    //Default method
    default void printDescription() {
        System.out.println("Electronic Description");
    }
} 
```

我们可以通过使用`implements`关键字在 Java 类中实现一个接口。

接下来，让我们也创建一个实现我们刚刚创建的`Electronic`接口的`Computer`类:

```java
public class Computer implements Electronic {

    @Override
    public int getElectricityUse() {
        return 1000;
    }
} 
```

### 2.1.创建接口的规则

在界面中，我们可以使用:

*   [常量变量](/web/20220930151459/https://www.baeldung.com/java-final)
*   [抽象方法](/web/20220930151459/https://www.baeldung.com/java-abstract-class)
*   [静态方法](/web/20220930151459/https://www.baeldung.com/java-static-default-methods)
*   [默认方法](/web/20220930151459/https://www.baeldung.com/java-static-default-methods)

我们还应该记住:

*   我们不能直接实例化接口
*   接口可以是空的，没有方法或变量
*   我们不能在接口定义中使用`final`这个词，因为它会导致编译错误
*   所有接口声明都应该有`public`或默认访问修饰符；编译器会自动添加`abstract`修饰符
*   接口方法不能是`protected`或`final`
*   直到 Java 9，接口方法都不能是`private`；然而，Java 9 引入了在接口中定义[私有方法的可能性](/web/20220930151459/https://www.baeldung.com/java-interface-private-methods)
*   接口变量定义为`public`、`static`、`final`；我们不允许改变他们的能见度

## 3.使用它们我们能达到什么目的？

### 3.1.行为功能

我们使用接口来添加某些不相关的类可以使用的行为功能。例如，`Comparable`、`Comparator`和`Cloneable`是可以由不相关的类实现的 Java 接口。下面是一个用于比较`Employee `类的两个实例的`Comparator`接口的例子:

```java
public class Employee {

    private double salary;

    public double getSalary() {
        return salary;
    }

    public void setSalary(double salary) {
        this.salary = salary;
    }
}

public class EmployeeSalaryComparator implements Comparator<Employee> {

    @Override
    public int compare(Employee employeeA, Employee employeeB) {
        if (employeeA.getSalary() < employeeB.getSalary()) {
            return -1;
        } else if (employeeA.getSalary() > employeeB.getSalary()) { 
            return 1;
        } else {
            return 0;
        }
    }
} 
```

更多信息，请访问我们关于 Java 中的 [`Comparator`和`Comparable`的教程。](/web/20220930151459/https://www.baeldung.com/java-comparator-comparable)

### 3.2.多重继承

Java 类支持单一继承。然而，通过使用接口，我们也能够实现多重继承。

例如，在下面的例子中，我们注意到`Car`类实现了`Fly`和`Transform`接口。通过这样做，它继承了方法`fly`和`transform`:

```java
public interface Transform {
    void transform();
}

public interface Fly {
    void fly();
}

public class Car implements Fly, Transform {

    @Override
    public void fly() {
        System.out.println("I can Fly!!");
    }

    @Override
    public void transform() {
        System.out.println("I can Transform!!");
    }
} 
```

### 3.3.多态性

让我们从提出这个问题开始:什么是[多态性](/web/20220930151459/https://www.baeldung.com/java-polymorphism)？它是一个对象在运行时采取不同形式的能力。更具体地说，它是在运行时与特定对象类型相关的 override 方法的执行。

**在 Java 中，我们可以使用接口实现多态性。**例如，`Shape`接口可以采取不同的形式——它可以是`Circle`或`Square.`

让我们从定义`Shape`接口开始:

```java
public interface Shape {
    String name();
} 
```

现在让我们也创建一个`Circle`类:

```java
public class Circle implements Shape {

    @Override
    public String name() {
        return "Circle";
    }
} 
```

还有`Square` 类:

```java
public class Square implements Shape {

    @Override
    public String name() {
        return "Square";
    }
} 
```

最后，是时候使用我们的`Shape`接口和它的实现来看看多态的作用了。让我们实例化一些`Shape`对象，将它们添加到一个`List`T3 中，最后，在一个循环中打印它们的名称:

```java
List<Shape> shapes = new ArrayList<>();
Shape circleShape = new Circle();
Shape squareShape = new Square();

shapes.add(circleShape);
shapes.add(squareShape);

for (Shape shape : shapes) {
    System.out.println(shape.name());
} 
```

## 4.接口中的默认方法

Java 7 及更低版本中的传统接口不提供向后兼容性。

这意味着**如果您有用 Java 7 或更早版本编写的遗留代码，并且您决定向现有接口添加一个抽象方法，那么实现该接口的所有类都必须覆盖新的抽象方法**。否则，代码将会中断。

**Java 8 通过引入默认方法**解决了这个问题，这个方法是可选的，可以在接口级实现。

## 5。接口继承规则

为了通过接口实现多重继承，我们必须记住一些规则。让我们详细检查一下这些。

### 5.1.接口扩展另一个接口

当一个接口`extends`继承另一个接口时，它继承了该接口的所有抽象方法。让我们首先创建两个接口，`HasColor`和`Shape`:

```java
public interface HasColor {
    String getColor();
}

public interface Box extends HasColor {
    int getHeight()
} 
```

在上面的例子中，*框*使用关键字`extends.`从`HasColor`继承，通过这样做，*框*接口继承`getColor`。因此，`Box `接口现在有两个方法:`getColor`和`getHeight`。

### 5.2.实现接口的抽象类

当一个抽象类实现一个接口时，它继承它所有的抽象和默认方法。让我们考虑一下`Transform`接口和实现它的`abstract`类`Vehicle`:

```java
public interface Transform {

    void transform();
    default void printSpecs(){
        System.out.println("Transform Specification");
    }
}

public abstract class Vehicle implements Transform {} 
```

在这个例子中，`Vehicle`类继承了两个方法:抽象的`transform`方法和默认的`printSpecs`方法。

## 6.功能界面

Java 从早期就有很多函数接口，比如`Comparable`(从 Java 1.2 开始)和`Runnable`(从 Java 1.0 开始)。

Java 8 引入了新的功能接口，如`Predicate`、`Consumer`和`Function`。要了解更多，请访问我们关于 Java 8 中[函数接口的教程。](/web/20220930151459/https://www.baeldung.com/java-8-functional-interfaces)

## 7.结论

在本教程中，我们对 Java 接口进行了概述，并讨论了如何使用它们来实现多态性和多重继承。

和往常一样，完整的代码样本可以在 GitHub 上获得。