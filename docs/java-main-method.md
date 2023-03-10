# Java main()方法解释

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-main-method>

## 1。概述

Every program needs a place to start its execution; talking about Java programs, that's the `main` method.We’re so used to writing the `main` method during our code sessions, that we don’t even pay attention to its details. In this quick article, we'll analyze this method and show some other ways of writing it.

## 2。普通签名

The most common main method template is:

```java
public static void main(String[] args) { }
```

这就是我们学习的方式，这就是 IDE 自动完成代码的方式。但是这并不是这个方法可以采用的唯一形式，**我们可以使用一些有效的变体**，并且不是每个开发人员都注意到这个事实。

在我们深入研究这些方法签名之前，让我们回顾一下公共签名的每个关键字的含义:

*   `public`–访问修饰符，意思是全局可见性
*   方法可以直接从类中访问，我们不需要实例化一个对象来引用和使用它
*   `void`–表示该方法不返回值
*   `main`–方法的名称，这是 JVM 在执行 Java 程序时寻找的标识符

至于`args`参数，表示方法接收到的值。当我们第一次启动程序时，我们就是这样把参数传递给程序的。

参数`args`是一个由`String`组成的数组。在下面的例子中:

```java
java CommonMainMethodSignature foo bar
```

我们正在执行一个名为`CommonMainMethodSignature`的 Java 程序，并传递两个参数:`foo`和`bar`。这些值可以在`main`方法中作为`args[0]`(值为`foo`)和`args[1]`(值为`bar`)来访问。

在下一个示例中，我们将检查参数以决定是加载测试参数还是生产参数:

```java
public static void main(String[] args) {
    if (args.length > 0) {
        if (args[0].equals("test")) {
            // load test parameters
        } else if (args[0].equals("production")) {
            // load production parameters
        }
    }
}
```

记住 ide 也可以传递参数给程序，这总是好的。

## 3。不同的方法写一个`main()`方法

让我们检查一些不同的方法来编写`main`方法。虽然它们不是很常见，但它们是有效的签名。

注意，这些都不是特定于`main`方法的，它们可以用于任何 Java 方法，但是它们也是`main`方法的有效部分。

方括号可以放在`String`附近，如在通用模板中，或者放在`args`附近的任一侧:

```java
public static void main(String []args) { } 
```

```java
public static void main(String args[]) { }
```

参数可以表示为 varargs:

```java
public static void main(String...args) { }
```

我们甚至可以为`main()`方法添加`strictfp`，该方法用于处理浮点值时处理器之间的兼容性:

```java
public strictfp static void main(String[] args) { }
```

`synchronized`和`final`也是`main`方法的有效关键字，但是它们在这里不起作用。

另一方面，`final`可以应用于`args`以防止数组被修改:

```java
public static void main(final String[] args) { }
```

为了结束这些例子，我们还可以用上述所有关键字编写`main`方法(当然，在实际应用程序中您可能不会用到这些关键字):

```java
final static synchronized strictfp void main(final String[] args) { }
```

## 4.拥有一个以上的`main()`方法

我们还可以在应用程序中定义不止一个`main`方法。

事实上，有些人把它作为验证单个类的原始测试技术(尽管像`JUnit`这样的测试框架更适合这个活动)。

为了指定 JVM 应该执行哪个`main`方法作为我们应用程序的入口点，我们使用了`MANIFEST.MF`文件。在清单中，我们可以指出主要的类:

```java
Main-Class: mypackage.ClassWithMainMethod
```

这主要在创建可执行文件`.jar`时使用。我们通过位于`META-INF/MANIFEST.MF`的清单文件(以 UTF-8 编码)来指示哪个类具有开始执行的`main`方法。

## 5。结论

本教程描述了`main`方法的细节以及它可以采用的一些其他形式，甚至是那些对大多数开发人员来说不太常见的形式。

请记住，**虽然我们展示的所有例子在语法上都是有效的，但它们只是为了教育目的**，大多数时候我们会坚持使用共同的签名来完成我们的工作。