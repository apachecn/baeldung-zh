# Java 17 中的新特性

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-17-new-features>

## 1.概观

在本教程中，我们将谈论与新版本 Java 生态系统相关的新闻， [Java SE 17](https://web.archive.org/web/20220907212832/https://jdk.java.net/17/release-notes) ，包括新功能及其发布过程中的变化，LTS 支持和许可证。

## 2.jep 列表

首先，我们来谈谈在 Java 开发人员的生活中，有哪些可以影响到日常工作。

### 2.1.恢复总是严格的浮点语义( [JEP 306](https://web.archive.org/web/20220907212832/https://openjdk.java.net/jeps/306)

这个 JEP 主要用于科学应用，它使得浮点运算始终如一地严格。**默认的浮点运算是`strict`或`strictfp`，两者都保证每个平台上的浮点计算结果相同。**

在 Java 1.2 之前，`strictfp`行为也是默认行为。然而，由于硬件问题，架构师发生了变化，关键字`strictfp`对于重新启用这种行为是必要的。所以，没有必要再用这个关键词了。

### 2.2.增强型伪随机数发生器( [JEP 356](https://web.archive.org/web/20220907212832/https://openjdk.java.net/jeps/356) )

还与更特殊的用例相关，JEP 356 为伪随机数发生器(PRNG)提供了新的接口和实现。

因此，更容易互换使用不同的算法，并且它还为基于流的编程提供了更好的支持:

```java
public IntStream getPseudoInts(String algorithm, int streamSize) {
    // returns an IntStream with size @streamSize of random numbers generated using the @algorithm
    // where the lower bound is 0 and the upper is 100 (exclusive)
    return RandomGeneratorFactory.of(algorithm)
            .create()
            .ints(streamSize, 0,100);
}
```

遗留的随机类，如`java.util.Random`、`SplittableRandom`和`SecureRandom`现在扩展了新的`RandomGenerator`接口。

### 2.3.新的 macOS 渲染管道( [JEP 382](https://web.archive.org/web/20220907212832/https://openjdk.java.net/jeps/382) )

这个 JEP 为 macOS 实现了一个 Java 2D 内部渲染管道，因为 Apple 不赞成在 Swing GUI 内部使用 OpenGL API(在 macOS 10.14 中)。新的实现使用 Apple Metal API，除了内部引擎之外，现有 API 没有任何变化。

### 2.4.`macOS/AArch64`港口( [JEP 391](https://web.archive.org/web/20220907212832/https://openjdk.java.net/jeps/391)

苹果宣布了一项长期计划，将电脑产品线从 X64 过渡到 AArch64。这个 JEP 将 JDK 移植到 macOS 平台的 AArch64 上运行。

### 2.5.不赞成删除小程序 API([JEP 398](https://web.archive.org/web/20220907212832/https://openjdk.java.net/jeps/398))

虽然这对于许多使用 Applet APIs 开始开发生涯的 Java 开发人员来说可能是悲哀的，但许多 web 浏览器已经取消了对 Java 插件的支持。由于 API 变得不相关，这个版本将它标记为删除，即使从版本 9 开始它已经被标记为不推荐使用。

### 2.6.强力封装 JDK 堆内构件( [JEP 403](https://web.archive.org/web/20220907212832/https://openjdk.java.net/jeps/403) )

JEP 403 代表着向强封装 JDK 内部迈出了又一步，因为它移除了标志`–illegal-access`。平台将忽略该标志，如果该标志存在，控制台将发出消息通知该标志停止。

**该功能将阻止 JDK 用户访问内部 API，除了像`sun.misc.Unsafe`这样的关键 API。**

### 2.7.开关模式匹配(预览)( [JEP 406](https://web.archive.org/web/20220907212832/https://openjdk.java.net/jeps/406)

通过增强对`switch`表达式和语句的模式匹配，这是迈向模式匹配的又一步。它减少了定义这些表达式所需的样板文件，提高了语言的表现力。

让我们看两个新功能的例子:

```java
 static record Human (String name, int age, String profession) {}

public String checkObject(Object obj) {
    return switch (obj) {
        case Human h -> "Name: %s, age: %s and profession: %s".formatted(h.name(), h.age(), h.profession());
        case Circle c -> "This is a circle";
        case Shape s -> "It is just a shape";
        case null -> "It is null";
        default -> "It is an object";
    };
}

public String checkShape(Shape shape) {
    return switch (shape) {
        case Triangle t && (t.getNumberOfSides() != 3) -> "This is a weird triangle";
        case Circle c && (c.getNumberOfSides() != 0) -> "This is a weird circle";
        default -> "Just a normal shape";
    };
} 
```

### 2.8.移除 RMI 激活( [JEP 407](https://web.archive.org/web/20220907212832/https://openjdk.java.net/jeps/407) )

该 JEP 在版本 15 中被标记为删除，在版本 17 中从平台中删除了 RMI 激活 API。

### 2.9.密封类( [JEP 409](https://web.archive.org/web/20220907212832/https://openjdk.java.net/jeps/409) )

密封类是琥珀项目的一部分，这次 JEP 正式引入了这种语言的新特性，尽管它在 JDK 版本 [15](https://web.archive.org/web/20220907212832/https://openjdk.java.net/jeps/360) 和 [16](https://web.archive.org/web/20220907212832/https://openjdk.java.net/jeps/397) 的预览模式中可用。

该功能限制了哪些其他类或接口可以扩展或实现密封组件。示出了与结合 JEP 406 的模式匹配相关的另一个改进，这将允许对类型、造型和动作代码模式进行更复杂和更清晰的检查。

让我们来看看它的实际应用:

```java
 int getNumberOfSides(Shape shape) {
    return switch (shape) {
        case WeirdTriangle t -> t.getNumberOfSides();
        case Circle c -> c.getNumberOfSides();
        case Triangle t -> t.getNumberOfSides();
        case Rectangle r -> r.getNumberOfSides();
        case Square s -> s.getNumberOfSides();
    };
} 
```

### 2.10.移除实验性的 AOT 和 JIT 编译器( [JEP 410](https://web.archive.org/web/20220907212832/https://openjdk.java.net/jeps/410) )

JDK 9 和 JDK 10 分别引入了超前(AOT)编译( [JEP 295](https://web.archive.org/web/20220907212832/https://openjdk.java.net/jeps/295) )和来自 GraalVM 的实时(JIT)编译器( [JEP-317](https://web.archive.org/web/20220907212832/https://openjdk.java.net/jeps/317) )作为实验特性，这些特性的维护成本很高。

另一方面，它们没有被大量采用。正因为如此，JEP 将它们从平台中移除，但是开发者仍然可以使用 [GraalVM](https://web.archive.org/web/20220907212832/https://www.graalvm.org/) 来利用它们。

### 2.11.反对安全管理器被删除( [JEP 411](https://web.archive.org/web/20220907212832/https://openjdk.java.net/jeps/411) )

旨在保护客户端 Java 代码的安全管理器是另一个由于不再相关而被删除的特性。

### 2.12.外来函数和内存 API(孵化器)( [JEP 412](https://web.archive.org/web/20220907212832/https://openjdk.java.net/jeps/412) )

外部函数和内存 API 允许 Java 开发人员从 JVM 外部访问代码，并管理堆外的内存。目标是取代 [JNI API](https://web.archive.org/web/20220907212832/https://docs.oracle.com/en/java/javase/16/docs/specs/jni/index.html) ，并相对于旧的 API 提高安全性和性能。

这个 API 是巴拿马项目[开发的另一个特性，经过 JEPs](https://web.archive.org/web/20220907212832/https://openjdk.java.net/projects/panama/) [393](https://web.archive.org/web/20220907212832/https://openjdk.java.net/jeps/393) 、 [389](https://web.archive.org/web/20220907212832/https://openjdk.java.net/jeps/389) 、 [383](https://web.archive.org/web/20220907212832/https://openjdk.java.net/jeps/383) 和 [370](https://web.archive.org/web/20220907212832/https://openjdk.java.net/jeps/370) 的演化和预解码。

有了这个特性，我们可以从 Java 类调用 C 库:

```java
 private static final SymbolLookup libLookup;

static {
    // loads a particular C library
    var path = JEP412.class.getResource("/print_name.so").getPath();
    System.load(path);
    libLookup = SymbolLookup.loaderLookup();
} 
```

首先，需要加载我们希望通过 API 调用的目标库。

接下来，我们需要指定目标方法的签名，最后调用它:

```java
 public String getPrintNameFormat(String name) {

    var printMethod = libLookup.lookup("printName");

    if (printMethod.isPresent()) {
        var methodReference = CLinker.getInstance()
            .downcallHandle(
                printMethod.get(),
                MethodType.methodType(MemoryAddress.class, MemoryAddress.class),
                FunctionDescriptor.of(CLinker.C_POINTER, CLinker.C_POINTER)
            );

        try {
            var nativeString = CLinker.toCString(name, newImplicitScope());
            var invokeReturn = methodReference.invoke(nativeString.address());
            var memoryAddress = (MemoryAddress) invokeReturn;
            return CLinker.toJavaString(memoryAddress);
        } catch (Throwable throwable) {
            throw new RuntimeException(throwable);
        }
    }
    throw new RuntimeException("printName function not found.");
} 
```

### 2.13.Vector API(第二个孵化器)( [JEP 414](https://web.archive.org/web/20220907212832/https://openjdk.java.net/jeps/414)

Vector API 处理 SIMD(单指令、多数据)类型的操作，这意味着多组指令并行执行。它利用支持向量指令的专用 CPU 硬件，并允许以流水线方式执行此类指令。

因此，新的 API 将使开发人员能够实现更高效的代码，充分利用底层硬件的潜力。

这种操作的日常用例是科学代数线性应用、图像处理、字符处理以及任何繁重的算术应用或任何需要对多个独立操作数应用操作的应用。

让我们使用 API 来说明一个简单的向量乘法示例:

```java
 public void newVectorComputation(float[] a, float[] b, float[] c) {
    for (var i = 0; i < a.length; i += SPECIES.length()) {
        var m = SPECIES.indexInRange(i, a.length);
        var va = FloatVector.fromArray(SPECIES, a, i, m);
        var vb = FloatVector.fromArray(SPECIES, b, i, m);
        var vc = va.mul(vb);
        vc.intoArray(c, i, m);
    }
}

public void commonVectorComputation(float[] a, float[] b, float[] c) {
    for (var i = 0; i < a.length; i ++) {
        c[i] = a[i] * b[i];
    }
} 
```

### 2.14.特定于上下文的反序列化过滤器( [JEP 415](https://web.archive.org/web/20220907212832/https://openjdk.java.net/jeps/415) )

首次在 JDK 9 中引入的 JEP 290 使我们能够验证来自不可信来源的传入序列化数据，这是许多安全问题的常见来源。这种验证发生在 JVM 级别，提供了更高的安全性和健壮性。

使用 JEP 415，应用程序可以配置在 JVM 级别定义的特定于上下文和动态选择的反序列化过滤器。每个反序列化操作都会调用这样的过滤器。

## 3.LTS 定义

变化不仅仅停留在代码中——流程也在变化。

众所周知，Java 平台版本历史悠久且不精确。尽管设计的发布间隔是三年，但它经常变成四年的过程。

此外，鉴于市场的新动态，创新和快速响应变得势在必行，负责平台发展的团队决定改变发布节奏以适应新的现实。

因此，从 Java 10(2018 年 3 月 20 日发布)开始，采用了新的半年特性发布模式。

### 3.1.六个月功能发布模型

新的六个月特性发布模式允许平台开发者在准备好的时候发布特性。这消除了将特征推入释放的压力。否则，他们将不得不等待三到四年才能向平台用户提供该功能。

新模型还改善了用户和平台架构师之间的反馈周期。这是因为功能可以在孵化模式下可用，并且只在几次交互后发布供一般使用。

### 3.2.LTS 模型

由于企业应用程序广泛使用 Java，稳定性至关重要。此外，为所有这些版本提供支持和补丁更新的成本很高。

因此，创建了长期支持(LTS)版本，为用户提供扩展支持。因此，由于错误修复、性能改进和安全补丁，这些版本自然变得更加稳定和安全。对于 Oracle，这种支持通常持续八年。

自从引入发布模型的变化以来，LTS 版本是 Java SE 11(2018 年 9 月发布)和 Java SE 17(2021 年 9 月发布)。尽管如此，版本 17 为模型带来了一些新的东西。简而言之，LTS 版本之间的间隔现在是两年而不是三年，这使得 Java 21(计划于 2023 年 9 月发布)很可能成为下一个 LTS。

还有一点值得一提的是，这种发布模式并不新鲜。它被无耻地复制并改编自其他项目，如 Mozilla Firefox、Ubuntu 和其他证明了自己的模型。

## 4.新发布流程

鉴于这篇文章描述了整个过程中的所有变化，我们将它建立在 JEP 3 的基础上。详情请查看。我们将尝试在这里提供一个简明的总结。

鉴于上面描述的新模型，结合平台的持续开发和新的六个月发布节奏(一般是六月和十二月)，Java 会走得更快。JDK 的开发团队将按照下面描述的流程启动下一个功能发布的发布周期。

该过程从[主线](https://web.archive.org/web/20220907212832/https://github.com/openjdk/jdk)的分叉开始。然后开发在一个稳定的存储库中继续，JDK/JDK$N(例如， [JDK17](https://web.archive.org/web/20220907212832/https://github.com/openjdk/jdk17) )。在那里，开发继续集中在版本的稳定性上。

在我们更深入地研究这个过程之前，让我们澄清一些术语:

*   **Bugs** :在这个上下文中，Bugs 指的是票证或任务:
    *   **当前**:这些或者是与当前版本(即将发布的新版本)相关的实际 bug，或者是对已经包含在这个版本中的新特性的调整(新 JEPs)。
    *   **目标**:与旧版本相关，计划在新版本中修复或解决
*   **优先级**:从 P1 到 P5，其中 P1 最重要，重要性逐渐降低到 P5

### 4.1.新格式

稳定过程将持续三个月:

*   JDK/JDK$N 存储库就像一个发布分支，在这一点上，没有新的 jep 进入存储库。
*   接下来，这个存储库中的开发将被稳定并移植到其他开发继续进行的主线上。
*   第一阶段(RDP 新协议):持续四至五周。开发者放弃所有当前的 P4-P5 和目标 P1-P3(取决于[推迟](https://web.archive.org/web/20220907212832/https://openjdk.java.net/jeps/3#Bug-Deferral-Process)、[修复](https://web.archive.org/web/20220907212832/https://openjdk.java.net/jeps/3#Fix-Request-Process)或[增强](https://web.archive.org/web/20220907212832/https://openjdk.java.net/jeps/3#Late-Enhancement-Request-Process))。这意味着 P5+测试/文档错误和目标 P3+代码错误是可选的。
*   第二阶段(新 RDP 协议):持续三到四周。现在他们推迟了所有当前的 P3-P5 和目标 P1-P3(取决于[推迟](https://web.archive.org/web/20220907212832/https://openjdk.java.net/jeps/3#Bug-Deferral-Process)、[修复](https://web.archive.org/web/20220907212832/https://openjdk.java.net/jeps/3#Fix-Request-Process)或[增强](https://web.archive.org/web/20220907212832/https://openjdk.java.net/jeps/3#Late-Enhancement-Request-Process))。
*   最后，团队发布一个候选版本，并向公众开放。这个阶段持续两到五周，并且只处理当前的 P1 补丁(使用[补丁](https://web.archive.org/web/20220907212832/https://openjdk.java.net/jeps/3#Fix-Request-Process))。

一旦所有这些周期结束，新版本就成为正式发布(GA)版本。

## 5.下一步是什么？

JDK 建筑师继续致力于许多项目，旨在现代化的平台。目标是提供更好的开发体验和更健壮、更高性能的 API。

因此，JDK 18 应该会在六个月后推出，尽管这个版本不太可能包含重大或破坏性的变化。我们可以在官方的 [OpenJDK 项目](https://web.archive.org/web/20220907212832/https://openjdk.java.net/projects/jdk/18/)门户中关注这个版本的 jep 建议列表。

另一条影响当前和未来版本的相关新闻是适用于甲骨文 JDK 发行版(或 Hotspot)的新的[免费条款和条件](https://web.archive.org/web/20220907212832/https://www.oracle.com/downloads/licenses/no-fee-license.html)许可。在大多数情况下，Oracle 为生产和其他环境免费提供其发行版，但也有少数例外。再次请参考链接。

如前所述，**新流程的下一个 LTS 版本是版本 21，计划在 2023 年 9 月发布。**

## 6.结论

在本文中，我们看了关于新 Java 17 版本的新闻，回顾了它的最新发展、新功能、支持定义和发布周期过程。

像往常一样，本文中使用的所有代码示例都可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220907212832/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-17)