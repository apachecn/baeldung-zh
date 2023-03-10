# 检查 Java 程序是在 64 位还是 32 位 JVM 中运行

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-detect-jvm-64-or-32-bit>

## 1.概观

尽管 Java 是独立于平台的，但有时我们不得不使用本地库。在这些情况下，我们可能需要识别底层平台，并在启动时加载适当的本地库。

在本教程中，我们将学习不同的方法来检查 Java 程序是否运行在 64 位或 32 位 JVM 上。

首先，我们将展示如何使用`System`类来实现这一点。

然后，我们将看到如何使用 [Java 本地访问](https://web.archive.org/web/20221208143832/https://github.com/java-native-access/jna) (JNA) API 来检查 JVM 的位。JNA 是一个社区开发的图书馆，支持所有本地访问。

## 2.使用`sun.arch.data.model`系统属性

Java 中的`System`类提供了对外部定义的属性和环境变量的访问。它维护一个描述当前工作环境配置的`Properties`对象。

我们可以使用“`sun.arch.data.model`”系统属性来标识 JVM 位:

```java
System.getProperty("sun.arch.data.model"); 
```

它包含“32”或“64”，分别表示 32 位或 64 位 JVM。虽然这种方法很容易使用，但如果属性不存在，它会返回“未知”。因此，它只适用于 Oracle Java 版本。

让我们看看代码:

```java
public class JVMBitVersion {
    public String getUsingSystemClass() {
        return System.getProperty("sun.arch.data.model") + "-bit";
    }

    //... other methods
} 
```

让我们通过单元测试来检查这种方法:

```java
@Test
public void whenUsingSystemClass_thenOutputIsAsExpected() {
    if ("64".equals(System.getProperty("sun.arch.data.model"))) {
        assertEquals("64-bit", jvmVersion.getUsingSystemClass());
    } else if ("32".equals(System.getProperty("sun.arch.data.model"))) {
        assertEquals("32-bit", jvmVersion.getUsingSystemClass());
    }
}
```

## 3.使用 JNA API

JNA ( [Java Native Access](https://web.archive.org/web/20221208143832/https://github.com/java-native-access/jna) )支持 macOS、微软 Windows、Solaris、GNU、Linux 等各种平台。

它使用本地函数按名称加载一个库，并检索指向该库中某个函数的指针。

### 3.1.`Native`阶级

我们可以使用来自`Native`类的`POINTER_SIZE`。此常数指定当前平台上本机指针的大小(以字节为单位)。

值 4 表示 32 位本机指针，而值 8 表示 64 位本机指针:

```java
if (com.sun.jna.Native.POINTER_SIZE == 4) {
    // 32-bit
} else if (com.sun.jna.Native.POINTER_SIZE == 8) {
    // 64-bit
}
```

### 3.2.`Platform`阶级

或者，我们可以使用`Platform`类，它提供简化的平台信息。

它包含**检测 JVM 是否是 64 位的**的`is64Bit()`方法。

让我们看看它是如何识别比特的:

```java
public static final boolean is64Bit() {
    String model = System.getProperty("sun.arch.data.model",
                                      System.getProperty("com.ibm.vm.bitmode"));
    if (model != null) {
        return "64".equals(model);
    }
    if ("x86-64".equals(ARCH)
        || "ia64".equals(ARCH)
        || "ppc64".equals(ARCH) || "ppc64le".equals(ARCH)
        || "sparcv9".equals(ARCH)
        || "mips64".equals(ARCH) || "mips64el".equals(ARCH)
        || "amd64".equals(ARCH)
        || "aarch64".equals(ARCH)) {
        return true;
    }
    return Native.POINTER_SIZE == 8;
}
```

在这里，`ARCH`常量是通过`System`类从属性`os.arch`中派生出来的。它用于获取操作系统架构:

```java
ARCH = getCanonicalArchitecture(System.getProperty("os.arch"), osType);
```

**这种方法适用于不同的操作系统以及不同的 JDK 供应商。**因此，它比“`sun.arch.data.model`”系统属性更可靠。

## 4.结论

在本教程中，我们学习了如何检查 JVM 位版本。我们还观察了 JNA 如何在不同平台上为我们简化解决方案。

和往常一样，完整的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221208143832/https://github.com/eugenp/tutorials/tree/master/java-native)