# Java 异常面试问题(+答案)

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-exceptions-interview-questions>

[This article is part of a series:](javascript:void(0);)[• Java Collections Interview Questions](/web/20221208143855/https://www.baeldung.com/java-collections-interview-questions)
[• Java Type System Interview Questions](/web/20221208143855/https://www.baeldung.com/java-type-system-interview-questions)
[• Java Concurrency Interview Questions (+ Answers)](/web/20221208143855/https://www.baeldung.com/java-concurrency-interview-questions)
[• Java Class Structure and Initialization Interview Questions](/web/20221208143855/https://www.baeldung.com/java-classes-initialization-questions)
[• Java 8 Interview Questions(+ Answers)](/web/20221208143855/https://www.baeldung.com/java-8-interview-questions)
[• Memory Management in Java Interview Questions (+Answers)](/web/20221208143855/https://www.baeldung.com/java-memory-management-interview-questions)
[• Java Generics Interview Questions (+Answers)](/web/20221208143855/https://www.baeldung.com/java-generics-interview-questions)
[• Java Flow Control Interview Questions (+ Answers)](/web/20221208143855/https://www.baeldung.com/java-flow-control-interview-questions)
• Java Exceptions Interview Questions (+ Answers) (current article)[• Java Annotations Interview Questions (+ Answers)](/web/20221208143855/https://www.baeldung.com/java-annotations-interview-questions)
[• Top Spring Framework Interview Questions](/web/20221208143855/https://www.baeldung.com/spring-interview-questions)

## 1。概述

异常是每个 Java 开发人员都应该熟悉的一个基本话题。这篇文章为面试中可能出现的一些问题提供了答案。

## 2。问题

### Q1。什么是例外？

异常是在程序执行过程中发生的异常事件，它扰乱了程序指令的正常流程。

### Q2。Throw 和 Throws 关键字的目的是什么？

关键字`throws`用于指定一个方法在执行过程中可能会引发异常。它在调用方法时强制执行显式异常处理:

```java
public void simpleMethod() throws Exception {
    // ...
}
```

`throw`关键字允许我们抛出一个异常对象来中断程序的正常流程。当程序不能满足给定条件时，这是最常用的:

```java
if (task.isTooComplicated()) {
    throw new TooComplicatedException("The task is too complicated");
}
```

### Q3。如何处理异常？

通过使用`try-catch-finally`语句:

```java
try {
    // ...
} catch (ExceptionType1 ex) {
    // ...
} catch (ExceptionType2 ex) {
    // ...
} finally {
    // ...
}
```

可能发生异常的代码块包含在`try`块中。这个代码块也被称为“受保护的”或“防护的”代码。

如果出现异常，则执行与抛出的异常相匹配的`catch`块，否则忽略所有`catch`块。

`finally`块总是在`try`块退出后执行，不管其中是否抛出了异常。

### Q4。如何捕捉多个异常？

有三种方法可以处理代码块中的多个异常。

第一种是使用一个能够处理所有抛出的异常类型的`catch`块:

```java
try {
    // ...
} catch (Exception ex) {
    // ...
}
```

您应该记住，推荐的做法是使用尽可能准确的异常处理程序。

过于宽泛的异常处理程序会使您的代码更容易出错，捕获没有预料到的异常，并导致程序中出现意外行为。

第二种方法是实现多个 catch 块:

```java
try {
    // ...
} catch (FileNotFoundException ex) {
    // ...
} catch (EOFException ex) {
    // ...
}
```

注意，如果异常有继承关系；子类型必须在前面，父类型在后面。如果我们做不到这一点，就会导致编译错误。

第三种是使用多重捕捉块:

```java
try {
    // ...
} catch (FileNotFoundException | EOFException ex) {
    // ...
}
```

这个特性最早是在 Java 7 中引入的；减少代码重复并使其更易于维护。

### Q5。已检查的异常和未检查的异常有什么区别？

被检查的异常必须在`try-catch`块中处理或在`throws`子句中声明；而未检查的异常既不需要处理也不需要声明。

检查异常和未检查异常也分别称为编译时异常和运行时异常。

除了由`Error`、`RuntimeException`及其子类指示的例外，所有例外都是检查过的例外。

### Q6。异常和错误的区别是什么？

异常是一个事件，它代表一种可能恢复的情况，而错误代表一种通常不可能恢复的外部情况。

JVM 抛出的所有错误都是`Error`或其子类之一的实例，更常见的错误包括但不限于:

*   `OutOfMemoryError`–当 JVM 由于内存不足而无法分配更多对象，并且垃圾收集器无法提供更多可用对象时抛出
*   `StackOverflowError`–当线程的堆栈空间耗尽时发生，通常是因为应用程序递归太深
*   `ExceptionInInitializerError`–表示在静态初始值设定项的评估过程中发生了意外异常
*   `NoClassDefFoundError`–当类加载器试图加载类的定义而找不到时抛出，通常是因为在类路径中找不到所需的`class`文件
*   `UnsupportedClassVersionError`–当 JVM 试图读取一个`class`文件并确定该文件中的版本不受支持时发生，通常是因为该文件是用较新版本的 Java 生成的

虽然可以用`try`语句处理错误，但这不是推荐的做法，因为不能保证程序在抛出错误后能够可靠地做任何事情。

### Q7。执行下面的代码块会抛出什么异常？

```java
Integer[][] ints = { { 1, 2, 3 }, { null }, { 7, 8, 9 } };
System.out.println("value = " + ints[1][1].intValue());
```

它抛出一个`ArrayIndexOutOfBoundsException` ，因为我们试图访问一个大于数组长度的位置。

### Q8。什么是异常链？

当引发一个异常以响应另一个异常时发生。这允许我们发现我们提出的问题的完整历史:

```java
try {
    task.readConfigFile();
} catch (FileNotFoundException ex) {
    throw new TaskException("Could not perform task", ex);
}
```

### Q9。什么是堆栈跟踪，它与异常有什么关系？

堆栈跟踪提供被调用的类和方法的名称，从应用程序开始到发生异常。

这是一个非常有用的调试工具，因为它使我们能够准确地确定异常是在应用程序的什么地方抛出的，以及导致异常的最初原因。

### Q10。为什么你想要子类化一个异常？

如果 Java 平台中已经存在的异常类型不能表示异常类型，或者如果您需要向客户端代码提供更多信息以便以更精确的方式处理异常，那么您应该创建一个自定义异常。

决定是否应该检查一个自定义异常完全取决于业务案例。然而，根据经验法则；如果使用您的异常的代码可以从中恢复，那么创建一个检查的异常，否则使它不被检查。

此外，您应该从最具体的`Exception`子类继承，这个子类与您想要抛出的那个子类密切相关。如果没有这样的类，那么选择`Exception` 作为父类。

### Q11。异常的一些优点是什么？

传统的错误检测和处理技术经常导致代码难以维护和阅读。然而，异常使我们能够将应用程序的核心逻辑与意外情况发生时该做什么的细节分开。

此外，由于 JVM 通过调用堆栈向后搜索，寻找对处理特定异常感兴趣的任何方法；我们获得了在调用堆栈中向上传播错误的能力，而无需编写额外的代码。

此外，因为程序中抛出的所有异常都是对象，所以可以根据类的层次结构对它们进行分组或分类。这允许我们通过在`catch`块中指定异常的超类，在单个异常处理程序中捕获一组异常。

### Q12。你能在 Lambda 表达式中抛出异常吗？

当使用 Java 已经提供的标准函数接口时，您只能抛出未检查的异常，因为标准函数接口在方法签名中没有“throws”子句:

```java
List<Integer> integers = Arrays.asList(3, 9, 7, 0, 10, 20);
integers.forEach(i -> {
    if (i == 0) {
        throw new IllegalArgumentException("Zero not allowed");
    }
    System.out.println(Math.PI / i);
});
```

但是，如果您使用自定义函数接口，则可能会引发检查异常:

```java
@FunctionalInterface
public static interface CheckedFunction<T> {
    void apply(T t) throws Exception;
}
```

```java
public void processTasks(
  List<Task> taks, CheckedFunction<Task> checkedFunction) {
    for (Task task : taks) {
        try {
            checkedFunction.apply(task);
        } catch (Exception e) {
            // ...
        }
    }
}

processTasks(taskList, t -> {
    // ...
    throw new Exception("Something happened");
});
```

### Q13。当重写一个抛出异常的方法时，我们需要遵循什么规则？

几条规则规定了在继承的上下文中必须如何声明异常。

当父类方法不抛出任何异常时，子类方法不能抛出任何选中的异常，但可以抛出任何未选中的异常。

下面是演示这一点的示例代码:

```java
class Parent {
    void doSomething() {
        // ...
    }
}

class Child extends Parent {
    void doSomething() throws IllegalArgumentException {
        // ...
    }
}
```

下一个范例将无法编译，因为覆写方法会掷回未在覆写方法中宣告的已检查例外状况:

```java
class Parent {
    void doSomething() {
        // ...
    }
}

class Child extends Parent {
    void doSomething() throws IOException {
        // Compilation error
    }
}
```

当父类方法抛出一个或多个已检查的异常时，子类方法可以抛出任何未检查的异常；声明的检查异常的全部、无或子集，甚至更多，只要它们具有相同或更窄的范围。

下面是一个成功遵循上述规则的示例代码:

```java
class Parent {
    void doSomething() throws IOException, ParseException {
        // ...
    }

    void doSomethingElse() throws IOException {
        // ...
    }
}

class Child extends Parent {
    void doSomething() throws IOException {
        // ...
    }

    void doSomethingElse() throws FileNotFoundException, EOFException {
        // ...
    }
}
```

请注意，这两种方法都遵守规则。第一个方法抛出的异常比被重写的方法少，而第二个方法抛出的异常更多；它们的范围更窄。

然而，如果我们试图抛出一个父类方法没有声明的检查异常，或者抛出一个范围更广的异常；我们将得到一个编译错误:

```java
class Parent {
    void doSomething() throws FileNotFoundException {
        // ...
    }
}

class Child extends Parent {
    void doSomething() throws IOException {
        // Compilation error
    }
}
```

当父类方法有一个带有未检查异常的 throws 子句时，子类方法可以不抛出任何异常或抛出任意数量的未检查异常，即使它们不相关。

这里有一个遵守规则的例子:

```java
class Parent {
    void doSomething() throws IllegalArgumentException {
        // ...
    }
}

class Child extends Parent {
    void doSomething()
      throws ArithmeticException, BufferOverflowException {
        // ...
    }
}
```

### Q14。下面的代码会编译吗？

```java
void doSomething() {
    // ...
    throw new RuntimeException(new Exception("Chained Exception"));
}
```

是的。当链接异常时，编译器只关心链中的第一个异常，因为它检测未检查的异常，所以我们不需要添加 throws 子句。

### Q15。有没有办法从一个没有 Throws 子句的方法中抛出一个检查过的异常？

是的。我们可以利用编译器执行的类型擦除，让它认为我们正在抛出一个未经检查的异常，而实际上；我们正在抛出一个检查异常:

```java
public <T extends Throwable> T sneakyThrow(Throwable ex) throws T {
    throw (T) ex;
}

public void methodWithoutThrows() {
    this.<RuntimeException>sneakyThrow(new Exception("Checked Exception"));
}
```

## 3。结论

在本文中，我们探讨了一些在 Java 开发人员的技术面试中可能出现的关于异常的问题。这不是一个详尽的列表，它只应被视为进一步研究的开始。

我们 Baeldung 祝您在即将到来的面试中取得成功。

Next **»**[Java Annotations Interview Questions (+ Answers)](/web/20221208143855/https://www.baeldung.com/java-annotations-interview-questions)**«** Previous[Java Flow Control Interview Questions (+ Answers)](/web/20221208143855/https://www.baeldung.com/java-flow-control-interview-questions)