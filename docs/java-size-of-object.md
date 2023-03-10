# 如何在 Java 中获得对象的大小

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-size-of-object>

## 1。概述

与 C/C++不同，在 C/c++中，我们可以使用 sizeof `()`方法来获取以字节为单位的对象大小，而在 Java 中没有真正的等效方法。

在本文中，我们将演示如何仍然可以获得特定对象的大小。

## 2.Java 中的内存消耗

虽然 Java 中没有`sizeof`运算符，但我们实际上并不需要。所有基本类型都有一个标准大小，通常没有填充或对齐字节。然而，这并不总是简单明了的。

**尽管原语必须表现得好像它们有正式的大小，JVM 可以在内部以它喜欢的任何方式存储数据，有任何数量的填充或开销**。它可以选择将一个`boolean[]`存储在类似`BitSet`的 64 位长的块中，在堆栈上分配一些临时的`Object`，或者优化一些完全不存在的变量或方法调用，用常量替换它们，等等……但是，只要程序给出相同的结果，就完全没问题。

考虑到硬件和操作系统缓存的影响(我们的数据可能会在每个缓存级别上复制)，这意味着**我们只能粗略地预测 RAM 消耗**。

### 2.1.对象、引用和包装类

**现代 64 位 JDK 的最小对象大小是 16 字节**，因为对象有 12 字节的头，填充为 8 字节的倍数。在 32 位 JDK 中，开销是 8 个字节，填充为 4 个字节的倍数。

**在堆边界小于 32Gb ( `-Xmx32G`)的 32 位平台和 64 位平台**上，引用的典型大小为 4 字节，对于 32Gb 以上的边界，引用的典型大小为 8 字节。

这意味着 64 位 JVM 通常需要多 30-50%的堆空间。

尤其值得注意的是，**装箱类型、数组、`String`和其他类似多维数组的容器是内存开销很大的，因为它们增加了一定的开销**。例如，当我们比较`int`原语(仅消耗 4 个字节)和`Integer`对象(占用 16 个字节)时，我们看到有 300%的内存开销。

## 3.使用仪器估计物体大小

在 Java 中估算对象大小的一种方法是使用 Java 5 中引入的 [`Instrumentation`接口](https://web.archive.org/web/20221205184948/https://docs.oracle.com/en/java/javase/11/docs/api/java.instrument/java/lang/instrument/Instrumentation.html)的`getObjectSize(Object)`方法。

正如我们在 Javadoc 文档中看到的，该方法提供了指定对象大小的“特定于实现的近似值”。值得注意的是，在单个 JVM 调用期间，大小中可能包含开销，并且值可能不同。

**这种方法仅支持对所考虑的对象本身的尺寸估计，而不支持对其引用的对象的尺寸估计**。为了估计对象的总大小，我们需要一个代码来检查这些引用并计算估计的大小。

### 3.1.正在创建检测代理

为了调用`Instrumentation.getObjectSize(Object)`来获取对象的大小，我们需要能够首先访问插装的实例。**我们需要使用插装代理**，有两种方法，如 [`java.lang.instrument`包的文档中所述。](https://web.archive.org/web/20221205184948/https://docs.oracle.com/en/java/javase/11/docs/api/java.instrument/java/lang/instrument/package-summary.html)

可以通过命令行指定插装代理，或者我们可以使用已经运行的 JVM 。我们将集中讨论第一个问题。

为了**通过命令行**指定插装代理，我们需要实现重载的`premain`方法，该方法将在使用插装时首先被 JVM 调用。除此之外，我们需要公开一个静态方法来访问`Instrumentation.getObjectSize(Object)`。

现在让我们创建`InstrumentationAgent`类:

```java
public class InstrumentationAgent {
    private static volatile Instrumentation globalInstrumentation;

    public static void premain(final String agentArgs, final Instrumentation inst) {
        globalInstrumentation = inst;
    }

    public static long getObjectSize(final Object object) {
        if (globalInstrumentation == null) {
            throw new IllegalStateException("Agent not initialized.");
        }
        return globalInstrumentation.getObjectSize(object);
    }
}
```

**在我们为这个代理创建 JAR 之前，我们需要确保一个简单的元文件`MANIFEST.MF`包含在其中**:

```java
Premain-class: com.baeldung.objectsize.InstrumentationAgent
```

现在我们可以用清单制作一个代理罐子。包括 MF 文件。一种方法是通过命令行:

```java
javac InstrumentationAgent.java
jar cmf MANIFEST.MF InstrumentationAgent.jar InstrumentationAgent.class
```

### 3.2。示例类

让我们通过创建一个使用我们的代理类的示例对象的类来看看这一点:

```java
public class InstrumentationExample {

    public static void printObjectSize(Object object) {
        System.out.println("Object type: " + object.getClass() +
          ", size: " + InstrumentationAgent.getObjectSize(object) + " bytes");
    }

    public static void main(String[] arguments) {
        String emptyString = "";
        String string = "Estimating Object Size Using Instrumentation";
        String[] stringArray = { emptyString, string, "com.baeldung" };
        String[] anotherStringArray = new String[100];
        List<String> stringList = new ArrayList<>();
        StringBuilder stringBuilder = new StringBuilder(100);
        int maxIntPrimitive = Integer.MAX_VALUE;
        int minIntPrimitive = Integer.MIN_VALUE;
        Integer maxInteger = Integer.MAX_VALUE;
        Integer minInteger = Integer.MIN_VALUE;
        long zeroLong = 0L;
        double zeroDouble = 0.0;
        boolean falseBoolean = false;
        Object object = new Object();

        class EmptyClass {
        }
        EmptyClass emptyClass = new EmptyClass();

        class StringClass {
            public String s;
        }
        StringClass stringClass = new StringClass();

        printObjectSize(emptyString);
        printObjectSize(string);
        printObjectSize(stringArray);
        printObjectSize(anotherStringArray);
        printObjectSize(stringList);
        printObjectSize(stringBuilder);
        printObjectSize(maxIntPrimitive);
        printObjectSize(minIntPrimitive);
        printObjectSize(maxInteger);
        printObjectSize(minInteger);
        printObjectSize(zeroLong);
        printObjectSize(zeroDouble);
        printObjectSize(falseBoolean);
        printObjectSize(Day.TUESDAY);
        printObjectSize(object);
        printObjectSize(emptyClass);
        printObjectSize(stringClass);
    }

    public enum Day {
        MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY
    }
}
```

为此，**我们需要在运行我们的应用程序**时，在代理 JAR 的路径中包含–`javaagent`选项:

```java
VM Options: -javaagent:"path_to_agent_directory\InstrumentationAgent.jar"
```

运行我们的类的输出将向我们显示估计的对象大小:

```java
Object type: class java.lang.String, size: 24 bytes
Object type: class java.lang.String, size: 24 bytes
Object type: class [Ljava.lang.String;, size: 32 bytes
Object type: class [Ljava.lang.String;, size: 416 bytes
Object type: class java.util.ArrayList, size: 24 bytes
Object type: class java.lang.StringBuilder, size: 24 bytes
Object type: class java.lang.Integer, size: 16 bytes
Object type: class java.lang.Integer, size: 16 bytes
Object type: class java.lang.Integer, size: 16 bytes
Object type: class java.lang.Integer, size: 16 bytes
Object type: class java.lang.Long, size: 24 bytes
Object type: class java.lang.Double, size: 24 bytes
Object type: class java.lang.Boolean, size: 16 bytes
Object type: class com.baeldung.objectsize.InstrumentationExample$Day, size: 24 bytes
Object type: class java.lang.Object, size: 16 bytes
Object type: class com.baeldung.objectsize.InstrumentationExample$1EmptyClass, size: 16 bytes
Object type: class com.baeldung.objectsize.InstrumentationExample$1StringClass, size: 16 bytes
```

## 4.**结论**

在本文中，我们描述了 Java 中特定类型如何使用内存，JVM 如何存储数据，并强调了会影响总内存消耗的因素。然后，我们演示了如何在实践中获得 Java 对象的估计大小。

和往常一样，与本文相关的完整代码可以在 [GitHub 项目](https://web.archive.org/web/20221205184948/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-jvm)中找到。