# 如何预热 JVM

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-jvm-warmup>

## 1。概述

JVM 是有史以来最古老而强大的虚拟机之一。

在本文中，我们快速了解一下预热 JVM 意味着什么以及如何预热。

## 2。JVM 架构基础知识

每当一个新的 JVM 进程启动时，所有需要的类都由[类加载器](https://web.archive.org/web/20220901150254/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/ClassLoader.html)的实例加载到内存中。这一过程分三步进行:

1.  **引导类加载:**“`Bootstrap Class Loader`”将 Java 代码和必要的 Java 类如`java.lang.Object` 加载到内存中。这些加载的类驻留在`JRE\lib\rt.jar`中。
2.  **扩展类加载**:ext Class loader 负责加载位于`java.ext.dirs`路径的所有 JAR 文件。在非 Maven 或非 Gradle 应用程序中，开发人员手动添加 jar，所有这些类都在这个阶段加载。
3.  **应用类加载**:app Class loader 加载位于应用类路径中的所有类。

这个初始化过程基于一个惰性加载方案。

## 3。什么在预热 JVM

一旦类加载完成，所有重要的类(在流程启动时使用)都被推送到 [JVM 缓存(本机代码)](https://web.archive.org/web/20220901150254/https://www.ibm.com/support/knowledgecenter/en/SSAW57_8.5.5/com.ibm.websphere.nd.doc/ae/rdyn_tunediskcache.html)——这使得它们在运行时可以更快地被访问。其他类是基于每个请求加载的。

向 Java web 应用程序发出的第一个请求通常比流程生命周期中的平均响应时间慢得多。这个准备阶段通常可以归因于懒惰的类加载和实时编译。

记住这一点，对于低延迟应用程序，我们需要预先缓存所有的类——以便它们在运行时被访问时立即可用。

这个调整 JVM 的过程被称为预热。

## 4。分层编译

由于 JVM 良好的体系结构，经常使用的方法在应用程序生命周期中被加载到本机缓存中。

当应用程序启动时，我们可以利用这个属性将关键方法强制加载到缓存中。为此，我们需要设置一个名为 **`Tiered Compilation` :** 的 VM 参数

```java
-XX:CompileThreshold -XX:TieredCompilation
```

通常，VM 使用解释器来收集关于方法的分析信息，这些信息被输入到编译器中。在分层方案中，除了解释器之外，客户端编译器还用于生成方法的编译版本，这些方法收集关于它们自身的分析信息。

因为编译的代码比解释的代码快得多，所以程序在分析阶段的执行性能更好。

在 JBoss 和 JDK 版本 7 上运行的应用程序，如果启用了这个 VM 参数，一段时间后会因为一个记录在案的 [bug](https://web.archive.org/web/20220901150254/https://issues.jboss.org/browse/JBEAP-26) 而崩溃。该问题已在 JDK 版本 8 中修复。

这里要注意的另一点是，为了强制加载，我们必须确保所有(或大多数)将要执行的类都需要被访问。这类似于在单元测试期间确定代码覆盖率。覆盖的代码越多，性能就越好。

下一节将演示如何实现这一点。

## 5。手动执行

我们可以实现另一种技术来预热 JVM。在这种情况下，一个简单的手动预热可能包括在应用程序一启动就重复创建不同的类数千次。

首先，我们需要用常规方法创建一个虚拟类:

```java
public class Dummy {
    public void m() {
    }
}
```

接下来，我们需要创建一个类，该类具有一个静态方法，一旦应用程序启动，该方法将至少执行 100000 次，并且每次执行时，它都会创建前面提到的我们之前创建的虚拟类的一个新实例:

```java
public class ManualClassLoader {
    protected static void load() {
        for (int i = 0; i < 100000; i++) {
            Dummy dummy = new Dummy();
            dummy.m();
        }
    }
}
```

现在，为了**测量性能增益**，我们需要创建一个主类。该类包含一个静态块，该静态块包含对`ManualClassLoader's load()`方法的直接调用。

在 main 函数中，我们再次调用`ManualClassLoader's load()`方法，并在函数调用前后捕获纳秒级的系统时间。最后，我们减去这些时间，得到实际的执行时间。

我们必须运行应用程序两次；一次使用静态块中的`load()`方法调用，一次不使用该方法调用:

```java
public class MainApplication {
    static {
        long start = System.nanoTime();
        ManualClassLoader.load();
        long end = System.nanoTime();
        System.out.println("Warm Up time : " + (end - start));
    }
    public static void main(String[] args) {
        long start = System.nanoTime();
        ManualClassLoader.load();
        long end = System.nanoTime();
        System.out.println("Total time taken : " + (end - start));
    }
}
```

以下是以纳秒为单位的结果:

| **带预热** | **没有预热** | **差值(%)** |
| 1220056 | 8903640 | Seven hundred and thirty |
| 1083797 | 13609530 | One thousand two hundred and fifty-six |
| 1026025 | 9283837 | Nine hundred and five |
| 1024047 | 7234871 | Seven hundred and six |
| 868782 | 9146180 | One thousand and fifty-three |

正如所料，预热方法比正常方法表现出更好的性能。

当然，这是一个**非常简单的基准**,并且仅仅提供了对这种技术影响的一些表面层次的洞察。此外，重要的是要理解，对于现实世界的应用程序，我们需要熟悉系统中的典型代码路径。

## 6。工具

我们还可以使用几个工具来预热 JVM。最著名的工具之一是 Java 微基准测试工具， [JMH](https://web.archive.org/web/20220901150254/https://openjdk.java.net/projects/code-tools/jmh/) 。它通常用于微观基准测试。一旦加载完毕，**就会反复点击一段代码，并监控预热迭代周期。**

要使用它，我们需要向`pom.xml`添加另一个依赖项:

```java
<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-core</artifactId>
    <version>1.35</version>
</dependency>
<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-generator-annprocess</artifactId>
    <version>1.35</version>
</dependency>
```

我们可以在[中央 Maven 资源库](https://web.archive.org/web/20220901150254/https://search.maven.org/classic/#search%7Cga%7C1%7Corg.openjdk.jmh)中查看 JMH 的最新版本。

或者，我们可以使用 JMH 的 maven 插件来生成一个示例项目:

```java
mvn archetype:generate \
    -DinteractiveMode=false \
    -DarchetypeGroupId=org.openjdk.jmh \
    -DarchetypeArtifactId=jmh-java-benchmark-archetype \
    -DgroupId=com.baeldung \
    -DartifactId=test \
    -Dversion=1.0
```

接下来，让我们创建一个`main`方法:

```java
public static void main(String[] args) 
  throws RunnerException, IOException {
    Main.main(args);
}
```

现在，我们需要创建一个方法，并用 JMH 的`@Benchmark`注释对其进行注释:

```java
@Benchmark
public void init() {
    //code snippet	
}
```

在这个`init`方法内部，我们需要编写需要重复执行的代码，以便预热。

## 7。性能指标评测

在过去的 20 年里，对 Java 的大部分贡献都与 GC(垃圾收集器)和 JIT(实时编译器)有关。几乎所有的在线性能基准都是在已经运行了一段时间的 JVM 上完成的。然而，

然而， [`Beihang University`](https://web.archive.org/web/20220901150254/http://www.eecg.toronto.edu/~yuan/papers/osdi16-hottub.pdf) 已经公布了一份考虑 JVM 预热时间的基准测试报告。他们使用基于 Hadoop 和 Spark 的系统来处理海量数据:

[![jvm](img/9a2c32f1fca51daa397df87a46cba03b.png)](/web/20220901150254/https://www.baeldung.com/wp-content/uploads/2017/06/jvm.png)

这里 HotTub 指定了 JVM 预热的环境。

如您所见，速度提升非常显著，尤其是对于相对较小的读取操作，这也是考虑这些数据的原因。

## 8。结论

在这篇简短的文章中，我们展示了 JVM 如何在应用程序启动时加载类，以及我们如何预热 JVM 以获得性能提升。

如果你想继续的话，这本书提供了更多关于这个主题的信息和指南。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220901150254/https://github.com/eugenp/tutorials/tree/master/libraries)