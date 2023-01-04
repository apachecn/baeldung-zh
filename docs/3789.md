# 在调用超类型构造函数之前，不能引用“X”

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-cannot-reference-x-before-supertype-constructor-error>

## 1。概述

在这个简短的教程中，我们将展示如何获得错误`Cannot reference “X” before supertype constructor has been called,`以及如何避免它。

## 2。构造函数链

一个构造函数只能调用一个其他构造函数。该调用必须位于其正文的第一行。

我们可以用关键字`this`调用同一个类的构造函数，也可以用关键字`super`调用超类的构造函数。

当一个构造函数不调用另一个构造函数时，编译器会添加一个对超类的无参数构造函数的调用。

## 3。我们的编译错误

这个错误归结为**在我们调用构造函数链之前试图访问实例级成员。**

让我们看看可能遇到这种情况的几种方式。

### 3.1.引用实例方法

在下一个例子中，我们将在第 5 行看到编译错误`**Cannot reference “X” before supertype constructor has been called**`。请注意，构造函数试图过早使用实例方法`getErrorCode()`:

```
public class MyException extends RuntimeException {
    private int errorCode = 0;

    public MyException(String message) {
        super(message + getErrorCode()); // compilation error
    }

    public int getErrorCode() {
        return errorCode;
    }
} 
```

这个错误是因为，**u**直到`super()`完成，没有类`MyException`的实例。因此，我们还不能调用实例方法`getErrorCode()`。

### 3.2.引用实例字段

在下一个例子中，我们看到了带有实例字段而不是实例方法的异常。让我们看看**第一个构造函数如何在实例本身准备好之前尝试使用实例成员:**

```
public class MyClass {

    private int myField1 = 10;
    private int myField2;

    public MyClass() {
        this(myField1); // compilation error
    }

    public MyClass(int i) {
        myField2 = i;
    }
}
```

对实例字段的引用只能在它的类被初始化之后进行，也就是说在对`this()`或`super()`的任何调用之后。

那么，为什么第二个构造函数也使用了实例字段，却没有出现编译错误呢？

记住**所有的类都是从类`Object`** 隐式派生的，所以编译器添加了一个隐式的`super() `调用:

```
public MyClass(int i) {
    super(); // added by compiler
    myField2 = i;
} 
```

在这里，`Object`的构造函数在我们访问`myField2`之前被调用，这意味着我们没事。

## 4。解决方案

这个问题的第一个可能的解决方案很简单:**我们不调用第二个构造函数。我们在第一个构造函数中明确地做了我们想在第二个构造函数中做的事情。**

在这种情况下，我们将把`myField1` 的值复制到`myField2`中:

```
public class MyClass {

    private int myField1 = 10;
    private int myField2;

    public MyClass() {
        myField2 = myField1;
    }

    public MyClass(int i) {
        myField2 = i;
    }
} 
```

然而，总的来说，我们可能需要重新思考我们正在建造的结构。

但是，如果我们调用第二个构造函数是有原因的，例如，为了避免重复代码，**我们可以将代码移到方法:**

```
public class MyClass {

    private int myField1 = 10;
    private int myField2;

    public MyClass() {
        setupMyFields(myField1);
    }

    public MyClass(int i) {
        setupMyFields(i);
    }

    private void setupMyFields(int i) {
        myField2 = i;
    }
} 
```

同样，这是可行的，因为编译器在调用方法之前已经隐式调用了构造函数链。

第三个解决方案可能是我们使用静态字段或方法 T2。如果我们将`myField1` 改为静态常数，那么编译器也很高兴:

```
public class MyClass {

    private static final int SOME_CONSTANT = 10;
    private int myField2;

    public MyClass() {
        this(SOME_CONSTANT);
    }

    public MyClass(int i) {
        myField2 = i;
    }
} 
```

我们应该注意，创建一个字段`static`意味着它将被该对象的所有实例共享，因此这不是一个可以轻易更改的更改。

为了得到正确的答案，我们需要一个强有力的理由。例如，也许该值实际上不是一个字段，而是一个常数，因此将其设为`static`和`final`是有意义的。可能我们要调用的构造方法不需要访问类的实例成员，意思应该是`static`。

## 5。结论

在本文中，我们看到了在调用`super()`或`this()` 之前引用实例成员是如何导致编译错误的。我们在显式声明的基类和隐式的`Object`基类中都看到了这种情况。

我们还演示了这是构造函数设计的一个问题，并展示了如何通过在构造函数中重复代码、委托给构造后设置方法，或者使用常量值或静态方法来帮助构造来解决这个问题。

一如既往，这个例子的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221205110052/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-oop-constructors)