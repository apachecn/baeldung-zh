# 在构造函数中引发异常

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-constructors-exceptions>

## 1.概观

**[异常](/web/20220628120724/https://www.baeldung.com/java-common-exceptions)将错误处理代码从应用程序的正常流程中分离出来。**在对象的实例化过程中抛出异常并不少见。

在本文中，我们将研究关于在构造函数中抛出异常的所有细节。

## 2.在构造函数中引发异常

**[构造函数](/web/20220628120724/https://www.baeldung.com/java-constructors)是被调用来创建对象的特殊类型的方法。**在接下来的部分中，我们将研究如何抛出异常，抛出哪些异常，以及为什么我们会在构造函数中抛出异常。

### 2.1.怎么会？

在构造函数中抛出异常与在任何其他方法中抛出异常没有什么不同。让我们从创建一个带有无参数构造函数的`Animal`类开始:

```java
public Animal() throws InstantiationException {
    throw new InstantiationException("Cannot be instantiated");
}
```

这里，我们抛出`InstantiationException`，它是一个[检查过的异常](/web/20220628120724/https://www.baeldung.com/java-checked-unchecked-exceptions)。

### 2.2.哪些？

尽管抛出任何类型的异常都是允许的，但是让我们建立一些最佳实践。

第一，我们不想扔“`java.lang.Exception”`”。这是因为调用者不可能识别出哪种异常，从而处理它。

第二，如果调用者必须强制处理异常，我们应该抛出一个检查过的异常。

第三，如果调用者不能从异常中恢复，我们应该抛出一个未检查的异常。

值得注意的是**这些实践同样适用于方法和构造函数**。

### 2.3.为什么？

在这一节中，让我们理解为什么我们可能想要在构造函数中抛出异常。

参数验证是在构造函数中抛出异常的常见用例。构造函数多用于给变量赋值。如果传递给构造函数的参数无效，我们可以抛出异常。让我们考虑一个简单的例子:

```java
public Animal(String id, int age) {
    if (id == null)
        throw new NullPointerException("Id cannot be null");
    if (age < 0)
        throw new IllegalArgumentException("Age cannot be negative");
} 
```

在上面的例子中，我们在初始化对象之前执行参数验证。这有助于确保我们只创建有效的对象。

这里，如果传递给`Animal`对象的`id`是`null`，对于非空但仍然无效的参数，我们可以抛出`NullPointerException` ，比如对于`age`的负值，我们可以抛出一个`IllegalArgumentException`。

安全检查是在构造函数中抛出异常的另一个常见用例。有些对象在创建过程中需要安全检查。如果构造函数执行了可能不安全或敏感的操作，我们可以抛出异常。

让我们考虑我们的`Animal`类从用户输入文件中加载属性:

```java
public Animal(File file) throws SecurityException, IOException {
    if (file.isAbsolute()) {
        throw new SecurityException("Traversal attempt");
    }
    if (!file.getCanonicalPath()
        .equals(file.getAbsolutePath())) {
        throw new SecurityException("Traversal attempt");
    }
} 
```

在上面的例子中，我们阻止了[路径遍历攻击](https://web.archive.org/web/20220628120724/https://owasp.org/www-community/attacks/Path_Traversal)。这是通过不允许绝对路径和目录遍历实现的。例如，考虑文件“a/../b.txt”。这里，规范路径和绝对路径是不同的，这可能是一种潜在的目录遍历攻击。

## 3.构造函数中的继承异常

现在，让我们谈谈在构造函数中处理超类异常。

让我们创建一个子类`Bird`，它扩展了我们的`Animal`类:

```java
public class Bird extends Animal {
    public Bird() throws ReflectiveOperationException {
        super();
    }
    public Bird(String id, int age) {
        super(id, age);
    }
}
```

由于`super()`必须是构造函数的第一行，我们不能简单地插入一个`try-catch`块来处理超类抛出的检查异常。

由于我们的父类`Animal`抛出了检查过的异常`InstantiationException`，我们不能在`Bird`构造函数中处理该异常。**相反，我们可以传播相同的异常或其父异常。**

需要注意的是，关于方法重写的异常处理规则是不同的。在方法重写中，如果超类方法声明异常，子类重写的方法可以声明相同的异常、子类异常或无异常，但不能声明父异常。

另一方面，未检查的异常不需要声明，也不能在子类构造函数中处理。

## 4.安全问题

在构造函数中引发异常会导致对象部分初始化。如 [Java 安全编码指南](https://web.archive.org/web/20220628120724/https://www.oracle.com/java/technologies/javase/seccodeguide.html)中的指南 7.3 所述，非 final 类的部分初始化对象容易受到安全问题，即终结器攻击。

简而言之，终结器攻击是通过对部分初始化的对象进行子类化并覆盖其`finalize()`方法来引发的，并试图创建该子类的新实例。这可能会绕过在子类的构造函数内部完成的安全检查。

重写`finalize()`方法并将其标记为`final`可以防止这种攻击。

然而，在 Java 9 中不赞成使用`finalize()`方法，这样可以防止这种类型的攻击。

## 5.结论

在本教程中，我们学习了在构造函数中抛出异常，以及相关的好处和安全问题。此外，我们还看了一些在构造函数中抛出异常的最佳实践。

和往常一样，本教程中使用的源代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220628120724/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-oop-constructors)