# 在 JVM 中配置堆栈大小

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jvm-configure-stack-sizes>

## 1.概观

在这个快速教程中，我们将看到如何在 HotSpot JVM 中配置线程堆栈大小。

## 2.默认堆栈大小

每个 JVM 线程都有一个私有的本机堆栈来存储[调用堆栈](/web/20220625083844/https://www.baeldung.com/cs/call-stack)信息、局部变量和部分结果。因此，堆栈在方法调用中起着至关重要的作用。这是 [JVM 规范](https://web.archive.org/web/20220625083844/https://docs.oracle.com/javase/specs/jvms/se14/html/jvms-2.html#jvms-2.5.2)的一部分，因此，每个 JVM 实现都使用堆栈。

然而，其他实现细节，如堆栈大小，是特定于实现的。从现在开始，我们将把重点放在 HotSpot JVM 上，并将互换使用术语 JVM 和 HotSpot JVM。

总之，**JVM 会在创建自己的线程的同时创建堆栈。**

如果我们不指定堆栈的大小，JVM 将创建一个默认大小的堆栈。通常，该默认大小取决于操作系统和计算机架构。例如，这些是从 Java 14 开始的一些默认大小:

*   [Linux/x86 (64 位)](https://web.archive.org/web/20220625083844/https://github.com/openjdk/jdk14u/blob/7a3bf58b8ad2c327229a94ae98f58ec96fa39332/src/hotspot/os_cpu/linux_x86/globals_linux_x86.hpp#L34) : 1 MB
*   [苹果操作系统(64 位)](https://web.archive.org/web/20220625083844/https://github.com/openjdk/jdk14u/blob/7a3bf58b8ad2c327229a94ae98f58ec96fa39332/src/hotspot/os_cpu/bsd_x86/globals_bsd_x86.hpp#L35) : 1 MB
*   [Oracle Solaris (64 位)](https://web.archive.org/web/20220625083844/https://github.com/openjdk/jdk14u/blob/7a3bf58b8ad2c327229a94ae98f58ec96fa39332/src/hotspot/os_cpu/solaris_x86/globals_solaris_x86.hpp#L34) : 1 MB
*   在 Windows 上，JVM 使用[系统范围的堆栈大小](https://web.archive.org/web/20220625083844/https://github.com/openjdk/jdk14u/blob/7a3bf58b8ad2c327229a94ae98f58ec96fa39332/src/hotspot/os_cpu/windows_x86/globals_windows_x86.hpp#L37)

基本上，在大多数现代操作系统和架构中，我们可以预期每个堆栈大约 1 MB。

## 3.自定义堆栈大小

**要改变堆栈大小，我们可以使用`-Xss `调** **标志。**例如，`-Xss1048576 `将堆栈大小设置为 1 MB:

```
java -Xss1048576 // omitted
```

如果我们不想以字节计算大小，我们可以使用一些方便的快捷方式来指定不同的单位——字母`k`或`K`表示 KB，`m`或`M`表示 MB，`g`或`G`表示 GB。例如，让我们看看将堆栈大小设置为 1 MB 的几种不同方法:

```
-Xss1m 
-Xss1024k
```

与`-Xss`、**类似，我们也可以使用`-XX:ThreadStackSize `调整标志来配置堆栈大小。然而，**和`-XX:ThreadStackSize`的语法有点不同。我们应该用等号分隔大小和旗帜名称:

```
java -XX:ThreadStackSize=1024 // omitted
```

HotSpot JVM [不允许我们使用小于最小值](https://web.archive.org/web/20220625083844/https://github.com/openjdk/jdk14u/blob/03db2e14dde027eb5dae1435bc9b7f95b1fb48df/src/hotspot/os/posix/os_posix.cpp#L1397)的大小:

```
$ java -Xss1K -version
The Java thread stack size specified is too small. Specify at least 144k
Error: Could not create the Java Virtual Machine.
Error: A fatal exception has occurred. Program will exit.
```

另外，[不允许我们使用超过最大值](https://web.archive.org/web/20220625083844/https://github.com/openjdk/jdk14u/blob/7a3bf58b8ad2c327229a94ae98f58ec96fa39332/src/hotspot/share/runtime/arguments.cpp#L2413)(通常为 1 GB)的大小:

```
$ java -Xss2g -version
Invalid thread stack size: -Xss2g
The specified size exceeds the maximum representable size.
Error: Could not create the Java Virtual Machine.
Error: A fatal exception has occurred. Program will exit.
```

## 4.结论

在这个快速教程中，我们看到了如何在 HotSpot JVM 中配置线程堆栈大小。