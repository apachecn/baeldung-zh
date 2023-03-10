# 找到 JVM 实例使用的 GC 算法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jvm-find-gc-algorithm>

## 1.概观

除了编译器和运行时等典型的开发工具之外，每个 JDK 版本都附带了大量的其他工具。其中一些工具可以帮助我们深入了解正在运行的应用程序。

在本文中，我们将看到如何使用这样的工具来找到关于特定 JVM 实例使用的 [GC 算法](/web/20220523152729/https://www.baeldung.com/jvm-garbage-collectors)的更多信息。

## 2.示例应用程序

在本文中，我们将使用一个非常简单的应用程序:

```java
public class App {
    public static void main(String[] args) throws IOException {
        System.out.println("Waiting for stdin");
        int read = System.in.read();
        System.out.println("I'm done: " + read);
    }
}
```

显然，这个应用程序等待并保持运行，直到它从标准输入接收到一些东西。这种挂起有助于我们模仿长时间运行的 JVM 应用程序的行为。

为了使用这个应用程序，我们必须用`javac `编译`App.java `文件，然后使用`java `工具运行它。

## 3.寻找 JVM 进程

为了找到 JVM 进程使用的 GC，首先，我们应该识别特定 JVM 实例的进程 id。假设我们用以下命令运行了我们的应用程序:

```java
>> java App
Waiting for stdin
```

如果我们安装了 JDK，找到 JVM 实例的进程 id 的最好方法是使用 [`jps`](https://web.archive.org/web/20220523152729/https://docs.oracle.com/en/java/javase/11/tools/jps.html) 工具。例如:

```java
>> jps -l
69569 
48347 App
48351 jdk.jcmd/sun.tools.jps.Jps 
```

如上所示，系统上运行着三个 JVM 实例。显然，第二个 JVM 实例(“App”)的描述与我们的应用程序名称相匹配。因此，我们要寻找的进程 id 是 48347。

除了`jps`之外，我们总是可以使用其他通用实用程序来过滤掉正在运行的进程。例如， [procps](https://web.archive.org/web/20220523152729/https://gitlab.com/procps-ng/procps) 包中著名的`[ps](/web/20220523152729/https://www.baeldung.com/linux/ps-command) `工具也可以工作:

```java
>> ps -ef | grep java
502 48347 36213   0  1:28AM ttys037    0:00.28 java App
```

然而，`jps `使用起来更简单，需要的过滤也更少。

## 4.用过的 GC

现在我们知道了如何找到进程 id，让我们找到已经运行的 JVM 应用程序所使用的 GC 算法。

### 4.1.Java 8 和更早版本

**如果我们在 Java 8 上，我们可以使用`[jmap](https://web.archive.org/web/20220523152729/https://docs.oracle.com/en/java/javase/11/tools/jmap.html) `实用程序来打印堆摘要、堆直方图，甚至生成堆转储**。为了找到 GC 算法，我们可以使用`-heap `选项:

```java
>> jmap -heap <pid>
```

因此，在我们的特殊情况下，我们使用 CMS GC:

```java
>> jmap -heap 48347 | grep GC
Concurrent Mark-Sweep GC
```

对于其他 GC 算法，输出几乎相同:

```java
>> jmap -heap 48347 | grep GC
Parallel GC with 8 thread(s)
```

### 4.2.Java 9+: `jhsdb jmap`

**从 Java 9 开始，我们可以使用`[jhsdb jmap](https://web.archive.org/web/20220523152729/https://docs.oracle.com/en/java/javase/11/tools/jps.html) `组合来打印一些关于 JVM 堆的信息。**更具体地说，这个特定的命令相当于前一个命令:

```java
>> jhsdb jmap --heap --pid <pid>
```

例如，我们的应用现在使用 G1GC 运行:

```java
>> jhsdb jmap --heap --pid 48347 | grep GC
Garbage-First (G1) GC with 8 thread(s) 
```

### 4.3.Java 9+: `jcmd`

在现代 JVM 中，`jcmd `命令非常通用。例如，**我们可以用它来获得一些关于堆的一般信息**:

```java
>> jcmd <pid> VM.info
```

因此，如果我们传递应用程序的进程 id，我们可以看到这个 JVM 实例正在使用串行 GC:

```java
>> jcmd 48347 VM.info | grep gc
# Java VM: OpenJDK 64-Bit Server VM (15+36-1562, mixed mode, sharing, tiered, compressed oops, serial gc, bsd-amd64)
// omitted
```

G1 或 ZGC 的产出类似:

```java
// ZGC
# Java VM: OpenJDK 64-Bit Server VM (15+36-1562, mixed mode, sharing, tiered, z gc, bsd-amd64)
// G1GC
# Java VM: OpenJDK 64-Bit Server VM (15+36-1562, mixed mode, sharing, tiered, compressed oops, g1 gc, bsd-amd64) 
```

用一点点 [`grep`](/web/20220523152729/https://www.baeldung.com/linux/common-text-search) 的魔法，我们也可以去掉所有那些杂音，只得到 GC 名:

```java
>> jcmd 48347 VM.info | grep -ohE "[^\s^,]+\sgc"
g1 gc
```

### 4.4.命令行参数

有时，我们(或其他人)在启动 JVM 应用程序时明确指定 GC 算法。例如，我们在这里选择使用 ZGC:

```java
>> java -XX:+UseZGC App
```

在这种情况下，有更简单的方法来找到用过的 GC。基本上，**我们所要做的就是以某种方式找到应用程序已经用**执行的命令。

例如，在基于 UNIX 的平台上，我们可以再次使用`ps`命令:

```java
>> ps -p 48347 -o command=
java -XX:+UseZGC App
```

从上面的输出来看，很明显 JVM 正在使用 ZGC。同样，**`jcmd `命令也可以打印命令行参数**:

```java
>> jcmd 48347 VM.flags
84020:
-XX:CICompilerCount=4 -XX:-UseCompressedOops -XX:-UseNUMA -XX:-UseNUMAInterleaving -XX:+UseZGC // omitted
```

**令人惊讶的是，如上所示，该命令将打印隐式和显式参数以及可调参数**。因此，即使我们没有明确指定 GC 算法，它也会显示所选的默认算法:

```java
>> jcmd 48347 VM.flags | grep -ohE '\S*GC\s'
-XX:+UseG1GC
```

更令人惊讶的是，这也适用于 Java 8:

```java
>> jcmd 48347 VM.flags | grep -ohE '\S*GC\s'
-XX:+UseParallelGC
```

## 5.结论

在本文中，我们看到了寻找特定 JVM 实例使用的 GC 算法的不同方法。提到的一些方法依赖于特定的 Java 版本，一些是可移植的。

此外，我们还看到了几种查找进程 id 的方法，这总是需要的。