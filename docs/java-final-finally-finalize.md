# Java 中 Final、Finally 和 Finalize 的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-final-finally-finalize>

## 1.概观

在本教程中，我们将概述三个 Java 关键字:`final, finally `和`finalize.`

虽然这些关键字彼此相似，但在 Java 中却有着非常不同的含义。我们将了解它们各自的用途，并通过一些代码查看一些示例。

## 2.`final`关键字

我们先来看看`final`这个关键词，在哪里用，为什么用。**我们可以将`final`关键字应用于类、方法、字段、变量和方法参数声明。**

**虽然对每个人的影响不同**:

*   创建一个类`final`意味着不可能扩展这个类
*   将`final`添加到一个方法意味着不可能覆盖那个方法
*   最后，将`final`放在字段、变量或参数的前面意味着一旦引用被赋值，它就不能被改变(然而，如果引用是对一个可变对象的，那么它的内部状态可能会改变，尽管它是最终的)

关于 `final`关键词的详细文章可以在[这里](/web/20221129014508/https://www.baeldung.com/java-final)找到。

让我们通过一些例子来看看`final`关键字是如何工作的。

### 2.1.`final`字段、参数和变量

让我们创建一个具有两个`int `字段的`Parent`类，一个`final`字段和一个常规的非最终字段:

```java
public class Parent {

    int field1 = 1;
    final int field2 = 2;

    Parent() {
        field1 = 2; // OK
        field2 = 3; // Compilation error
    }

}
```

正如我们所见，编译器禁止我们给`field2`赋值。

现在让我们添加一个带有常规参数和最后一个参数的方法:

```java
 void method1(int arg1, final int arg2) {
        arg1 = 2; // OK
        arg2 = 3; // Compilation error
    }
```

类似于字段，不可能给`arg2`赋值，因为它被声明为 final。

我们现在可以添加第二个方法来说明如何使用局部变量:

```java
 void method2() {
        final int localVar = 2; // OK
        localVar = 3; // Compilation error
    }
```

没有令人惊讶的事情发生，编译器不允许我们在第一次赋值后给`localVar`赋值。

### 2.2.`final`方法

现在假设我们将`method2`设为 final 并创建`Parent`的一个子类，比如说`Child`，我们试图覆盖它的两个超类方法:

```java
public class Child extends Parent {

    @Override
    void method1(int arg1, int arg2) {
        // OK
    }

    @Override
    final void method2() {
        // Compilation error
    }

}
```

正如我们所看到的，覆盖`method1()`没有问题，但是当我们试图覆盖`method2()`时，我们得到了一个编译错误。

### 2.3.`final`阶级

最后，让我们将`Child`类设为 final，并尝试创建它的子类`GrandChild`:

```java
public final class Child extends Parent { 
    // ... 
}
```

```java
public class GrandChild extends Child {
    // Compilation error
}
```

编译器再次报错。`Child`类是最终的，因此不可能扩展。

## 3.`finally`块

`finally`块是与`try/catch`语句一起使用的可选块。**在这个块中，我们包含了在`try/catch `结构之后执行的代码，不管是否抛出了异常**。

如果我们包括一个`finally` 块，它甚至可以和没有任何`catch`块的`try`块一起使用。代码将在`try`之后或抛出异常之后执行。

我们有一篇关于 Java 异常处理的深入文章[在这里](/web/20221129014508/https://www.baeldung.com/java-exceptions)。

现在让我们在一个简短的例子中演示一个`finally`块。我们将创建一个带有`try/catch/finally`结构的虚拟`main()`方法:

```java
public static void main(String args[]) {
    try {
        System.out.println("Execute try block");
        throw new Exception();
    } catch (Exception e) {
        System.out.println("Execute catch block");
    } finally {
        System.out.println("Execute finally block");
    }
}
```

如果我们运行这段代码，它将输出以下内容:

```java
Execute try block
Execute catch block
Execute finally block
```

现在让我们通过移除 catch 块来修改该方法(并将`throws Exception`添加到签名中):

```java
public static void main(String args[]) throws Exception {
    try {
        System.out.println("Execute try block");
        throw new Exception();
    } finally {
        System.out.println("Execute finally block");
    }
}
```

现在的输出是:

```java
Execute try block
Execute finally block
```

如果我们现在删除`throw new Exception()`指令，我们可以观察到输出保持不变。我们的`finally`块执行每次都发生。

## 4.`finalize`方法

最后，`finalize`方法是一个受保护的方法，在[对象类](https://web.archive.org/web/20221129014508/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Object.html#finalize())中定义。**它由`garbage collector`在不再被引用并且已经被选择进行垃圾收集的对象上调用**。

像任何其他非最终方法一样，我们可以覆盖这个方法来定义一个对象在被`garbage collector`收集时必须具有的行为。

再说一次，关于`finalize`方法的详细文章可以在[这里](/web/20221129014508/https://www.baeldung.com/java-finalize)找到。

让我们来看一个它是如何工作的例子。我们将使用`System.gc()`来建议`JVM`触发`garbage collection`:

```java
 @Override
    protected void finalize() throws Throwable {
        System.out.println("Execute finalize method");
        super.finalize();
    }
```

```java
 public static void main(String[] args) throws Exception {
        FinalizeObject object = new FinalizeObject();
        object = null;
        System.gc();
        Thread.sleep(1000);
    }
```

在这个例子中，我们在我们的对象中覆盖了`finalize()`方法，并创建了一个`main()`方法，它实例化了我们的对象，并通过将创建的变量设置为`null`来立即删除引用。

之后，我们调用`System.gc()`来运行`garbage collector`(至少我们期望它运行)并等待一秒钟(只是为了确保在`garbage collector` 有机会调用`finalize()`方法之前`JVM`不会关闭)。

这段代码执行的输出应该是:

```java
Execute finalize method
```

注意，覆盖`finalize()`方法被认为是不好的做法，因为它的执行依赖于`garbage collection`，而后者在`JVM`手中。**另外，这个方法从 Java 9 开始就已经是`deprecated`了。**

## 5.结论

在本文中，我们简要讨论了三个类似 Java 的关键字:`final, finally`和`finalize`之间的区别。

这篇文章的完整代码可以在 GitHub 上找到。