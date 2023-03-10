# 带有运行时 API 的 Java 堆空间内存

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-heap-memory-api>

## 1.概观

在本文中，我们将讨论 Java 提供的 API，这些 API 可以帮助我们理解与 [Java 堆空间](/web/20221128025946/https://www.baeldung.com/java-stack-heap)相关的几个方面。

这有助于了解 JVM 的当前内存状态，并将其外包给监控服务，如 [StatsD](https://web.archive.org/web/20221128025946/https://www.datadoghq.com/blog/statsd/) 和 [Datadog](https://web.archive.org/web/20221128025946/https://docs.datadoghq.com/developers/dogstatsd/) ，然后可以对其进行配置，以采取先发制人的行动，避免应用程序故障。

## 2.访问内存参数

每个 Java 应用程序都有一个单独的`java.lang.Runtime` 实例，可以帮助我们了解应用程序的当前内存状态。可以调用静态方法`Runtime#getRuntime` 来获得单例的`Runtime`实例。

### 2.1.总内存

`Runtime#getTotalMemory`方法返回 JVM 当前保留的总堆空间，以字节为单位。**它包括为当前和未来对象保留的内存。**因此，不能保证它在程序执行期间是恒定的，因为 Java 堆空间可以随着更多对象的分配而扩展或减少。

另外，**这个值不一定是正在使用的内存或最大可用内存。**

### 2.2.空闲存储器

`Runtime#freeMemory`方法以字节为单位返回可用于新对象分配的空闲堆空间。它可能会因垃圾收集操作而增加，在垃圾收集操作之后，会有更多的可用内存。

### 2.3.最大内存

`Runtime#maxMemory` 方法返回 JVM 试图使用的最大内存。一旦 JVM 内存使用量达到这个值，它就不会分配更多的内存，而是会更频繁地进行垃圾收集。

如果 JVM 对象甚至在垃圾收集器运行之后仍然需要更多的内存，那么 JVM 可能抛出一个运行时异常。

## 3.例子

在下面的例子中，我们初始化一个`ArrayList`并向其中添加元素，同时使用上面的三种方法跟踪 JVM 堆空间:

```java
ArrayList<Integer> arrayList = new ArrayList<>();
System.out.println("i \t Free Memory \t Total Memory \t Max Memory");
for (int i = 0; i < 1000000; i++) {
    arrayList.add(i);
    System.out.println(i + " \t " + Runtime.getRuntime().freeMemory() + 
      " \t \t " + Runtime.getRuntime().totalMemory() + 
      " \t \t " + Runtime.getRuntime().maxMemory());
}

// ...
```

```java
Output:
i 	   Free Memory 	   Total Memory 	 Max Memory
0 	     254741016 	 	 257425408 	 	 3817865216
1 	     254741016 	 	 257425408 	 	 3817865216
...
1498 	 254741016 	 	 257425408 	 	 3817865216
1499 	 253398840 	 	 257425408 	 	 3817865216
1500 	 253398840 	 	 257425408 	 	 3817865216
...
900079 	 179608120 	 	 260046848 	 	 3817865216
900080 	 302140152 	 	 324534272 	 	 3817865216
900081 	 302140152 	 	 324534272 	 	 3817865216
...
```

*   第 1498 行:当在 Java 堆中为足够多的对象分配了空间时，`Runtime#freeMemory`值会减少。
*   Row 900080:此时，JVM 有更多的可用空间，因为 GC 已经运行，因此`Runtime#freeMemory`和`Runtime#totalMemory`的值增加了。

上面显示的值在 Java 应用程序的每次运行中都应该是不同的。

## 4.定制内存参数

当运行我们的 Java 程序时，我们可以通过将自定义值设置为某些标志来覆盖 JVM 内存参数的默认值，以便获得所需的内存性能:

*   `-Xms:`分配给`-Xms`标志的值设置 Java 堆的初始值和最小值。当我们的应用程序在启动 JVM 时需要比缺省最小值更多的内存时，可以使用它
*   `-Xmx:`同样，我们可以通过将堆空间的最大值分配给`-Xmx`标志来设置它。当我们想要故意限制应用程序使用的内存量时，可以使用它。

还请注意，`-Xms`值需要等于或小于`-Xmx` 值。

### 4.1.使用

```java
java -Xms32M -Xmx64M Main                                                                                        
Free Memory   : 31792664 bytes
Total Memory  : 32505856 bytes
Max Memory    : 59768832 bytes

java -Xms64M -Xmx64M Main
Free Memory   : 63480640 bytes
Total Memory  : 64487424 bytes
Max Memory    : 64487424 bytes

java -Xms64M -Xmx32M Main                                                                                        
Error occurred during initialization of VM
Initial heap size set to a larger value than the maximum heap size
```

## 5.结论

在本文中，我们已经看到了如何通过`Runtime`类检索 JVM 内存指标。在调查 [JVM 内存泄漏](/web/20221128025946/https://www.baeldung.com/java-memory-leaks)和其他与 JVM 内存性能相关的问题时，这些方法会很有用。

我们还展示了如何为某些标志指定自定义值，从而导致不同场景下不同的 JVM 内存行为。