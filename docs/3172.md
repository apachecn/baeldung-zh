# 为什么缺少注释不会导致 ClassNotFoundException

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/classnotfoundexception-missing-annotation>

## 1.概观

在本教程中，我们将熟悉 Java 编程语言中一个看似奇怪的特性:缺少注释不会在运行时导致任何异常。

然后，我们将深入挖掘，看看是什么原因和规则支配着这种行为，以及这些规则的例外是什么。

## 2.快速复习

让我们从一个熟悉的 Java 例子开始。有阶级`A`，然后有阶级`B`，这取决于`A`:

```
public class A {
}

public class B {
    public static void main(String[] args) {
        System.out.println(new A());
    }
}
```

现在，如果我们编译这些类并运行编译后的`B`，它会在控制台上为我们打印一条消息:

```
>> javac A.java
>> javac B.java
>> java B
[[email protected]](/web/20220523152548/https://www.baeldung.com/cdn-cgi/l/email-protection)
```

但是，如果我们删除编译好的`A` `.class`文件，重新运行类`B`，我们会看到一个 [`NoClassDefFoundError` 导致一个`ClassNotFoundException`](/web/20220523152548/https://www.baeldung.com/java-classnotfoundexception-and-noclassdeffounderror) :

```
>> rm A.class
>> java B
Exception in thread "main" java.lang.NoClassDefFoundError: A
        at B.main(B.java:3)
Caused by: java.lang.ClassNotFoundException: A
        at java.base/jdk.internal.loader.BuiltinClassLoader.loadClass(BuiltinClassLoader.java:606)
        at java.base/jdk.internal.loader.ClassLoaders$AppClassLoader.loadClass(ClassLoaders.java:168)
        at java.base/java.lang.ClassLoader.loadClass(ClassLoader.java:522)
        ... 1 more
```

发生这种情况是因为类加载器在运行时找不到类文件，即使它在编译期间就在那里。这是许多 Java 开发人员期望的正常行为。

## 3.缺少注释

现在，让我们看看在同样的情况下注释会发生什么。为了做到这一点，我们将把`A`类改为一个注释:

```
@Retention(RetentionPolicy.RUNTIME)
public @interface A {
} 
```

如上所示，Java 将在运行时保留注释信息。之后，是时候用`A`注释类`B`了:

```
@A
public class B {
    public static void main(String[] args) {
        System.out.println("It worked!");
    }
} 
```

接下来，让我们编译并运行这些类:

```
>> javac A.java
>> javac B.java
>> java B
It worked!
```

因此，我们看到`B`成功地在控制台上打印了它的消息，这是有意义的，因为所有的东西都被很好地编译和连接在一起。

现在，让我们删除`A`的类文件:

```
>> rm A.class
>> java B
It worked!
```

如上所示，**即使注释类文件丢失，带注释的类也会运行，没有任何异常**。

### 3.1.带有类标记的注释

为了更加有趣，让我们引入另一个具有`Class<?>`属性的注释:

```
@Retention(RetentionPolicy.RUNTIME)
public @interface C {
    Class<?> value();
}
```

如上所示，这个注释有一个名为`value `的属性，返回类型为`Class<?>`。作为该属性的参数，让我们添加另一个名为`D`的空类:

```
public class D {
}
```

现在，我们要用这个新注释来注释`B`类:

```
@A
@C(D.class)
public class B {
    public static void main(String[] args) {
        System.out.println("It worked!");
    }
} 
```

当所有的类文件都存在时，一切都应该正常工作。但是，如果我们只删除了`D`类文件，不碰其他的，会怎么样呢？让我们来看看:

```
>> rm D.class
>> java B
It worked!
```

如上图，尽管运行时没有了`D`，但一切都还在工作！因此，除了注释，**属性引用的类标记也不需要在运行时出现**。

### 3.2.Java 语言规范

因此，我们看到一些带有运行时保留的注释在运行时丢失了，但是带注释的类运行得很好。听起来可能出乎意料，但根据 Java 语言规范 9.6.4.2 的说法，这种行为实际上完全没问题:

> 注释可能只出现在源代码中，也可能以类或接口的二进制形式出现。通过 Java SE 平台的反射库，以二进制形式出现的注释在运行时可能可用，也可能不可用。

此外， [JLS 13.5.7](https://web.archive.org/web/20220523152548/https://docs.oracle.com/javase/specs/jls/se16/html/jls-13.html#jls-13.5.7) 条目还声明:

> 添加或删除注释对 Java 编程语言中程序的二进制表示的正确链接没有影响。

**底线是，运行时不会因为缺少注释而抛出异常，因为 JLS 允许这样做**。

### 3.3.访问缺失的注释

让我们改变`B`类，使其反射性地检索`A`信息:

```
@A
public class B {
    public static void main(String[] args) {
        System.out.println(A.class.getSimpleName());
    }
}
```

如果我们编译并运行它们，一切都会好的:

```
>> javac A.java
>> javac B.java
>> java B
A
```

现在，如果我们删除`A`类文件并运行`B`，我们将看到由`ClassNotFoundException`引起的相同的`NoClassDefFoundError`:

```
Exception in thread "main" java.lang.NoClassDefFoundError: A
        at B.main(B.java:5)
Caused by: java.lang.ClassNotFoundException: A
        at java.base/jdk.internal.loader.BuiltinClassLoader.loadClass(BuiltinClassLoader.java:606)
        at java.base/jdk.internal.loader.ClassLoaders$AppClassLoader.loadClass(ClassLoaders.java:168)
        at java.base/java.lang.ClassLoader.loadClass(ClassLoader.java:522)
        ... 1 more
```

根据 JLS 的说法，注释不一定要在运行时可用。然而，**当一些其他代码读取注释并对其做一些事情时(就像我们所做的)，注释必须在运行时出现**。否则，我们会看到一个`ClassNotFoundException`。

## 4.结论

在本文中，我们看到了一些注释是如何在运行时消失的，即使它们是类的二进制表示的一部分。

像往常一样，所有的例子都可以在 GitHub 上找到[。](https://web.archive.org/web/20220523152548/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-annotations)