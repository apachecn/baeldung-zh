# ClassNotFoundException vs NoClassDefFoundError

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-classnotfoundexception-and-noclassdeffounderror>

## 1.介绍

当 JVM 在类路径上找不到请求的类时,`ClassNotFoundException`和`NoClassDefFoundError`都会发生。虽然他们看起来很熟悉，但这两者之间有一些核心差异。

在本教程中，我们将讨论它们发生的一些原因及其解决方案。

## 2.`ClassNotFoundException`

`ClassNotFoundException`是一个已检查的异常，当应用程序试图通过类的完全限定名加载类，但在类路径中找不到它的定义时，就会出现该异常。

这主要发生在试图使用`Class.forName()`、`ClassLoader.loadClass()`或`ClassLoader.findSystemClass()`加载类的时候。因此，在使用反射时，我们需要格外小心`java.lang.ClassNotFoundException` 。

例如，让我们尝试加载 JDBC 驱动程序类，而不添加必要的依赖项，这会让我们`ClassNotFoundException:`

```java
@Test(expected = ClassNotFoundException.class)
public void givenNoDrivers_whenLoadDriverClass_thenClassNotFoundException() 
  throws ClassNotFoundException {
      Class.forName("oracle.jdbc.driver.OracleDriver");
}
```

## 3.`NoClassDefFoundError`

`NoClassDefFoundError`是致命错误。当 JVM 在尝试以下操作时找不到类的定义时，就会出现这种情况:

*   使用`new`关键字实例化一个类
*   用方法调用加载类

当编译器可以成功编译该类，但 Java 运行时找不到该类文件时，就会出现该错误。它通常发生在执行静态块或初始化类的静态字段时出现异常，因此类初始化失败。

让我们考虑一个场景，这是重现该问题的一种简单方法。初始化抛出异常。所以，当我们试图创建一个`ClassWithInitErrors,`的对象时，它抛出了`ExceptionInInitializerError.`

如果我们再次尝试加载同一个类，我们会得到`NoClassDefFoundError:`

```java
public class ClassWithInitErrors {
    static int data = 1 / 0;
}
```

```java
public class NoClassDefFoundErrorExample {
    public ClassWithInitErrors getClassWithInitErrors() {
        ClassWithInitErrors test;
        try {
            test = new ClassWithInitErrors();
        } catch (Throwable t) {
            System.out.println(t);
        }
        test = new ClassWithInitErrors();
        return test;
    }
}
```

让我们为这个场景编写一个测试用例:

```java
@Test(expected = NoClassDefFoundError.class)
public void givenInitErrorInClass_whenloadClass_thenNoClassDefFoundError() {

    NoClassDefFoundErrorExample sample
     = new NoClassDefFoundErrorExample();
    sample.getClassWithInitErrors();
}
```

## 4.解决

有时，诊断和修复这两个问题会非常耗时。这两个问题的主要原因是运行时类文件(在类路径中)不可用。

让我们来看看在处理这些问题时可以考虑的几种方法:

1.  我们需要确保类或包含该类的 jar 在类路径中是否可用。如果没有，我们需要添加它
2.  如果它在应用程序的类路径中可用，那么很可能类路径被覆盖了。为了解决这个问题，我们需要找到应用程序使用的确切的类路径
3.  此外，如果一个应用程序使用多个类装入器，由一个类装入器装入的类可能对其他类装入器不可用。为了更好地解决这个问题，了解 Java 中的类加载器是如何工作的是很重要的

## 5.摘要

虽然这两种异常都与类路径和 Java 运行时在运行时找不到类有关，但是注意它们的区别是很重要的。

Java 运行时在试图只在运行时加载一个类时抛出了`ClassNotFoundException` ，并且在运行时提供了该类的名称。在`NoClassDefFoundError, the` 的情况下，类在编译时存在，但 Java 运行时在运行时无法在 Java 类路径中找到它。

和往常一样，所有例子的完整代码都可以在 GitHub 上找到[。](https://web.archive.org/web/20220626085449/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-exceptions)