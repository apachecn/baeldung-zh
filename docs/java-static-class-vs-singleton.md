# Java 中的静态类与单例模式

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-static-class-vs-singleton>

## 1.介绍

在这个快速教程中，我们将讨论使用 [Singleton](/web/20220526055001/https://www.baeldung.com/java-singleton) 设计模式编程和在 Java 中使用静态类之间的一些显著差异。我们将回顾这两种编码方法，并从编程的不同方面对它们进行比较。

到本文结束时，我们将能够在两个选项之间做出正确的选择。

## 2.基础知识

让我们从零开始吧。Singleton 是一种[设计模式](/web/20220526055001/https://www.baeldung.com/design-patterns-series)，它确保在应用程序的生命周期中只有一个`Class` 实例。
它还提供了对该实例的全局访问点。

[`static`](/web/20220526055001/https://www.baeldung.com/java-static)–保留关键字–是一个修饰符，使实例变量成为类变量。因此，这些变量与类(与任何对象)相关联。当与方法一起使用时，它使它们可以只通过类名来访问。最后，我们还可以创建静态的[嵌套内部类](/web/20220526055001/https://www.baeldung.com/java-nested-classes)。

在这个上下文中，静态类包含静态方法和静态变量。

## 3.单例与静态实用程序类

现在，让我们深入兔子洞，了解两大巨头之间的一些突出差异。我们从一些面向对象的概念开始我们的探索。

### 3.1.运行时多态性

Java 中的静态方法在编译时被解析，不能在运行时被覆盖。因此，静态类不能真正受益于运行时多态性:

```java
public class SuperUtility {

    public static String echoIt(String data) {
        return "SUPER";
    }
}

public class SubUtility extends SuperUtility {

    public static String echoIt(String data) {
        return data;
    }
}

@Test
public void whenStaticUtilClassInheritance_thenOverridingFails() {
    SuperUtility superUtility = new SubUtility();
    Assert.assertNotEquals("ECHO", superUtility.echoIt("ECHO"));
    Assert.assertEquals("SUPER", superUtility.echoIt("ECHO"));
}
```

相比之下，**singleton 可以像任何其他类一样通过从基类**派生来利用运行时多态性:

```java
public class MyLock {

    protected String takeLock(int locks) {
        return "Taken Specific Lock";
    }
}

public class SingletonLock extends MyLock {

    // private constructor and getInstance method 

    @Override
    public String takeLock(int locks) {
        return "Taken Singleton Lock";
    }
}

@Test
public void whenSingletonDerivesBaseClass_thenRuntimePolymorphism() {
    MyLock myLock = new MyLock();
    Assert.assertEquals("Taken Specific Lock", myLock.takeLock(10));
    myLock = SingletonLock.getInstance();
    Assert.assertEquals("Taken Singleton Lock", myLock.takeLock(10));
}
```

此外，**singleton 还可以实现接口**，这让它们比静态类更有优势:

```java
public class FileSystemSingleton implements SingletonInterface {

    // private constructor and getInstance method

    @Override
    public String describeMe() {
        return "File System Responsibilities";
    }
}

public class CachingSingleton implements SingletonInterface {

    // private constructor and getInstance method

    @Override
    public String describeMe() {
        return "Caching Responsibilities";
    }
}

@Test
public void whenSingletonImplementsInterface_thenRuntimePolymorphism() {
    SingletonInterface singleton = FileSystemSingleton.getInstance();
    Assert.assertEquals("File System Responsibilities", singleton.describeMe());
    singleton = CachingSingleton.getInstance();
    Assert.assertEquals("Caching Responsibilities", singleton.describeMe());
}
```

实现接口的单例范围的 Spring bean 是这种范例的完美例子。

### 3.2.方法参数

由于它本质上是一个对象，**我们可以很容易地将 singleton 传递给其他方法**作为参数:

```java
@Test
public void whenSingleton_thenPassAsArguments() {
    SingletonInterface singleton = FileSystemSingleton.getInstance();
    Assert.assertEquals("Taken Singleton Lock", singleton.passOnLocks(SingletonLock.getInstance()));
}
```

然而，创建一个静态实用程序类对象并在方法中传递它是没有价值的，也是一个坏主意。

### 3.3.对象状态、序列化和可克隆性

单例可以有实例变量，就像任何其他对象一样，它可以维护这些变量的状态:

```java
@Test
public void whenSingleton_thenAllowState() {
    SingletonInterface singleton = FileSystemSingleton.getInstance();
    IntStream.range(0, 5)
        .forEach(i -> singleton.increment());
    Assert.assertEquals(5, ((FileSystemSingleton) singleton).getFilesWritten());
} 
```

此外，**单例可以被[序列化](/web/20220526055001/https://www.baeldung.com/java-serialization)以保持其状态或者通过媒介**传输，例如网络:

```java
new ObjectOutputStream(baos).writeObject(singleton);
SerializableSingleton singletonNew = (SerializableSingleton) new ObjectInputStream
   (new ByteArrayInputStream(baos.toByteArray())).readObject();
```

最后，实例的存在也为使用`Object's` 克隆方法[克隆](/web/20220526055001/https://www.baeldung.com/java-deep-copy)它提供了可能性:

```java
@Test
public void whenSingleton_thenAllowCloneable() {
    Assert.assertEquals(2, ((SerializableCloneableSingleton) singleton.cloneObject()).getState());
}
```

相反，静态类只有类变量和静态方法，因此，它们没有特定于对象的状态。由于静态成员属于类，我们不能序列化它们。此外，由于缺少要克隆的对象，克隆对于静态类是没有意义的。

### 3.4.加载机制和内存分配

像类的任何其他实例一样，单例驻留在堆上。有利的是，只要应用程序需要，一个巨大的单例对象可以被延迟加载。

另一方面，静态类在编译时包含静态方法和静态绑定变量，并在堆栈上分配。
因此，静态类总是在 JVM 中的[类装载](/web/20220526055001/https://www.baeldung.com/java-classloaders)时被急切地装载。

### 3.5.效率和性能

如前所述，静态类不需要对象初始化。这消除了创建对象所需的时间开销。

此外，通过在编译时进行静态绑定，它们比单例模式更有效，速度也更快。

我们必须仅仅出于设计原因选择单例，而不是为了效率或性能增益而选择单例解决方案。

### 3.6.其他微小差异

对单例类而不是静态类编程也有利于所需的重构量。

毫无疑问，单例是一个类的对象。因此，我们可以很容易地从它转移到一个类的多实例世界。

由于静态方法是在没有对象但有类名的情况下调用的，所以迁移到多实例环境可能是一个相对较大的重构。

其次，在静态方法中，由于逻辑耦合到类定义而不是对象，来自被单元测试的对象的静态方法调用变得更难被伪或存根实现模仿甚至覆盖。

## 4.做出正确的选择

如果我们符合以下条件，请选择单身:

*   应用程序需要完整的面向对象的解决方案
*   在所有给定的时间里只需要一个类的实例来维护一个状态
*   想要一个类的延迟加载解决方案，以便它只在需要的时候加载

在以下情况下使用静态类:

*   只需要存储许多只对输入参数进行操作而不修改任何内部状态的静态实用程序方法
*   不需要运行时多态性或面向对象的解决方案

## 5.结论

在本文中，我们回顾了 Java 中静态类和单例模式之间的一些本质区别。我们还推断了在开发软件时何时使用这两种方法中的任何一种。

和往常一样，我们可以在 GitHub 上找到完整的代码[。](https://web.archive.org/web/20220526055001/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-oop-modifiers)