# java.security.egd JVM 选项

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-security-egd>

## 1。概述

当启动 Java 虚拟机(JVM)时，我们可以定义各种属性来改变 JVM 的行为。一个这样的属性是`java.security.egd.`

在本教程中，我们将检查它是什么，如何使用它，以及它有什么影响。

## 2.什么是`java.security.egd`？

作为一个 JVM 属性，我们可以使用`java.security.egd`来影响 [`SecureRandom`](/web/20220806183929/https://www.baeldung.com/java-secure-random) 类如何初始化。

像所有 JVM 属性一样，当启动 JVM 时，我们在命令行中使用`-D` 参数来声明它:

```java
java -Djava.security.egd=file:/dev/urandom -cp . com.baeldung.java.security.JavaSecurityEgdTester
```

通常，如果我们运行 Java 8 或更高版本，并且运行在 Linux 上，那么我们的 JVM 将默认使用`file:/dev/urandom`。

## 3.`java.security.egd` 有什么作用？

当我们第一次调用从`SecureRandom`读取字节时，我们使它初始化并读取 JVM 的`java.security` 配置文件`.` **该文件包含一个`securerandom.source`属性**:

```java
securerandom.source=file:/dev/random
```

**安全提供者，比如默认的`sun.security.provider.Sun`，在初始化**时读取这个属性。

当我们设置我们的`java.security.egd` JVM 属性时，安全提供者可能用它来覆盖在`securerandom.source`中配置的属性。

当我们使用`SecureRandom`生成随机数时，`java.security.egd`和`securerandom.source` 共同控制哪个`entropy gathering device` (EGD)将被用作种子数据的主要来源。

直到 Java 8，我们在`$JAVA_HOME/jre/lib/security`中找到了`java.security`，但是在后来的实现中，它在`$JAVA_HOME/conf/security`中。

**`egd`选项是否有效取决于安全提供者的实现。**

## 4.`java.security.egd` 可以取什么值？

我们可以在 URL 格式中用如下值指定`java.security.egd` :

*   `file:/dev/random`
*   `file:/dev/urandom`
*   `file:/dev/./urandom`

这个设置是否有效，或者其他值是否有影响，取决于我们使用的平台和 Java 版本，以及我们的 JVM 的安全性是如何配置的。

在基于 Unix 的操作系统(OS)上，`/dev/random`是一个特殊的文件路径，它以普通文件的形式出现在文件系统中，但是对它的读取实际上与 OS 的设备驱动程序进行交互，以生成随机数。一些设备实现还提供通过`/dev/urandom`甚至`/dev/arandom` URIs 的访问。

## 5.`file:/dev/./urandom`有什么特别之处？

首先，让我们了解一下文件`/dev/random`和`/dev/urandom:`的区别

*   `/dev/random`从各种来源收集熵；`/dev/random`将阻塞，直到它有足够的熵来满足我们对不可预测数据的读取请求
*   `/dev/urandom`将从任何可用的东西中导出伪随机性，没有阻塞。

当我们第一次使用`SecureRandom`时，我们默认的 Sun `SeedGenerator`初始化。

当我们使用特殊值`file:/dev/random`或`file:/dev/urandom`时，我们让 Sun `SeedGenerator`使用本地(平台)实现。

Unix 上的 Provider 实现可能会因仍从`/dev/random`读取而阻塞。在 Java 1.4 中，一些实现被发现有这个问题。该漏洞随后在 Java 8 的 [JDK 增强提案(JEP 123)](https://web.archive.org/web/20220806183929/https://openjdk.java.net/jeps/123) 下被修复。

**使用一个 URL，比如`file:/dev/./urandom`，或者任何其他值，导致`SeedGenerator`将它视为我们想要使用的种子源的 URL**。

在类 Unix 系统上，我们的`file:/dev/./urandom` URL 解析为同一个非阻塞的`/dev/urandom`文件。

然而，**我们并不总是想用这个值。**在 Windows 上，我们没有这个文件，所以我们的 URL 无法解析。这触发了一个产生随机性的最终机制，并可能使我们的初始化延迟大约 5 秒。

## 6.`SecureRandom`的演变

在不同的 Java 版本中,`java.security.egd`的效果已经发生了变化。

那么，让我们来看看影响`SecureRandom`行为的一些更重要的事件:

*   **Java 1.4**
    *   在 [JDK-4705093 下出现的/dev/random 阻塞问题:如果/dev/random 存在，请使用/dev/urandom 而不是/dev/random](https://web.archive.org/web/20220806183929/https://bugs.java.com/bugdatabase/view_bug.do?bug_id=4705093)
*   **Java 5**
    *   修为 [JDK-4705093](https://web.archive.org/web/20220806183929/https://bugs.java.com/bugdatabase/view_bug.do?bug_id=4705093)
        *   添加了`NativePRNG` 算法以尊重`java.security.egd`设置，但我们需要手动配置它
        *   如果使用`SHA1PRNG` ，那么如果我们使用`file:/dev/urandom.`之外的任何东西，它可能会阻塞。换句话说，如果我们使用`file:/dev/./urandom` ，它可能会阻塞
*   **Java 8**
    *   [JEP123:可配置的安全随机数生成](https://web.archive.org/web/20220806183929/https://openjdk.java.net/jeps/123)
        *   添加新的`SecureRandom`实现，它考虑了安全属性
        *   增加了一个新的`getInstanceStrong()`方法，用于平台原生的强随机数。非常适合生成高价值和长期的秘密，如 RSA 私钥/公钥对
        *   [我们不再需要`file:/dev/./urandom`解决方案](https://web.archive.org/web/20220806183929/https://docs.oracle.com/javase/8/docs/technotes/guides/security/enhancements-8.html)
*   **Java 9**
    *   [JEP273:基于 DRBG 的 SecureRandom 实现](https://web.archive.org/web/20220806183929/https://openjdk.java.net/jeps/273)
        *   实施三种确定性随机位生成器(DRBG)机制，如[使用确定性随机位生成器生成随机数的建议](https://web.archive.org/web/20220806183929/http://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-90Ar1.pdf "National Institute of Standards and Technology - Recommendation for Random Number Generation Using Deterministic Random Bit Generators")中所述

理解`SecureRandom`是如何变化的，让我们深入了解了`java.security.egd`属性的可能影响。

## 7.测试`java.security.egd`的效果

确定 JVM 属性效果的最好方法是尝试一下。所以，**让我们通过运行一些代码**来创建一个新的`SecureRandom`并计时多长时间得到一些随机字节来看看`java.security.egd`的效果。

首先，让我们用一个`main()`方法创建一个`JavaSecurityEgdTester`类。我们将使用`System.nanoTime()`为我们对`secureRandom.nextBytes()`的呼叫计时，并显示结果:

```java
public class JavaSecurityEgdTester {
    public static final double NANOSECS = 1000000000.0;

    public static void main(String[] args) {
        SecureRandom secureRandom = new SecureRandom();
        long start = System.nanoTime();
        byte[] randomBytes = new byte[256];
        secureRandom.nextBytes(randomBytes);
        double duration = (System.nanoTime() - start) / NANOSECS;

        System.out.println("java.security.egd = " + System.getProperty("java.security.egd") + " took " + duration + " seconds and used the " + secureRandom.getAlgorithm() + " algorithm");
    }
}
```

现在，让我们通过启动新的 Java 实例并为`java.security.egd`属性指定一个值来运行一个`JavaSecurityEgdTester`测试:

```java
java -Djava.security.egd=file:/dev/random -cp . com.baeldung.java.security.JavaSecurityEgdTester
```

让我们检查输出，看看我们的测试花了多长时间，使用了哪种算法:

```java
java.security.egd=file:/dev/random took 0.692 seconds and used the SHA1PRNG algorithm
```

因为我们的系统属性只在初始化时读取，所以让我们在一个新的 JVM 中为每个不同的`java.security.egd`值启动我们的类:

```java
java -Djava.security.egd=file:/dev/urandom -cp . com.baeldung.java.security.JavaSecurityEgdTester
java -Djava.security.egd=file:/dev/./urandom -cp . com.baeldung.java.security.JavaSecurityEgdTester
java -Djava.security.egd=baeldung -cp . com.baeldung.java.security.JavaSecurityEgdTester 
```

**在使用 Java 8 或 Java 11 的 Windows 上，使用值`file:/dev/random`或`file:/dev/urandom`的测试给出亚秒时间。使用其他任何东西，比如`file:/dev/./urandom`，甚至`baeldung`，都会使我们的测试花费超过 5 秒钟！**

关于为什么会发生这种情况，请参见我们之前的章节。在 Linux 上，我们可能会得到不同的结果。

## 8.那`SecureRandom.getInstanceStrong()`呢？

Java 8 引入了一个 [`SecureRandom.getInstanceStrong()`](https://web.archive.org/web/20220806183929/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/security/SecureRandom.html#getInstanceStrong()) 方法。让我们看看这会如何影响我们的结果。

首先，让我们把我们的`new SecureRandom()`换成`SecureRandom.` `getInstanceStrong()`:

```java
SecureRandom secureRandom = SecureRandom.getInstanceStrong();
```

现在，让我们再次运行测试:

```java
java -Djava.security.egd=file:/dev/random -cp . com.baeldung.java.security.JavaSecurityEgdTester
```

**在 Windows 上运行时，当我们使用`SecureRandom.getInstanceStrong()`时，`java.security.egd` 属性的值没有明显的影响。**即使是无法识别的值，我们也能快速响应。

让我们再次检查我们的输出，注意不到 0.01 秒的时间。我们还可以观察到，该算法现在是 Windows-PRNG 算法:

```java
java.security.egd=baeldung took 0.003 seconds and used the Windows-PRNG algorithm
```

注意，算法名称中的 PRNG 代表伪随机数发生器。

## 9.播种算法

因为随机数在加密中大量用于安全密钥，所以它们需要是不可预测的。

因此，我们如何播种我们的算法直接影响它们产生的随机数的可预测性。

**为了产生不可预测性，`SecureRandom`实现使用从累积输入中收集的熵来播种它们的算法。**这来自鼠标和键盘等 IO 设备。

**在类 Unix 系统上，我们的熵在文件`/dev/random`** 中累积。

Windows 上没有`/dev/random`文件**。** **将`-Djava.security.egd`设置为`file:/dev/random`或`file:/dev/urandom`会导致默认算法(SHA1PRNG)使用原生微软加密 API 播种**。

## 10.虚拟机呢？

有时，我们的应用程序可能运行在一个虚拟机中，该虚拟机在`/dev/random`中很少或没有熵聚集。

**虚拟机没有物理鼠标或键盘来生成数据**，因此`/dev/random`中的熵积累要慢得多。**这可能会导致我们的默认`SecureRandom`调用阻塞，直到有足够的熵让它**生成一个不可预测的数字。

我们可以采取一些措施来减轻这种情况。例如，当在 RedHat Linux 中运行 VM 时，系统管理员可以[配置一个虚拟 IO 随机数生成器，`virtio-rng`](https://web.archive.org/web/20220806183929/https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/virtualization_deployment_and_administration_guide/sect-guest_virtual_machine_device_configuration-random_number_generator_device) 。它从托管它的物理机器上读取熵。

## 11.故障排除提示

如果我们的应用程序在它或它的依赖项生成`SecureRandom`数字时挂起，考虑一下`java.security.egd`——特别是，当我们运行在 Linux 上，如果我们运行在 Java 8 之前的版本上。

**我们的 Spring Boot 应用经常使用嵌入式** [**Tomcat**](/web/20220806183929/https://www.baeldung.com/tomcat) 。这使用`SecureRandom` s 来生成会话密钥。**当我们看到 Tomcat 的“创建 SecureRandom 实例”操作花费了 5 秒或更长时间时，我们应该为** `**java.security.egd**.`尝试不同的值

## 12.结论

在本教程中，我们学习了 JVM 属性`java.security.egd` 是什么，如何使用它，以及它有什么作用。我们还发现，它的效果会因我们运行的平台和我们使用的 Java 版本而异。

最后一点，我们可以在 JCA 参考指南的 [SecureRandom 部分和](https://web.archive.org/web/20220806183929/https://docs.oracle.com/en/java/javase/11/security/java-cryptography-architecture-jca-reference-guide.html#GUID-AEB77CD8-D28F-4BBE-B9E5-160B5DC35D36) [SecureRandom API 规范](https://web.archive.org/web/20220806183929/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/security/SecureRandom.html)中阅读更多关于`SecureRandom`及其工作原理的内容，并了解一些关于 urandom 的[神话。](https://web.archive.org/web/20220806183929/https://www.2uo.de/myths-about-urandom/)

像往常一样，代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220806183929/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-security-2)