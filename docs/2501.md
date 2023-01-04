# Java“受保护的”访问修饰符

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-protected-access-modifier>

## 1.概观

在 Java 编程语言中，可以用访问修饰符标记字段、构造函数、方法和类。**在本教程中，我们将了解`protected` access。**

## 2.`protected`关键字

[声明为`private`的元素只能被声明它们的类访问，](/web/20221205222713/https://www.baeldung.com/java-private-keyword#private-modifier)`protected`关键字允许来自子类和同一个包的成员的访问。

通过使用`protected`关键字，我们决定哪些方法和字段应该被认为是包或类层次结构的内部，哪些应该被暴露给外部代码。

## 3.声明`protected`字段、方法和构造函数

首先，让我们创建一个名为 *FirstClass* 的类，包含一个`protected`字段、方法和构造函数:

```
public class FirstClass {

    protected String name;

    protected FirstClass(String name) {
        this.name = name;
    }

    protected String getName() {
        return name;
    }
}
```

在这个例子中，通过使用`protected`关键字，我们已经向与 *FirstClass* 在同一个包中的类以及`FirstClass`的子类授予了对这些字段的访问权。

## 4.访问`protected`字段、方法和构造函数

### 4.1。来自同一个包

现在，让我们看看如何通过创建一个新的*泛型类*来访问`protected`字段，该类在与`FirstClass`相同的包中声明:

```
public class GenericClass {

    public static void main(String[] args) {
        FirstClass first = new FirstClass("random name");
        System.out.println("FirstClass name is " + first.getName());
        first.name = "new name";
    }
}
```

由于这个调用类与`FirstClass,`在同一个包中，所以它可以看到所有的`protected`字段、方法和构造函数并与之交互。

### 4.2.来自不同的包装

现在让我们尝试与这些字段进行交互，这些字段来自一个不同于`FirstClass`的包中声明的类:

```
public class SecondGenericClass {

    public static void main(String[] args) {
        FirstClass first = new FirstClass("random name");
        System.out.println("FirstClass name is "+ first.getName());
        first.name = "new name";
    }
}
```

正如我们所见，**我们得到编译错误**:

```
The constructor FirstClass(String) is not visible
The method getName() from the type FirstClass is not visible
The field FirstClass.name is not visible
```

这正是我们使用`protected`关键字所期待的。这是因为`SecondGenericClass`和`FirstClass`不在同一个包中，也没有子类化它。

### 4.3。来自子类

现在让我们看看当我们声明**是一个扩展`FirstClass `的类，但是在不同的包**中声明时会发生什么:

```
public class SecondClass extends FirstClass {

    public SecondClass(String name) {
        super(name);
        System.out.println("SecondClass name is " + this.getName());
        this.name = "new name";
    } 
}
```

正如所料，我们可以访问所有受保护的字段、方法和构造函数。这是因为`SecondClass`是`FirstClass`的子类。

## 5.`protected`内部阶层

在前面的例子中，我们看到了`protected`字段、方法和构造函数的作用。还有一个更特殊的情况——`protected`内部类。

让我们在第一个类中创建这个空的内部类:

```
package com.baeldung.core.modifiers;

public class FirstClass {

    // ...

    protected static class InnerClass {

    }
}
```

正如我们所见，这是一个静态内部类，因此可以从`FirstClass`的实例外部构建。然而，由于它是`protected`，**，我们只能从与*一级*，**相同的包中的代码实例化它。

### 5.1。来自同一个包

为了测试这一点，让我们编辑我们的*泛型类*:

```
public class GenericClass {

    public static void main(String[] args) {
        // ...
        FirstClass.InnerClass innerClass = new FirstClass.InnerClass();
    }
}
```

正如我们所见，我们可以毫无问题地实例化`InnerClass`，因为`GenericClass` 和 `FirstClass`在同一个包中。

### 5.2.来自不同的包装

让我们试着从我们的`SecondGenericClass`中实例化一个`InnerClass`，我们记得，它在`FirstClass'` 包之外:

```
public class SecondGenericClass {

    public static void main(String[] args) {
        // ...

        FirstClass.InnerClass innerClass = new FirstClass.InnerClass();
    }
}
```

不出所料，**我们得到一个编译错误**:

```
The type FirstClass.InnerClass is not visible
```

### 5.3.来自子类

让我们试着从`SecondClass`开始做同样的事情:

```
public class SecondClass extends FirstClass {

    public SecondClass(String name) {
        // ...

        FirstClass.InnerClass innerClass = new FirstClass.InnerClass();
    }     
}
```

**我们期望轻松地实例化我们的`InnerClass`。然而，我们在这里也得到了一个编译错误:**

```
The constructor FirstClass.InnerClass() is not visible
```

让我们来看看我们的`InnerClass`宣言:

```
protected static class InnerClass {
}
```

我们得到这个错误的主要原因是`protected`类的默认构造函数**是隐式的** `**protected**.` 另外，`**SecondClass**` **是 FirstClass 的子类，但不是`InnerClass`** 的子类。最后，**我们还声明** **`SecondClass`外`FirstClass'`包**。

由于这些原因，`SecondClass`不能访问`protected` `InnerClass`构造函数。

如果我们希望**解决这个问题**并允许我们的`SecondClass`实例化一个`InnerClass`对象，**我们可以显式声明一个公共构造函数**:

```
protected static class InnerClass {
    public InnerClass() {
    }
}
```

通过这样做，我们不再得到编译错误，我们现在可以从`SecondClass`实例化一个`InnerClass`。

## 6.结论

在这个快速教程中，我们讨论了 Java 中的`protected`访问修饰符。有了它，我们可以确保只向同一个包中的子类和类公开所需的数据和方法。

与往常一样，GitHub 上的示例代码[可用。](https://web.archive.org/web/20221205222713/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-syntax-2)