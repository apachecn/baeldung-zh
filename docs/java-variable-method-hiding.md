# 隐藏在 Java 中的变量和方法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-variable-method-hiding>

## 1。简介

在本教程中，**我们将学习隐藏在 Java 语言**中的变量和方法。

首先，我们将理解每个场景的概念和目的。之后，我们将深入到用例中，检查不同的例子。

## 2。变量隐藏

当我们在一个局部作用域中声明了一个与外部作用域中已经存在的属性同名的属性时，就会发生变量隐藏。

在开始举例之前，让我们简要回顾一下 Java 中可能的变量作用域。我们可以用以下类别来定义它们:

*   局部变量——在一段代码中声明，如方法、构造函数，在任何带花括号的代码块中
*   实例变量——在类内部定义，属于对象的实例
*   类或`static`变量——在带有`static`关键字的类中声明。它们有一个类级别的作用域。

现在，让我们用例子来描述每一类变量的隐藏。

### 2.1。当地的力量

让我们来看看`HideVariable`类:

```java
public class HideVariable {

    private String message = "this is instance variable";

    HideVariable() {
        String message = "constructor local variable";
        System.out.println(message);
    }

    public void printLocalVariable() {
        String message = "method local variable";
        System.out.println(message);
    }

    public void printInstanceVariable() {
        String message = "method local variable";
        System.out.println(this.message);
    }
}
```

这里我们有在 4 个不同地方声明的*消息*变量。构造函数和两个方法内部声明的局部变量隐藏了实例变量。

让我们测试一个对象的初始化和调用方法:

```java
HideVariable variable = new HideVariable();
variable.printLocalVariable();

variable.printInstanceVariable();
```

上面代码的输出是:

```java
constructor local variable
method local variable
this is instance variable
```

这里，前两个调用是检索局部变量。

为了从局部范围访问实例变量，我们可以像在`printInstanceVariable()`方法中那样使用`this`关键字。

### 2.2。隐藏和层级

类似地，当子类和父类都有一个同名的变量时，子类的变量会对父类隐藏这个变量。

假设我们有父类:

```java
public class ParentVariable {

    String instanceVariable = "parent variable";

    public void printInstanceVariable() {
        System.out.println(instanceVariable);
    }
}
```

之后，我们定义一个子类:

```java
public class ChildVariable extends ParentVariable {

    String instanceVariable = "child variable";

    public void printInstanceVariable() {
        System.out.println(instanceVariable);
    }
}
```

为了测试它，让我们初始化两个实例。一个包含父类，另一个包含子类，然后对它们分别调用`printInstanceVariable()`方法:

```java
ParentVariable parentVariable = new ParentVariable();
ParentVariable childVariable = new ChildVariable();

parentVariable.printInstanceVariable();
childVariable.printInstanceVariable();
```

输出显示隐藏的属性:

```java
parent variable
child variable
```

在大多数情况下，我们应该避免在父类和子类中创建同名的变量。相反，我们应该使用合适的访问修饰符，比如`private `，并为此提供 getter/setter 方法。

## 3。方法隐藏

方法隐藏可能发生在 java 的任何层次结构中。当一个子类定义了一个与父类中的静态方法具有相同签名的静态方法时，那么子类的方法*隐藏了父类中的方法*。要了解更多关于关键字`static`、[的信息，这篇文章是一个很好的起点。](/web/20221129003946/https://www.baeldung.com/spring-bean-scopes)

涉及实例方法的相同行为称为方法重写。要了解更多关于方法覆盖 checkout 的信息，请点击查看我们的[指南。](/web/20221129003946/https://www.baeldung.com/java-method-overload-override)

现在，让我们来看看这个实际的例子:

```java
public class BaseMethodClass {

    public static void printMessage() {
        System.out.println("base static method");
    }
}
```

`BaseMethodClass`有一个单独的`printMessage() static`方法。

接下来，让我们创建一个与基类具有相同签名的子类:

```java
public class ChildMethodClass extends BaseMethodClass {

    public static void printMessage() {
        System.out.println("child static method");
    }
}
```

它是这样工作的:

```java
ChildMethodClass.printMessage();
```

调用`printMessage()`方法后的输出:

```java
child static method
```

`ChildMethodClass.printMessage() `隐藏了`BaseMethodClass`中的方法。

### 3.1。方法隐藏 vs 重写

隐藏不像重写那样工作，因为静态方法不是多态的。重写只发生在实例方法中。它支持后期绑定，所以哪个方法将被调用是在运行时决定的。

另一方面，方法隐藏适用于静态方法。因此它是在编译时确定的。

## 4。结论

在本文中，我们回顾了 Java 中隐藏方法和变量的概念。我们展示了变量隐藏和阴影的不同场景。文章的重要亮点还在于比较了方法覆盖和隐藏。

和往常一样，完整的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221129003946/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-oop-inheritance)