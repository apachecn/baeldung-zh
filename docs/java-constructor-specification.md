# Java 中的构造函数规范

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-constructor-specification>

## 1.概观

在本教程中，我们将学习 Java 如何处理[构造函数](/web/20220528124242/https://www.baeldung.com/java-constructors)，并回顾来自 [Java 语言规范](https://web.archive.org/web/20220528124242/https://docs.oracle.com/javase/specs/)的一些与它们相关的规则。

## 2.构造函数声明

在 Java 中，每个类都必须有一个构造函数。它的结构看起来类似于一个方法，但是它有不同的用途。

让我们看看构造函数的规范:

```java
<Constructor Modifiers> <Constructor Declarator> [Throws Clause] <Constructor Body>
```

让我们分开来看每一件作品。

### 2.1.构造函数修饰符

构造函数声明以访问修饰符开始:根据其他访问修饰符，它们可以是`public`、`private`、`protected`或包访问。

**为了防止编译错误，构造函数声明不能有一个以上的`private`、`protected`或`public`访问修饰符。**

与方法不同，构造函数不能是`abstract`、`static`、`final`、本机或`synchronized`:

*   没有必要声明构造函数`final`，因为它们不是类成员，也不继承。
*   抽象是不必要的，因为我们必须实现构造函数。
*   不需要静态构造函数，因为每个构造函数都是用对象调用的。
*   一个正在构造的对象不应该被`synchronized`锁定，因为它会在构造的时候锁定这个对象，通常在所有的构造函数完成它们的工作之前，这个对象是不能被其他线程使用的。
*   Java 中没有`native` `constructors`，因为这是一个语言设计决策，旨在确保超类构造函数在对象创建期间总是被调用。

### 2.2.构造函数声明符

让我们检查一下构造函数声明符的语法:

```java
Constrcutor Name (Parameter List)
```

声明符中的构造函数名和包含构造函数声明的类名之间必须匹配，否则将发生编译时错误。

### 2.3.抛出条款

方法和构造函数的 [`throws`](/web/20220528124242/https://www.baeldung.com/java-throw-throws#throws-in-java) 子句的结构和行为都是相同的。

### 2.4.构造体

构造函数体的语法是:

```java
Constructor Body: { [Explicit Constructor Invocation] [Block Statements] }
```

我们可以显式调用同一个类的另一个构造函数或直接超类作为构造函数体中的第一个命令。不允许直接或间接调用同一个构造函数。

## 3.显式构造函数调用

我们可以将构造函数的调用分为两种类型:

*   可选的构造函数调用以关键字`this`开始。它们用于调用同一个类的备用构造函数。
*   超类构造函数调用以关键字`super.`开始

让我们看一个如何使用`this`和`super`关键字调用另一个构造函数的例子:

```java
class Person {
    String name;

    public Person() {
        this("Arash");   //ExplicitConstructorInvocation
    }

    public Person(String name){
        this.name = name;
    }
}
```

这里，`Employee`的第一个构造函数调用其超类`Person`的构造函数，传递 id:

```java
class Person {
    int id;
    public Person(int id) {
        this.id = id;
    }
}

class Employee extends Person {
    String name;
    public Employee(int id) {
        super(id);
    }
    public Employee(int id, String name) {
        super(id);
        this.name = name;
    }
}
```

## 4.构造函数调用规则

### 4.1.`this`或`super`必须是构造函数中的第一条语句

每当我们调用一个构造函数时，它必须调用其基类的构造函数。此外，您可以在类中调用另一个构造函数。 **Java 通过在构造函数中第一次调用 be 到`this`或`super`来执行这个规则。**

让我们来看一个例子:

```java
class Person {
    Person() {
        //
    }
}
class Employee extends Person {
    Employee() {
        // 
    }
}
```

下面是一个构造函数编译的例子:

```java
.class Employee
.super Person
; A constructor taking no arguments
.method <init>()V
aload_0
invokespecial Person/<init>()V
return
.end method
```

除了生成的方法名为`<init>.`之外，构造函数编译类似于编译任何其他方法。验证`<init>`方法的一个要求是，调用超类构造函数(或当前类中的其他构造函数)必须是方法的第一步。

正如我们在上面看到的，`Person`类必须调用它的超类构造函数，依此类推，直到`java.lang.Object.`

当类必须调用它们的超类构造函数时，它确保了在没有正确初始化的情况下它们永远不会被使用。JVM 的安全性依赖于此，因为一些方法在类初始化之前不会工作。

### 4.2.不要在构造函数中同时使用`this`和`super`

想象一下，如果我们可以在构造函数体中一起使用`this`和`super`。

让我们通过一个例子来看看会发生什么:

```java
class Person {
    String name;
    public Person() {
        this("Arash");
    }

    public Person(String name) {
        this.name = name;
    }
}

class Employee extends Person {
    int id;
    public Employee() {
        super();
    }

    public Employee(String name) {
        super(name);
    }

    public Employee(int id) {
        this();
        super("John"); // syntax error
        this.id = id;
    }

    public static void main(String[] args) {
        new Employee(100);
    }
}
```

我们不能执行上面的代码，因为**会出现编译时错误**。当然，Java 编译器有其合理的解释。

让我们来看看构造函数调用序列:

[![constructor-2](img/51b5e4ccb827d15896f21ed3afe8ad68.png)](/web/20220528124242/https://www.baeldung.com/wp-content/uploads/2022/01/uu-2.png)

Java 编译器不允许编译这个程序，因为初始化不明确。

### 4.3.递归构造函数调用

如果构造函数调用自身，编译器将引发错误。例如，在下面的 Java 代码中，编译器会抛出一个错误，因为我们试图在构造函数中调用同一个构造函数:

```java
public class RecursiveConstructorInvocation {
    public RecursiveConstructorInvocation() {
        this();
    }
}
```

尽管有 Java 编译器的限制，我们可以通过稍微修改代码来编译程序，但是我们会遇到这样的堆栈溢出:

```java
public class RecursiveConstructorInvocation {
    public RecursiveConstructorInvocation() {
        RecursiveConstructorInvocation rci = new RecursiveConstructorInvocation();
    }

    public static void main(String[] args) {
        new RecursiveConstructorInvocation();
    }
}
```

**我们创建了一个`RecursiveConstructorInvocation`对象，通过调用构造函数来初始化。然后，构造函数创建另一个`RecursiveConstructorInvocation`对象，该对象通过再次调用构造函数进行初始化，直到堆栈溢出。**

现在，让我们看看输出:

```java
Exception in thread "main" java.lang.StackOverflowError
	at org.example.RecursiveConstructorInvocation.<init>(RecursiveConstructorInvocation.java:29)
	at org.example.RecursiveConstructorInvocation.<init>(RecursiveConstructorInvocation.java:29)
	at org.example.RecursiveConstructorInvocation.<init>(RecursiveConstructorInvocation.java:29)
//...
```

## 5.结论

在本教程中，我们讨论了 Java 中构造函数的规范，并回顾了理解类和超类中构造函数调用的一些规则。

和往常一样，代码样本可以在 GitHub 上找到[。](https://web.archive.org/web/20220528124242/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-oop-constructors)