# Class.getResource()和 ClassLoader.getResource()之间的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-class-vs-classloader-getresource>

## 1.概观

在这个简短的教程中，我们将看看`Class.getResource()`和`ClassLoader.getResource()`方法之间的区别。

## 2.`getResource()`法

我们可以在`Class`或`ClassLoader`实例上使用`getResource()`方法来查找具有给定名称的资源。资源被认为是数据，例如图像、文本、音频等等。作为路径分隔符，我们应该始终使用斜杠(“/”)。

该方法返回一个用于读取资源的 [URL](/web/20221112192243/https://www.baeldung.com/java-url) 对象，或者如果找不到资源或者调用者没有检索资源的权限，则返回`null`值。

## 3.`Class.getResource()`

现在，让我们看看如何使用 [`Class`](https://web.archive.org/web/20221112192243/https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html) 实例获取资源。**当用`Class`对象定位资源时，我们可以传递一个绝对或相对路径。**

用于搜索与给定类相关联的资源的规则由该类的类加载器实现。

寻找资源的过程将委托给类对象的类装入器。换句话说，定义在`Class`实例上的`getResouce()`方法最终将调用`ClassLoader`的`getResouce()`方法。

在委托之前，将从给定的资源名称中派生出一个绝对资源名称。创建绝对资源名时，将使用以下算法:

*   如果资源名称以斜杠("/")开头，则表明资源名称是绝对的。绝对资源名称的前导斜杠被清除，并且不做任何修改地传递给适当的`ClassLoader`方法来定位资源。
*   如果提供的资源名称不是以斜杠开头，则该名称被视为相对于该类的包。相对名称首先被转换成绝对名称，然后被传递给`ClassLoader`方法。

首先，假设我们在`com/bealdung/resource`目录中定义了`example.txt`资源。此外，让我们假设我们在`com.baeldung.resourc` e 包中定义了类`ClassGetResourceExample`。

现在，我们可以使用绝对路径检索资源:

```java
void givenAbsoluteResourcePath_whenGetResource_thenReturnResource() {
    URL resourceAbsolutePath = ClassGetResourceExample.class
        .getResource("/com/baeldung/resource/example.txt");
    Assertions.assertNotNull(resourceAbsolutePath);
}
```

**使用`Class.getResource()`时，绝对资源路径应该以前导斜杠开始。**

此外，由于我们的资源和我们的类在同一个包中，我们也可以使用相对路径来检索它:

```java
void givenRelativeResourcePath_whenGetResource_thenReturnResource() {
    URL resourceRelativePath = ClassGetResourceExample.class.getResource("example.txt");
    Assertions.assertNotNull(resourceRelativePath);
}
```

然而，重要的是要提到，只有当资源在同一个包中被定义为一个类时，我们才能使用相对路径来获取资源。否则，我们将得到一个`null`作为值。

## 4.`ClassLoader.getResource()`

顾名思义， [`ClassLoader`](/web/20221112192243/https://www.baeldung.com/java-classloaders) 代表一个负责加载类的类。每个`Class`实例都包含对其`ClassLoader`的引用。

`ClassLoader`类使用委托模型来搜索类和资源。此外，`ClassLoader`类的每个实例都有一个关联的父类`ClassLoader`。

当被要求查找资源时，`ClassLoader`实例将首先将搜索委托给它的父实例`ClassLoader`,然后再尝试查找资源本身。

如果父`ClassLoader`不存在，则搜索虚拟机内置`ClassLoader`的路径，称为引导类加载器。引导类加载器没有父类，但是可以作为一个`ClassLoader`实例的父类。

或者，如果先前的搜索失败，该方法将调用`findResource()`方法来查找资源。

被指定为输入的资源名总是被认为是绝对的。需要注意的是，Java 从类路径加载资源。

让我们使用绝对路径和`ClassLoader`实例来获取资源:

```java
void givenAbsoluteResourcePath_whenGetResource_thenReturnResource() {
    URL resourceAbsolutePath = ClassLoaderGetResourceExample.class.getClassLoader()
        .getResource("com/baeldung/resource/example.txt");
    Assertions.assertNotNull(resourceAbsolutePath);
}
```

**我们调用`ClassLoader.getResource()`的时候，在定义绝对路径的时候要省略前导斜杠。**

使用`ClassLoader`实例，**我们无法使用相对路径**获得资源:

```java
void givenRelativeResourcePath_whenGetResource_thenReturnNull() {
    URL resourceRelativePath = ClassLoaderGetResourceExample.class.getClassLoader()
        .getResource("example.txt");
    Assertions.assertNull(resourceRelativePath);
}
```

上面的测试演示了该方法返回一个`null`值作为结果。

## 5.结论

这篇简短的教程解释了从`Class`和`ClassLoader`实例中调用`getResource()`方法的区别。综上所述，在使用`Class`实例调用方法时，我们可以传递相对或绝对资源路径，但是在`ClassLoader`上调用方法时，我们只能使用绝对路径。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20221112192243/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-jvm-2)