# 提前编译

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ahead-of-time-compilation>

## 1.介绍

在本文中，我们将提前了解 Java(AOT)编译器，它在 [JEP-295](https://web.archive.org/web/20221121102018/https://openjdk.java.net/jeps/295) 中有所描述，是作为 Java 9 中的一个实验特性添加的。

首先，我们来看看 AOT 是什么，其次，我们来看一个简单的例子。第三，我们将看到 AOT 的一些限制，最后，我们将讨论一些可能的用例。

## 2.什么是提前编译？

AOT 编译是提高 Java 程序性能的一种方式，特别是 JVM 的启动时间。JVM 执行 Java 字节码，并将频繁执行的代码编译成本机代码。这被称为实时(JIT)编译。JVM 根据执行期间收集的分析信息决定 JIT 编译哪些代码。

虽然这种技术使 JVM 能够产生高度优化的代码并提高峰值性能，但启动时间可能不是最佳的，因为执行的代码还没有经过 JIT 编译。AOT 的目标是改善这个所谓的热身期。用于 AOT 的编译器是 Graal。

在本文中，我们不会详细讨论 JIT 和 Graal。请参考我们的其他文章，了解 Java 9 和 10 中的[性能改进，以及对 Graal JIT 编译器](/web/20221121102018/https://www.baeldung.com/java-10-performance-improvements)的[深入探究。](/web/20221121102018/https://www.baeldung.com/graal-java-jit-compiler)

## 3.例子

对于这个例子，我们将使用一个非常简单的类，编译它，然后看看如何使用生成的库。

### 3.1.AOT 汇编

让我们快速看一下我们的示例类:

```java
public class JaotCompilation {

    public static void main(String[] argv) {
        System.out.println(message());
    }

    public static String message() {
        return "The JAOT compiler says 'Hello'";
    }
} 
```

在使用 AOT 编译器之前，我们需要用 Java 编译器编译这个类:

```java
javac JaotCompilation.java 
```

然后，我们将结果`JaotCompilation.class`传递给 AOT 编译器，它与标准 Java 编译器位于同一个目录中:

```java
jaotc --output jaotCompilation.so JaotCompilation.class 
```

这将在当前目录中生成库`jaotCompilation.so`。

### 3.2.运行程序

然后我们可以执行程序:

```java
java -XX:AOTLibrary=./jaotCompilation.so JaotCompilation 
```

参数`-XX:AOTLibrary`接受库的相对或完整路径。或者，我们可以将库复制到 Java 主目录中的`lib`文件夹，并且只传递库的名称。

### 3.3.验证库是否被调用和使用

我们可以看到，通过添加`-XX:+PrintAOT`作为 JVM 参数，库确实被加载了:

```java
java -XX:+PrintAOT -XX:AOTLibrary=./jaotCompilation.so JaotCompilation 
```

输出将类似于:

```java
77    1     loaded    ./jaotCompilation.so  aot library 
```

然而，这只是告诉我们库被加载了，而不是它被实际使用了。通过传递参数`-verbose`，我们可以看到库中的方法确实被调用了:

```java
java -XX:AOTLibrary=./jaotCompilation.so -verbose -XX:+PrintAOT JaotCompilation 
```

输出将包含以下行:

```java
11    1     loaded    ./jaotCompilation.so  aot library
116    1     aot[ 1]   jaotc.JaotCompilation.<init>()V
116    2     aot[ 1]   jaotc.JaotCompilation.message()Ljava/lang/String;
116    3     aot[ 1]   jaotc.JaotCompilation.main([Ljava/lang/String;)V
The JAOT compiler says 'Hello' 
```

**AOT 编译库包含一个`class fingerprint`，它必须匹配`.class`文件的指纹。**

让我们更改类`JaotCompilation.java`中的代码以返回不同的消息:

```java
public static String message() {
    return "The JAOT compiler says 'Good morning'";
} 
```

如果我们在 AOT 没有编译修改后的类的情况下执行程序:

```java
java -XX:AOTLibrary=./jaotCompilation.so -verbose -XX:+PrintAOT JaotCompilation 
```

那么输出将只包含:

```java
 11 1 loaded ./jaotCompilation.so aot library
The JAOT compiler says 'Good morning'
```

我们可以看到库中的方法不会被调用，因为类的字节码已经改变了。这背后的想法是程序将总是产生相同的结果，不管 AOT 编译的库是否被加载。

## 4.更多关于 AOT 和 JVM 的争论

### 4.1.Java 模块的 AOT 编译

也有可能 AOT 编译一个模块:

```java
jaotc --output javaBase.so --module java.base 
```

生成的库`javaBase.so`大小约为 320 MB，加载需要一些时间。可以通过选择要进行 AOT 编译的包和类来减小大小。

我们将在下面看看如何做到这一点，但是，我们不会深入到所有的细节。

### 4.2.使用编译命令进行选择性编译

为了防止 Java 模块的 AOT 编译库变得太大，我们可以添加编译命令来限制 AOT 编译的范围。这些命令需要在一个文本文件中——在我们的例子中，我们将使用文件`complileCommands.txt`:

```java
compileOnly java.lang.*
```

然后，我们将它添加到编译命令中:

```java
jaotc --output javaBaseLang.so --module java.base --compile-commands compileCommands.txt 
```

最终的库将只包含`package java.lang`中 AOT 编译的类。

为了获得真正的性能提升，我们需要找出在 JVM 预热期间调用了哪些类。

这可以通过添加几个 JVM 参数来实现:

```java
java -XX:+UnlockDiagnosticVMOptions -XX:+LogTouchedMethods -XX:+PrintTouchedMethodsAtExit JaotCompilation 
```

在本文中，我们不会深入研究这种技术。

### 4.3.AOT 编译单个类

我们可以用参数`–class-name`编译一个类:

```java
jaotc --output javaBaseString.so --class-name java.lang.String 
```

最终的库将只包含类`String`。

### 4.4.为分层编译

默认情况下，将始终使用 AOT 编译的代码，并且不会对库中包含的类进行 JIT 编译。**如果我们想在库中包含分析信息，我们可以添加参数`compile-for-tiered` :**

```java
jaotc --output jaotCompilation.so --compile-for-tiered JaotCompilation.class 
```

将使用库中预编译的代码，直到字节码适合 JIT 编译。

## 5.AOT 汇编的可能用例

AOT 的一个用例是短期运行程序，它在任何 JIT 编译发生之前完成执行。

另一个用例是嵌入式环境，在那里 JIT 是不可能的。

在这一点上，我们还需要注意，AOT 编译库只能从具有相同字节码的 Java 类加载，因此它不能通过 JNI 加载。

## 6.AOT 和亚马逊拉姆达

AOT 编译的代码的一个可能的用例是短暂的 lambda 函数，其中短的启动时间很重要。在这一节中，我们将看看如何在 AWS Lambda 上运行 AOT 编译的 Java 代码。

**在 AWS Lambda 上使用 AOT 编译需要在与 AWS 上使用的操作系统兼容的操作系统上构建库。**在写作的时候，这是`Amazon Linux 2`。

再者，Java 版本需要匹配。AWS 提供了`Amazon Corretto Java 11 JVM`。为了有一个环境来编译我们的库，我们将在 Docker 中安装`Amazon Linux 2`和`Amazon Corretto`。

我们不会讨论使用 Docker 和 AWS Lambda 的所有细节，而只是概述最重要的步骤。关于如何使用 Docker 的更多信息，请参考其官方文档[这里](https://web.archive.org/web/20221121102018/https://www.docker.com/get-started)。

关于用 Java 创建 Lambda 函数的更多细节，可以看看我们的文章 [AWS Lambda With Java](/web/20221121102018/https://www.baeldung.com/java-aws-lambda) 。

### 6.1.我们开发环境的配置

首先，我们需要获取`Amazon Linux 2`的 Docker 映像并安装`Amazon Corretto`:

```java
# download Amazon Linux 
docker pull amazonlinux 

# inside the Docker container, install Amazon Corretto
yum install java-11-amazon-corretto

# some additional libraries needed for jaotc
yum install binutils.x86_64 
```

### 6.2.编译类和库

在 Docker 容器中，我们执行以下命令:

```java
# create folder aot
mkdir aot
cd aot
mkdir jaotc
cd jaotc
```

文件夹的名称只是一个例子，当然可以是任何其他名称。

```java
package jaotc;

public class JaotCompilation {
    public static int message(int input) {
        return input * 2;
    }
}
```

下一步是编译类和库:

```java
javac JaotCompilation.java
cd ..
jaotc -J-XX:+UseSerialGC --output jaotCompilation.so jaotc/JaotCompilation.class
```

这里，使用与 AWS 上相同的垃圾收集器很重要。如果我们的库无法在 AWS Lambda 上加载，我们可能需要使用以下命令来检查实际使用了哪个垃圾收集器:

```java
java -XX:+PrintCommandLineFlags -version
```

现在，我们可以创建一个包含我们的库和类文件的 zip 文件:

```java
zip -r jaot.zip jaotCompilation.so jaotc/
```

### 6.3.配置 AWS Lambda

最后一步是登录 AWS Lamda 控制台，上传 zip 文件，并使用以下参数配置出 Lambda:

*   运行时:`Java 11`
*   经手人:`jaotc.JaotCompilation::message`

此外，我们需要创建一个名为 JAVA_TOOL_OPTIONS 的环境变量，并将其值设置为:

```java
-XX:+UnlockExperimentalVMOptions -XX:+PrintAOT -XX:AOTLibrary=./jaotCompilation.so
```

这个变量让我们将参数传递给 JVM。

最后一步是为我们的 Lambda 配置输入。默认值是一个 JSON 输入，它不能传递给我们的函数，因此我们需要将它设置为一个包含整数的字符串，例如“1”。

最后，我们可以执行我们的 Lambda 函数，并且应该在日志中看到我们的 AOT 编译库被加载了:

```java
57    1     loaded    ./jaotCompilation.so  aot library
```

## 7.结论

在本文中，我们看到了如何 AOT 编译 Java 类和模块。由于这仍然是一个实验性的特性，AOT 编译器并不是所有发行版的一部分。真实的例子仍然很少，找到应用 AOT 的最佳用例将取决于 Java 社区。

本文中的所有代码片段都可以在我们的 [GitHub 资源库](https://web.archive.org/web/20221121102018/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-9-new-features)中找到。