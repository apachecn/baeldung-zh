# Spring Boot 的默认内存设置是什么？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-default-memory-settings>

## 1.概观

在本教程中，我们将学习**Spring Boot 应用程序使用的默认内存设置。**

通常，Spring 没有任何特定于内存的配置，它使用底层 Java 进程的配置运行。下面是我们配置 Java 应用程序内存的方法。

## 2.内存设置

Java 进程或 JVM 的内存在[堆、栈](/web/20221128054842/https://www.baeldung.com/java-stack-heap)、[元空间](https://web.archive.org/web/20221128054842/https://matthung0807.blogspot.com/2019/03/about-g1-garbage-collector-permanent.html)、 [JIT](/web/20221128054842/https://www.baeldung.com/graal-java-jit-compiler) 代码缓存和共享库之间划分。

### 2.1.许多

堆是内存中对象在被垃圾收集器收集之前所在的部分。

最小堆的默认值是 **8 Mb，即 8 Mb 到 1 Gb 范围内物理内存的 1/64**。

对于大于 192 MB 的物理内存，最大堆的默认值为物理内存的 1/4，否则为物理内存的 1/2。

在堆里面，我们有一个托儿所的大小限制，当超过这个限制时，就会导致新一代的垃圾收集运行。**它的默认值是特定于平台的**。

我们也有保留区域限制。它是总堆大小的百分比，当达到该百分比时，足够长寿命的对象将从年轻一代提升到老一代。**其默认值为 25%。**

从 Java 8 开始，我们也将元空间作为存储所有类元数据的堆的一部分。默认情况下，其最小值**取决于平台，最大值不受限制**。

要覆盖最小堆、最大堆和元空间大小的默认值，请参考这篇关于配置堆大小的[文章。](/web/20221128054842/https://www.baeldung.com/spring-boot-heap-size)

我们可以使用`-Xns`参数覆盖 nursery 大小限制。因为 nursery 是堆的一部分，所以它的值不应该大于`-Xmx`值:

```java
java -Xns:10m MyApplication
```

我们还可以使用–`XXkeepAreaRatio`参数覆盖保留区域限制的默认值。例如，我们可以将其设置为 10 %:

```java
java -XXkeepAreaRatio:10 MyApplication
```

最后，下面是我们如何在 Linux 上检查堆大小:

```java
java -XX:+PrintFlagsFinal -version | grep HeapSize
```

在 Windows 上检查堆大小的相同命令将是:

```java
java -XX:+PrintFlagsFinal -version | findstr HeapSize
```

### 2.2.堆

它是提供给每个线程执行的内存量。它的**默认值是特定于平台的**。

因此，我们可以使用`-Xss`参数来定义线程堆栈的大小。例如，我们可以将其分配到 512 kB:

```java
java -Xss:512k MyApplication
```

然后，我们可以检查 Linux 上的线程堆栈大小:

```java
java -XX:+PrintFlagsFinal -version | grep ThreadStackSize
```

或者在 Windows 机器上做同样的事情:

```java
java -XX:+PrintFlagsFinal -version | findstr ThreadStackSize
```

## 3.结论

在本文中，我们已经了解了 Java 应用程序可用的各种堆和堆栈内存配置选项的默认值。

因此，在启动我们的 Spring Boot 应用程序时，我们可以根据我们的要求定义这些参数。

对于进一步的调整选项，我们确实参考了官方指南。关于所有配置参数的列表，请参考本[文件](https://web.archive.org/web/20221128054842/https://docs.oracle.com/cd/E13150_01/jrockit_jvm/jrockit/jrdocs/refman/optionX.html#wp999527)。