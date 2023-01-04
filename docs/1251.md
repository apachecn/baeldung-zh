# Java 数组的最大大小

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-arrays-max-size>

## 1.概观

在本教程中，我们将看看 Java 中数组的最大大小。

## 2.最大尺寸

一个 Java 程序只能分配一定大小的数组。这通常取决于我们使用的 JVM 和平台。因为数组的索引是`int, the` **，所以近似的索引值可以是 2^31–1。基于这种近似，我们可以说该数组理论上可以容纳 2147483647 个元素**。

对于我们的例子，**我们在 Linux 和 Mac 机器上使用了 Java 8 和 Java 15 的 [OpenJDK](https://web.archive.org/web/20221126230756/https://openjdk.java.net/) 和 [Oracle](https://web.archive.org/web/20221126230756/https://www.oracle.com/in/java/technologies/javase-downloads.html) 实现。在整个测试过程中，结果都是一样的。**

这可以用一个简单的例子来验证:

```
for (int i = 2; i >= 0; i--) {
    try {
        int[] arr = new int[Integer.MAX_VALUE - i];
        System.out.println("Max-Size : " + arr.length);
    } catch (Throwable t) {
        t.printStackTrace();
    }
}
```

在使用 Linux 和 Mac 机器执行上述程序的过程中，可以观察到类似的行为。在使用 **VM 参数`-Xms2G -Xmx2G,`** 执行时，我们会收到以下错误:

```
java.lang.OutOfMemoryError: Java heap space
	at com.example.demo.ArraySizeCheck.main(ArraySizeCheck.java:8)
java.lang.OutOfMemoryError: Requested array size exceeds VM limit
	at com.example.demo.ArraySizeCheck.main(ArraySizeCheck.java:8)
java.lang.OutOfMemoryError: Requested array size exceeds VM limit 
```

请注意，第一个错误与后两个不同。最后的 **两个错误提到了 VM 限制，而第一个错误是关于堆内存限制**。

现在让我们尝试使用 **VM 参数** `**-Xms9G -Xmx9G**` 来获得准确的最大大小:

```
Max-Size: 2147483645
java.lang.OutOfMemoryError: Requested array size exceeds VM limit
	at com.example.demo.ArraySizeCheck.main(ArraySizeCheck.java:8)
java.lang.OutOfMemoryError: Requested array size exceeds VM limit
	at com.example.demo.ArraySizeCheck.main(ArraySizeCheck.java:8) 
```

结果显示**最大尺寸为 2147483645**。

对于数组中的`byte`、`boolean`、`long`和其他数据类型，可以观察到相同的行为，并且结果是相同的。

## 3.`ArraySupport`

`[ArraysSupport](https://web.archive.org/web/20221126230756/https://github.com/openjdk/jdk14u/blob/84917a040a81af2863fddc6eace3dda3e31bf4b5/src/java.base/share/classes/jdk/internal/util/ArraysSupport.java#L577)` 是 OpenJDK 中的一个实用程序类，它建议使用**最大大小**a**s`Integer.MAX_VALUE`–8**来使其与**所有的 JDK 版本和实现**一起工作。

## 4.结论

在本文中，我们研究了 Java 中数组的最大大小。

像往常一样，本教程中使用的所有代码示例都可以在 GitHub 上获得。