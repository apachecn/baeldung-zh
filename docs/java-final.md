# Java 中的“final”关键字

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-final>

## 1。概述

虽然继承使我们能够重用现有的代码，但有时我们确实需要**为可扩展性**设置限制，原因有很多；关键字`final`允许我们这样做。

在本教程中，我们将看看`final`关键字对类、方法和变量的意义。

## 2。`Final`班级

**标记为`final`的类不能扩展。**如果我们看看 Java 核心库的代码，我们会发现那里有许多`final`类。一个例子是`String`级。

如果我们可以扩展`String`类，覆盖它的任何方法，并用我们特定的`String`子类的实例替换所有的`String`实例，考虑这种情况。

对`String`对象的操作结果将变得不可预测。鉴于`String`类到处都在使用，这是不可接受的。这就是为什么`String`级被标为`final`的原因。

任何从`final`类继承的尝试都会导致编译器错误。为了演示这一点，让我们创建`final`类`Cat`:

```java
public final class Cat {

    private int weight;

    // standard getter and setter
}
```

让我们试着扩展它:

```java
public class BlackCat extends Cat {
}
```

我们将看到编译器错误:

```java
The type BlackCat cannot subclass the final class Cat
```

注意**类声明中的`final`关键字并不意味着这个类的对象是不可变的**。我们可以自由改变`Cat`对象的字段:

```java
Cat cat = new Cat();
cat.setWeight(1);

assertEquals(1, cat.getWeight()); 
```

我们只是不能延长它。

如果我们严格遵循良好设计的规则，我们应该仔细地创建并记录一个类，或者出于安全原因将其声明为`final`。然而，我们在创建`final`类时应该小心。

注意，创建一个类`final`意味着没有其他程序员可以改进它。想象一下，我们正在使用一个类，但是没有它的源代码，并且有一个方法有问题。

如果类是`final,`，我们就不能扩展它来覆盖方法并修复问题。换句话说，我们失去了可扩展性，这是面向对象编程的好处之一。

## 3。`Final`方法

**标记为`final`的方法不能被覆盖。**当我们设计一个类，觉得一个方法不应该被覆盖时，我们可以让这个方法`final`。我们还可以在 Java 核心库中找到很多`final`方法。

有时我们不需要完全禁止类扩展，只需要防止一些方法的重写。一个很好的例子就是`Thread`类。扩展它从而创建一个自定义线程类是合法的。但它的`isAlive()`方法是`final`。

此方法检查线程是否处于活动状态。由于多种原因，正确地覆盖`isAlive()`方法是不可能的。其中之一就是这个方法是原生的。本机代码是用另一种编程语言实现的，通常特定于运行它的操作系统和硬件。

让我们创建一个`Dog`类，并使它的`sound()`方法`final`:

```java
public class Dog {
    public final void sound() {
        // ...
    }
}
```

现在让我们扩展`Dog`类并尝试覆盖它的`sound()`方法:

```java
public class BlackDog extends Dog {
    public void sound() {
    }
}
```

我们将看到编译器错误:

```java
- overrides
com.baeldung.finalkeyword.Dog.sound
- Cannot override the final method from Dog
sound() method is final and can’t be overridden
```

如果我们类的一些方法被其他方法调用，要考虑让被调用的方法`final`。否则，重写它们会影响调用方的工作，并导致令人惊讶的结果。

如果我们的构造函数调用其他方法，出于上述原因，我们一般应该声明这些方法`final`。

制作类的所有方法`final`和标记类本身`final`有什么区别？在第一种情况下，我们可以扩展该类并向其添加新方法。

在第二种情况下，我们不能这样做。

## 4。`Final`变数

**标记为`final`的变量不能被重新分配。**一旦`final`变量被初始化，它就不能被改变。

### 4.1。`Final`原始变量

让我们声明一个原始的`final`变量`i,`，然后给它赋值 1。

让我们试着给它赋值 2:

```java
public void whenFinalVariableAssign_thenOnlyOnce() {
    final int i = 1;
    //...
    i=2;
}
```

编译器说:

```java
The final local variable i may already have been assigned
```

### 4.2。`Final`参考变量

如果我们有一个`final`引用变量，我们也不能给它重新赋值。但是**这并不意味着它引用的对象是不可变的**。我们可以随意改变这个物体的属性。

为了演示这一点，让我们声明`final`引用变量`cat`并初始化它:

```java
final Cat cat = new Cat();
```

如果我们尝试重新分配它，我们会看到一个编译器错误:

```java
The final local variable cat cannot be assigned. It must be blank and not using a compound assignment
```

但是我们可以改变`Cat`实例的属性:

```java
cat.setWeight(5);

assertEquals(5, cat.getWeight());
```

### 4.3。`Final`原野

**`Final`字段可以是常量，也可以是一次写入字段。**为了区分它们，我们应该问一个问题——如果我们要序列化对象，我们会包含这个字段吗？如果不是，那么它就不是对象的一部分，而是一个常数。

请注意，根据命名约定，类常量应该是大写的，由下划线(“_”)字符分隔各个部分:

```java
static final int MAX_WIDTH = 999;
```

注意**任何`final`字段必须在构造函数完成**之前初始化。

对于`static final`字段，这意味着我们可以初始化它们:

*   如上例所示的声明
*   在静态初始化程序块中

例如`final`字段，这意味着我们可以初始化它们:

*   申报时
*   在实例初始化程序块中
*   在构造函数中

否则，编译器会给我们一个错误。

### 4.4。`Final`争论

`final`关键字放在方法参数之前也是合法的。**一个`final`参数不能在一个**方法中被改变:

```java
public void methodWithFinalArguments(final int x) {
    x=1;
}
```

上述赋值导致编译器错误:

```java
The final local variable x cannot be assigned. It must be blank and not using a compound assignment
```

## 5。结论

在本文中，我们学习了`final`关键字对类、方法和变量的意义。虽然我们可能不会在内部代码中经常使用`final`关键字，但它可能是一个很好的设计解决方案。

一如既往，本文的完整代码可以在 [GitHub 项目](https://web.archive.org/web/20221127024826/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-oop-modifiers)中找到。