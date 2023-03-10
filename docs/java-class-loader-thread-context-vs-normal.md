# 线程的上下文类装入器和普通类装入器的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-class-loader-thread-context-vs-normal>

## 1.概观

Java 使用不同类型的[类加载器](/web/20220706094102/https://www.baeldung.com/java-classloaders)在程序执行过程中加载资源。在本教程中，我们将探究 Java 中当前类装入器和线程类装入器的行为差异。

## 2.类装入器是做什么的？

Java 类加载器定位并加载应用程序执行所需的类。如果请求的类依赖于任何其他资源，它们也会被加载。

我们需要 **合适的类加载器，以便在 Java 程序**需要的时候 **加载不同类型的类。**

## 3.类装入器之间的关系

Java 类加载器遵循一个层次关系。

每个寻找或装入类的请求都被委托给各自的父类装入器。如果所有的祖先类装入器都找不到一个类，那么当前的类装入器就试图找到它。这里，“当前类”意味着当前正在执行的方法的类。

类加载器之间的这种关系有助于维护应用程序中资源的唯一性。另外，如果一个类已经被父类装入器装入，子类装入器不需要重新装入它。

## 4.默认类装入器

类装入器装入存在于它们各自的类路径中的类和资源:

*   系统或应用程序类装入器从应用程序类路径装入类
*   扩展类装入器搜索扩展类路径(`JRE/lib/ext`)
*   引导类装入器查看引导类路径(`JRE/lib/rt.jar`)

引导或原始类装入器是所有类装入器的父类。它加载 Java 运行时——运行 JVM 本身所需的类。

当前的类装入器以线性、分层的方式搜索资源。如果一个类装入器找不到一个类，它会将`java.lang.ClassNotFoundException`抛出给相应的子类装入器。然后，子类加载器尝试搜索该类。

对于在层次结构中任何类装入器的类路径上都找不到所需资源的场景，我们得到与`java.lang.ClassNotFoundException`相关的错误消息作为最终结果。

我们也可以定制默认的类加载行为。**我们可以在动态加载类的同时显式指定类加载器**。

然而，我们应该注意，如果我们从不同类型的类装入器中装入同一个类，JVM 会把它们看作不同的资源。

## 5.上下文类加载器

除了默认的类加载器，J2SE 还引入了[上下文类加载器](/web/20220706094102/https://www.baeldung.com/java-classloaders#context-classloaders)。

Java 中的每个 t **hread** **都有一个关联的上下文类加载器**。

我们可以使用`Thread` 类的 *getContextClassLoader()* 和 *setContextClassLoader()* 方法来访问/修改线程的上下文类加载器。

上下文类加载器在创建线程时设置。**如果没有显式设置，则** **默认为父线程**的上下文类加载器。

上下文类装入器也遵循层次模型。在这种情况下，根类加载器是原始线程的上下文类加载器。原始线程是操作系统创建的初始线程。

随着应用程序开始执行，可能会创建其他线程。原始线程的上下文类加载器最初被设置为加载应用程序的类加载器，即系统类加载器。

假设我们没有为层次结构中任何级别的任何线程更新上下文类加载器。因此，我们可以说，默认情况下，线程的上下文类加载器与系统类加载器相同。对于这样的场景，如果我们执行 *Thread.currentThread()。getContextClassLoader()* 和 *getClass()。getClassLoader()* 操作，两者将返回相同的对象。

### 5.1.通过授权处理问题

当缺省 Java 类装入器的类路径中没有所需的资源时，上下文类装入器非常重要。因此，**我们可以使用上下文类加载器来偏离传统的线性委托模型**。

在类装入器的层次模型中，父类装入器装入的资源对子类装入器是可见的，但反之则不然。在某些情况下，父类装入器可能需要访问子类装入器的类路径中的类。

上下文类装入器是实现这一点的有用工具。当访问所需的资源时，我们可以将上下文类加载器设置为所需的值。因此，在上面的例子中，我们可以使用子线程的上下文类加载器，并且可以定位存在于子类加载器级别的资源。

### 5.2.多模块环境

在设置上下文类加载器属性时，**我们基本上是在切换加载资源的上下文**。我们不是在当前的类路径中搜索，而是获取一个指向不同类路径的新类加载器。如果我们希望从第三方模块加载资源，或者如果我们在具有不同类命名空间的环境中工作，这将特别有帮助。

然而，**我们应该小心，将上下文类加载器属性重置回原始类加载器，以避免将来出现任何差异。**

## 6.结论

在本文中，我们分析了使用上下文类装入器来装入无法通过普通类装入器访问的资源的重要性。我们看到，我们还可以选择临时更新给定线程的上下文类加载器，以加载所需的类。

理解当前方法的工作环境是至关重要的。我们可以让同名的资源存在于不同的类路径中。因此，当从多个类装入器装入资源时，我们应该小心。