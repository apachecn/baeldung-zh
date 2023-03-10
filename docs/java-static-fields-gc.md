# 静态字段和垃圾收集

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-static-fields-gc>

## 1.介绍

**在本教程中，我们将学习[垃圾收集](/web/20221103190720/https://www.baeldung.com/jvm-garbage-collectors)如何处理`static`字段。**此外，我们将触及诸如[类加载](/web/20221103190720/https://www.baeldung.com/java-classloaders)和类对象这样的主题。在本文之后，我们将更好地理解类、类加载器和静态字段之间的联系，以及垃圾收集器如何处理它们。

## 2.Java 中的垃圾收集概述

Java 提供了一个非常好的自动内存管理特性。在大多数情况下，这种方法不如手动方法有效。然而，它有助于避免难以调试的问题，并减少样板代码。此外，随着垃圾收集的改进，这个过程变得越来越好。因此，我们应该回顾一下垃圾收集器是如何工作的，以及我们的应用程序中有哪些垃圾。

### 2.1.垃圾对象

**[引用计数](/web/20221103190720/https://www.baeldung.com/java-gc-cyclic-references#reference-counting)是识别垃圾对象最直接直观的方法。**这种方法允许我们检查当前对象是否有对它的引用。然而，这种方法有一些缺点，其中最重要的是循环引用。

处理循环引用的方法之一是追踪。当对象与应用程序的[垃圾收集根](/web/20221103190720/https://www.baeldung.com/java-gc-roots#types-of-gc-roots)没有任何链接时，它们就会变成垃圾。

### 2.2.静态字段和类对象

**在 Java 中，一切都是一个`Object`，包括类的定义。它们包含了关于类、方法和静态字段值的所有元信息。因此，所有的`static`字段都被相关的`class`对象引用。因此，**在类对象存在并且被应用程序引用之前，`static`字段将没有资格进行垃圾收集**。**

同时，所有加载的类都引用了用于加载这个特定类的类加载器。这样，我们可以跟踪加载的类。

在这种情况下，我们有一个层次结构的参考。一个类加载器保存了对所有被加载类的引用。同时，类存储对各自类加载器的引用。在这种情况下，我们有双向引用。每当我们实例化一个新对象时，它都会保存一个对其类定义的引用。因此，我们有以下层次结构:

[![](img/b297fed4a19bf478e6dd81566ff1b268.png)](/web/20221103190720/https://www.baeldung.com/wp-content/uploads/2022/09/classloader_diagram.png)

在我们的应用程序引用一个类之前，我们不能卸载它。让我们来看看我们需要什么来使一个类定义符合垃圾收集的条件。首先，应用程序不应该引用一个类的实例。这很重要，因为所有的实例都引用了它们的类。第二，这个类的类加载器应该不能从应用程序中获得。最后，类本身在应用程序中不应该有引用。

## 3.垃圾收集静态字段的示例

让我们创建一个例子，让垃圾收集器删除我们的静态字段。对于扩展和系统类加载器加载的类，JVM 支持类卸载。然而，这将很难重现，我们将为此使用一个定制的类加载器，因为我们将对它有更多的控制。

### 3.1.自定义类加载器

首先，让我们创建自己的`CustomClassloader`将从应用程序的资源文件夹中加载一个类。为了让我们的类加载器工作，我们应该覆盖`loadClass(String name)` 方法:

```java
public class CustomClassloader extends ClassLoader {

    public static final String PREFIX = "com.baeldung.classloader";

    public CustomClassloader(ClassLoader parent) {
        super(parent);
    }

    @Override
    public Class<?> loadClass(String name) throws ClassNotFoundException {
        if (name.startsWith(PREFIX)) {
            return getClass(name);
        } else {
            return super.loadClass(name);
        }
    }

    ...
}
```

在这个实现中，我们使用了`getClass`方法，它隐藏了从资源中加载类的复杂性:

```java
private Class<?> getClass(String name) {
    String fileName = name.replace('.', File.separatorChar) + ".class";
    try {
        byte[] byteArr = IOUtils.toByteArray(getClass().getClassLoader().getResourceAsStream(fileName));
        Class<?> c = defineClass(name, byteArr, 0, byteArr.length);
        resolveClass(c);
        return c;
    } catch (IOException e) {
        throw new RuntimeException(e);
    }
} 
```

### 3.2.文件夹结构

为了正确工作，我们的自定义类应该在我们的类路径的范围之外。这样就不会被[系统类加载器](/web/20221103190720/https://www.baeldung.com/java-system-gc#garbage%20collection)上传了。唯一处理这个特定类的类加载器将是我们的`CustomClassloader`。

我们的文件夹结构将如下所示:

[![](img/9e465dd0aa209d567ccedf47885a6274.png)](/web/20221103190720/https://www.baeldung.com/wp-content/uploads/2022/09/Screen-Shot-2022-09-05-at-10.46.21.png)

### 3.3.静态场保持器

我们将使用一个自定义类来扮演我们的`static`字段的容器角色。在定义了类加载器的实现之后，我们可以用它来上传我们已经准备好的类。这是一个简单的类:

```java
public class GarbageCollectedStaticFieldHolder {

    private static GarbageCollectedInnerObject garbageCollectedInnerObject =
      new GarbageCollectedInnerObject("Hello from a garbage collected static field");

    public void printValue() {
        System.out.println(garbageCollectedInnerObject.getMessage());
    }
}
```

### 3.4.静态字段类

`GarbageCollectedInnerObject `将代表一个我们想变成垃圾的物体。为了简单和方便起见，这个类与`GarbageCollectedStaticFieldHolder. `定义在同一个文件中。这个类包含一条消息，并且覆盖了`finalize()`方法。**虽然`[finalize()](/web/20221103190720/https://www.baeldung.com/java-finalize) `方法被弃用并且有很多缺点，但它将允许我们可视化垃圾收集器何时移除对象。我们将仅出于演示目的使用这种方法。下面是我们班的一个`static`字段:**

```java
class GarbageCollectedInnerObject {

    private final String message;

    public GarbageCollectedInnerObject(final String message) {
        this.message = message;
    }

    public String getMessage() {
        return message;
    }

    @Override
    protected void finalize() {
        System.out.println("The object is garbage now");
    }
}
```

### 3.5.上传课程

现在我们可以上传和实例化我们的类。创建实例后，我们可以确保类已上传，对象已创建，并且静态字段包含所需的信息:

```java
private static void loadClass() {
    try {
        final String className = "com.baeldung.classloader.GarbageCollectedStaticFieldHolder";
        CustomClassloader loader = new CustomClassloader(Main.class.getClassLoader());
        Class<?> clazz = loader.loadClass(className);
        Object instance = clazz.getConstructor().newInstance();
        clazz.getMethod(METHOD_NAME).invoke(instance);
    } catch (Exception e) {
        throw new RuntimeException(e);
    }
}
```

这个方法应该创建我们的特殊类的实例，并输出消息:

```java
Hello from a garbage collected static field
```

### 3.6.垃圾收集正在进行

现在，让我们启动我们的应用程序，并尝试删除垃圾:

```java
public static void main(String[] args) throws InterruptedException {
    loadClass();
    System.gc();
    Thread.sleep(1000);
} 
```

调用方法`loadClass(),`后，该方法中的所有变量，即 classloader、我们的类和实例，都将超出范围，并失去与垃圾收集根的连接。也可以将`null`赋给引用，但是使用 scope 的选项更简洁:

```java
public static void main(String[] args) throws InterruptedException {
    CustomClassloader loader;
    Class<?> clazz;
    Object instance;
    try {
        final String className = "com.baeldung.classloader.GarbageCollectedStaticFieldHolder";
        loader = new CustomClassloader(GarbageCollectionNullExample.class.getClassLoader());
        clazz = loader.loadClass(className);
        instance = clazz.getConstructor().newInstance();
        clazz.getMethod(METHOD_NAME).invoke(instance);
    } catch (Exception e) {
        throw new RuntimeException(e);
    }
    loader = null;
    clazz = null;
    instance = null;
    System.gc();
    Thread.sleep(1000);
}
```

尽管我们对这段代码有一些问题，但它在大多数情况下都能工作。**主要问题是我们不能在 Java 中强制垃圾收集，并且 [`System.gc()`](/web/20221103190720/https://www.baeldung.com/java-system-gc#systemgc) 的调用不能保证车库收集会发生。**然而，在大多数 JVM 实现中，这将触发[大规模垃圾收集](/web/20221103190720/https://www.baeldung.com/java-choosing-gc-algorithm)。因此，我们应该在输出中看到以下几行:

```java
Hello from a garbage collected static field 
The object is garbage now
```

这个输出向我们展示了垃圾收集器删除了静态字段。垃圾收集器还移除了类加载器、容器的类、静态字段和连接的对象。

### 3.7.没有`System.gc()`的示例

我们还可以更自然地触发垃圾收集。这种方式会更稳定。但是，调用垃圾收集器需要更多的周期:

```java
public static void main(String[] args) {
    while (true) {
        loadClass();
    }
}
```

这里我们使用相同的`loadClass() `方法，但是我们不调用`System.gc()`，当我们耗尽内存时垃圾收集器被触发，因为我们在无限循环中加载类。

## 4.结论

这篇文章告诉了我们 Java 中关于类和静态字段的垃圾收集是如何工作的。我们创建了一个定制的类装入器，并在我们的例子中使用了它。此外，我们学习了类加载器、类和它们的`static`字段之间的联系。为了进一步理解，有必要浏览一下文中链接的文章。

代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221103190720/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-jvm-2)