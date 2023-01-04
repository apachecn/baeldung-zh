# Java 中的开放/封闭原则

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-open-closed-principle>

## 1.概观

在本教程中，我们将讨论作为面向对象编程坚实原则之一的开/闭原则(OCP)。

总的来说，我们将详细讨论这个原则是什么，以及在设计软件时如何实现它。

## 2.开/关原则

顾名思义，这个原则规定软件实体应该对扩展开放，但对修改关闭。因此，当业务需求改变时，实体可以被扩展，但不能被修改。

对于下面的插图，**我们将关注接口是如何成为遵循 OCP 的一种方式。**

### 2.1.不合规

让我们假设我们正在构建一个计算器应用程序，它可能有几种运算，比如加法和减法。

首先，我们将定义一个顶级接口—`CalculatorOperation`:

```java
public interface CalculatorOperation {}
```

**让我们定义一个`Addition`类，它将两个数相加并实现 C `alculatorOperation` :**

```java
public class Addition implements CalculatorOperation {
    private double left;
    private double right;
    private double result = 0.0;

    public Addition(double left, double right) {
        this.left = left;
        this.right = right;
    }

    // getters and setters

}
```

到目前为止，我们只有一个类`Addition,` ，所以我们需要定义另一个名为`Subtraction`的类:

```java
public class Subtraction implements CalculatorOperation {
    private double left;
    private double right;
    private double result = 0.0;

    public Subtraction(double left, double right) {
        this.left = left;
        this.right = right;
    }

    // getters and setters
}
```

**现在让我们定义我们的主类，它将执行我们的计算器操作:**

```java
public class Calculator {

    public void calculate(CalculatorOperation operation) {
        if (operation == null) {
            throw new InvalidParameterException("Can not perform operation");
        }

        if (operation instanceof Addition) {
            Addition addition = (Addition) operation;
            addition.setResult(addition.getLeft() + addition.getRight());
        } else if (operation instanceof Subtraction) {
            Subtraction subtraction = (Subtraction) operation;
            subtraction.setResult(subtraction.getLeft() - subtraction.getRight());
        }
    }
}
```

**虽然这看起来不错，但这不是 OCP 的好例子。**当增加乘法或除法功能的新需求到来时，我们除了改变`Calculator`类的`calculate`方法之外别无选择。

因此，我们可以说这段代码不符合 OCP 标准。

### 2.2.符合 OCP 标准

正如我们已经看到的，我们的计算器应用程序还不符合 OCP。`calculate`方法中的代码将随着每个新的操作支持请求而改变。因此，我们需要提取这些代码，并将其放入抽象层。

一种解决方案是将每个操作委托给它们各自的类:

```java
public interface CalculatorOperation {
    void perform();
}
```

**因此，`Addition`类可以实现两个数相加的逻辑:**

```java
public class Addition implements CalculatorOperation {
    private double left;
    private double right;
    private double result;

    // constructor, getters and setters

    @Override
    public void perform() {
        result = left + right;
    }
}
```

同样，更新后的`Subtraction`类也有类似的逻辑。类似于`Addition`和`Subtraction`，作为一个新的变更请求，我们可以实现`division`逻辑:

```java
public class Division implements CalculatorOperation {
    private double left;
    private double right;
    private double result;

    // constructor, getters and setters
    @Override
    public void perform() {
        if (right != 0) {
            result = left / right;
        }
    }
}
```

**最后，我们的`Calculator`类不需要实现新的逻辑，因为我们引入了新的操作符:**

```java
public class Calculator {

    public void calculate(CalculatorOperation operation) {
        if (operation == null) {
            throw new InvalidParameterException("Cannot perform operation");
        }
        operation.perform();
    }
} 
```

那样的话，这个类对于修改来说是*关闭的*，而对于扩展来说是`open` 。

## 3.结论

在本教程中，我们学习了什么是 OCP 的定义，然后阐述了这个定义。然后我们看到了一个简单的计算器应用程序的例子，它的设计是有缺陷的。最后，我们通过使设计符合 OCP 来使它变得更好。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20220805100812/https://github.com/eugenp/tutorials/tree/master/patterns/solid)