# 使用 JNA 访问本地动态库

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-jna-dynamic-libraries>

## 1.概观

在本教程中，我们将看到如何使用 Java 本地访问库(简称 JNA)来访问本地库，而无需编写任何 [JNI (Java 本地接口)](/web/20221101060427/https://www.baeldung.com/jni)代码。

## 2.为什么是 JNA？

多年来，Java 和其他基于 JVM 的语言在很大程度上实现了“编写一次，到处运行”的座右铭。然而，有时我们需要使用本地代码来实现某些功能:

*   重用用 C/C++或任何其他能够创建本机代码的语言编写的遗留代码
*   访问标准 Java 运行时中没有的特定于系统的功能
*   针对给定应用程序的特定部分优化速度和/或内存使用。

最初，这种需求意味着我们必须求助于 JNI——Java 本地接口。**虽然这种方法很有效，但它也有缺点，通常会因为一些问题而被避免:**

*   需要开发人员编写 C/C++“粘合代码”来桥接 Java 和本机代码
*   需要为每个目标系统提供完整的编译和链接工具链
*   在 JVM 之间封送和解组值是一项单调乏味且容易出错的任务
*   混合 Java 和本地库时的法律和支持问题

JNA 解决了与使用 JNI 相关的大部分复杂性。特别是，不需要创建任何 JNI 代码来使用位于动态库中的本地代码，这使得整个过程更加容易。

当然，也有一些权衡:

*   我们不能直接使用静态库
*   与手工制作的 JNI 代码相比，速度较慢

然而，对于大多数应用程序来说，JNA 的简单性远远超过了这些缺点。因此，公平地说，除非我们有非常具体的需求，否则今天的 JNA 可能是从 Java——或者任何其他基于 JVM 的语言——访问本机代码的最佳选择。

## 3.JNA 项目设置

要使用 JNA，我们必须做的第一件事是将它的依赖项添加到我们项目的`pom.xml`:

```java
<dependency>
    <groupId>net.java.dev.jna</groupId>
    <artifactId>jna-platform</artifactId>
    <version>5.6.0</version>
</dependency> 
```

[`jna-platform`](https://web.archive.org/web/20221101060427/https://search.maven.org/search?q=g:net.java.dev.jna%20a:jna-platform) 的最新版本可以从 Maven Central 下载。

## 4.使用 JNA

使用 JNA 分为两步:

*   首先，我们创建一个 Java 接口，它扩展了 JNA 的 [`Library`](https://web.archive.org/web/20221101060427/https://java-native-access.github.io/jna/5.6.0/javadoc/com/sun/jna/Library.html) 接口来描述调用目标本地代码时使用的方法和类型
*   接下来，我们将这个接口传递给 JNA，它返回这个接口的具体实现，我们用它来调用本地方法

### 4.1.从 C 标准库中调用方法

对于我们的第一个例子，让我们使用 JNA 从标准 C 库中调用 [`cosh`](https://web.archive.org/web/20221101060427/https://man7.org/linux/man-pages/man3/cosh.3.html) 函数，这在大多数系统中都是可用的。这个方法接受一个`double`参数，并计算它的[双曲余弦值](https://web.archive.org/web/20221101060427/https://en.wikipedia.org/wiki/Hyperbolic_functions)。A-C 程序只要包含`<math.h>`头文件就可以使用这个函数；

```java
#include <math.h>
#include <stdio.h>
int main(int argc, char** argv) {
    double v = cosh(0.0);
    printf("Result: %f\n", v);
}
```

让我们创建调用该方法所需的 Java 接口:

```java
public interface CMath extends Library { 
    double cosh(double value);
} 
```

接下来，我们使用 JNA 的 [`Native`](https://web.archive.org/web/20221101060427/https://java-native-access.github.io/jna/5.6.0/javadoc/com/sun/jna/Native.html) 类来创建这个接口的具体实现，这样我们就可以调用我们的 API:

```java
CMath lib = Native.load(Platform.isWindows()?"msvcrt":"c", CMath.class);
double result = lib.cosh(0); 
```

**这里真正有趣的部分是对`load()`方法**的调用。它有两个参数:动态库名和描述我们将使用的方法的 Java 接口。**它返回这个接口的具体实现，允许我们调用它的任何方法。**

现在，动态库名通常是系统相关的，C 标准库也不例外:在大多数基于 Linux 的系统中是`libc.so`，但在 Windows 中是`msvcrt.dll`。这就是为什么我们使用了包含在 JNA 中的`Platform`助手类来检查我们正在哪个平台上运行并选择正确的库名。

请注意，我们不必像暗示的那样添加`.so`或`.dll`扩展名。此外，对于基于 Linux 的系统，我们不需要指定共享库的标准前缀“lib”。

**因为从 Java 的角度来看，动态库的行为类似于[单态](/web/20221101060427/https://www.baeldung.com/java-singleton)，所以通常的做法是声明一个`INSTANCE`字段作为接口声明的一部分:**

```java
public interface CMath extends Library {
    CMath INSTANCE = Native.load(Platform.isWindows() ? "msvcrt" : "c", CMath.class);
    double cosh(double value);
} 
```

### 4.2.基本类型映射

在我们最初的例子中，被调用的方法只使用原始类型作为它的参数和返回值。JNA 自动处理这些情况，当从 C 类型映射时，通常使用它们自然的 Java 对应物:

*   `char => byte`
*   `short => short`
*   `wchar_t => char`
*   `int => int`
*   `long => com.sun.jna.NativeLong`
*   `long long => long`
*   `float => float`
*   `double => double`
*   `char * => String`

可能看起来奇怪的映射是用于本机`long`类型的映射。**这是因为，在 C/C++中，`long` 类型可能代表 32 位或 64 位值，这取决于我们运行的是 32 位还是 64 位系统。**

为了解决这个问题，JNA 提供了`NativeLong` 类型，它根据系统的架构使用合适的类型。

### 4.3.结构和联合

另一个常见的场景是处理本机代码 API，当创建 Java 接口来访问它时，这些 API 需要指向某个`struct`或`union` 类型`.` 的指针，相应的参数或返回值必须分别是扩展`Structure or Union`的 Java 类型。

例如，给定这个 C 结构:

```java
struct foo_t {
    int field1;
    int field2;
    char *field3;
};
```

它的 Java 对等类应该是:

```java
@FieldOrder({"field1","field2","field3"})
public class FooType extends Structure {
    int field1;
    int field2;
    String field3;
};
```

**JNA 需要`@FieldOrder`注释，以便在将数据用作目标方法的参数之前，能够正确地将数据序列化到内存缓冲区中。**

或者，我们可以覆盖`getFieldOrder()`方法以获得相同的效果。当以单一架构/平台为目标时，前一种方法通常就足够了。我们可以使用后者来处理跨平台的对齐问题，这有时需要添加一些额外的填充字段。

类似地工作，除了几点:

*   不需要使用`@FieldOrder`注释或实现`getFieldOrder()`
*   我们必须在调用本机方法之前调用`setType()`

让我们通过一个简单的例子来看看如何做到这一点:

```java
public class MyUnion extends Union {
    public String foo;
    public double bar;
}; 
```

现在，让我们对一个假想的库使用`MyUnion`:

```java
MyUnion u = new MyUnion();
u.foo = "test";
u.setType(String.class);
lib.some_method(u); 
```

如果`foo`和`bar`属于同一类型，我们必须使用字段名:

```java
u.foo = "test";
u.setType("foo");
lib.some_method(u);
```

### 4.4.使用指针

JNA 提供了一个 [`Pointer`](https://web.archive.org/web/20221101060427/https://java-native-access.github.io/jna/5.6.0/javadoc/com/sun/jna/Pointer.html) 抽象，帮助处理用非类型化指针声明的 APIs 通常是一个`void *`。**这个类提供了允许对底层本机内存缓冲区进行读写访问的方法，这有明显的风险。**

在开始使用这个类之前，我们必须确保清楚地了解每次谁“拥有”被引用的内存。不这样做可能会产生与内存泄漏和/或无效访问相关的难以调试的错误。

假设我们知道自己在做什么(一如既往)，让我们看看如何将众所周知的`malloc()` 和`free()` 函数用于 JNA，用来分配和释放内存缓冲区。首先，让我们再次创建我们的包装器接口:

```java
public interface StdC extends Library {
    StdC INSTANCE = // ... instance creation omitted
    Pointer malloc(long n);
    void free(Pointer p);
} 
```

现在，让我们用它来分配一个缓冲区并使用它:

```java
StdC lib = StdC.INSTANCE;
Pointer p = lib.malloc(1024);
p.setMemory(0l, 1024l, (byte) 0);
lib.free(p); 
```

`setMemory()`方法只是用一个常量字节值(在本例中为零)填充底层缓冲区。注意，`Pointer`实例不知道它指向什么，更不知道它的大小。**这意味着我们可以很容易地使用它的方法来破坏我们的堆。**

我们将在后面看到如何使用 JNA 的碰撞保护功能来减少这样的错误。

### 4.5.处理错误

旧版本的标准 C 库使用全局`errno`变量来存储特定调用失败的原因。例如，这是典型的`open()`调用在 C:

```java
int fd = open("some path", O_RDONLY);
if (fd < 0) {
    printf("Open failed: errno=%d\n", errno);
    exit(1);
}
```

当然，在现代的多线程程序中，这些代码是不起作用的，对吗？嗯，多亏了 C 的预处理器，开发人员仍然可以编写这样的代码，而且运行良好。**原来，现在的`errno`是一个扩展成函数调用的宏:**

```java
// ... excerpt from bits/errno.h on Linux
#define errno (*__errno_location ())

// ... excerpt from <errno.h> from Visual Studio
#define errno (*_errno())
```

现在，这种方法在编译源代码时工作得很好，但是在使用 JNA 时就没有这样的事情了。我们可以在包装器接口中声明扩展的函数并显式调用它，但是 JNA 提供了一个更好的选择: [`LastErrorException`](https://web.archive.org/web/20221101060427/https://java-native-access.github.io/jna/5.6.0/javadoc/com/sun/jna/LastErrorException.html) 。

用`throws LastErrorException`在包装器接口中声明的任何方法都将在本地调用后自动包含错误检查。如果它报告一个错误，JNA 将抛出一个`LastErrorException`，其中包括原始的错误代码。

让我们给之前使用过的`StdC`包装器接口添加几个方法来展示这个特性:

```java
public interface StdC extends Library {
    // ... other methods omitted
    int open(String path, int flags) throws LastErrorException;
    int close(int fd) throws LastErrorException;
} 
```

现在，我们可以在 try/catch 子句中使用`open()`:

```java
StdC lib = StdC.INSTANCE;
int fd = 0;
try {
    fd = lib.open("/some/path",0);
    // ... use fd
}
catch (LastErrorException err) {
    // ... error handling
}
finally {
    if (fd > 0) {
       lib.close(fd);
    }
} 
```

在`catch`块中，我们可以使用`LastErrorException.getErrorCode()`获得原始的`errno`值，并将其用作错误处理逻辑的一部分。

### 4.6.处理访问冲突

如前所述，JNA 不能防止我们误用给定的 API，尤其是在处理来回传递本机代码的内存缓冲区时。在正常情况下，这种错误会导致访问冲突并终止 JVM。

在某种程度上，JNA 支持一种允许 Java 代码处理访问冲突错误的方法。有两种方法可以激活它:

*   将`jna.protected`系统属性设置为`true`
*   呼叫`Native.setProtected(true)`

一旦我们激活了这个保护模式，JNA 将捕获通常会导致崩溃的访问冲突错误，并抛出一个`java.lang.Error` 异常。我们可以使用一个用无效地址初始化的`Pointer`并尝试向其写入一些数据来验证这一点:

```java
Native.setProtected(true);
Pointer p = new Pointer(0l);
try {
    p.setMemory(0, 100*1024, (byte) 0);
}
catch (Error err) {
    // ... error handling omitted
} 
```

然而，正如文档所述，该功能仅用于调试/开发目的。

## 5.结论

在本文中，我们展示了与 JNI 相比，如何使用 JNA 轻松访问本机代码。

像往常一样，GitHub 上的所有代码[都是可用的。](https://web.archive.org/web/20221101060427/https://github.com/eugenp/tutorials/tree/master/java-native)