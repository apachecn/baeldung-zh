# Java 14 中的外部内存访问 API

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-foreign-memory-access>

## 1.概观

Java 对象驻留在堆上。然而，这偶尔会导致一些问题，如**低效的内存使用、低性能和垃圾收集问题**。在这些情况下，本机内存可能更有效，但是使用它传统上非常困难并且容易出错。

Java 14 引入了外来内存访问 API **来更安全高效地访问原生内存。**

在本教程中，我们将看看这个 API。

## 2.动机

高效使用内存一直是一项具有挑战性的任务。这主要是由于诸如对存储器、其组织和复杂的存储器寻址技术的理解不足的因素。

例如，未正确实现的内存缓存可能会导致频繁的垃圾收集。这将大大降低应用程序的性能。

在 Java 中引入外来内存访问 API 之前，有两种主要的方法来访问 Java 中的本地内存。这些是 **`java.nio.ByteBuffer`** 和 **`sun.misc.Unsafe`** 类。

让我们快速看一下这些 API 的优缺点。

### 2.1\. `ByteBuffer` API

API **允许创建直接的堆外字节缓冲区**。这些缓冲区可以从 Java 程序中直接访问。但是，有一些限制:

*   缓冲区大小不能超过 2gb
*   垃圾收集器负责内存释放

此外，不正确地使用`ByteBuffer`会导致内存泄漏和`OutOfMemory`错误。这是因为未使用的内存引用会阻止垃圾收集器释放内存。

### 2.2\. `Unsafe` API

由于其寻址模型， [`Unsafe` API](/web/20221124161003/https://www.baeldung.com/java-unsafe) 效率极高。然而，顾名思义，这个 API 是不安全的，并且有几个缺点:

*   它经常**允许 Java 程序由于非法内存使用而使 JVM 崩溃**
*   这是非标准的 Java API

### 2.3.对新 API 的需求

总之，获取一段外国记忆给我们带来了一个难题。我们是否应该使用安全但有限的路径(`ByteBuffer`)？或者我们应该冒险使用不受支持且危险的`Unsafe` API 吗？

新的外部存储器访问 API 旨在解决这些问题。

## 3.外部存储器 API

外部内存访问 API 提供了一个受支持的、安全的、高效的 API 来访问堆内存和本机内存。它基于三个主要的抽象概念:

*   *内存段*–模拟一个连续的内存区域
*   *内存地址*–内存段中的一个位置
*   *memory layout*–一种以语言中立的方式定义内存段布局的方法

下面详细讨论这些。

### 3.1.`MemorySegment`

内存段**是一个连续的内存区域。**这可以是堆内或堆外内存。而且，有几种方法可以获得内存段。

由本地内存支持的内存段被称为`native memory segment.`，它是使用重载的`allocateNative`方法之一创建的。

让我们创建一个 200 字节的本机内存段:

```java
MemorySegment memorySegment = MemorySegment.allocateNative(200);
```

内存段也可以由现有的堆分配的 Java 数组支持。例如，我们可以从一个数组`long`中创建一个`array memory segment` :

```java
MemorySegment memorySegment = MemorySegment.ofArray(new long[100]);
```

此外，内存段可以由现有的 Java `ByteBuffer`支持。这就是所谓的`buffer memory segment`:

```java
MemorySegment memorySegment = MemorySegment.ofByteBuffer(ByteBuffer.allocateDirect(200));
```

或者，我们可以使用内存映射文件。这就是所谓的`mapped memory segment.` 让我们定义一个 200 字节的内存段，使用一个具有读写权限的文件路径:

```java
MemorySegment memorySegment = MemorySegment.mapFromPath(
  Path.of("/tmp/memory.txt"), 200, FileChannel.MapMode.READ_WRITE);
```

一个内存段**被附加到一个特定的线程**。因此，如果任何其他线程需要访问内存段，它必须使用`acquire`方法获得访问权。

此外，就存储器访问而言，存储器段具有`spatial `和*时间*边界:

*   **空间边界** —内存段有下限和上限
*   **时间边界** —管理内存段的创建、使用和关闭

空间和时间检查共同确保了 JVM 的安全性。

### 3.2.`MemoryAddress`

**A `MemoryAddress`是内存段**内的偏移量。通常使用`baseAddress`方法获得:

```java
MemoryAddress address = MemorySegment.allocateNative(100).baseAddress();
```

内存地址用于执行操作，例如从底层内存段上的内存中检索数据。

### 3.3.`MemoryLayout`

`MemoryLayout`类让我们**描述内存段的内容。**具体来说，它让我们定义如何将内存分解成元素，其中提供了每个元素的大小。

这有点像将内存布局描述为具体的类型，但是没有提供 Java 类。这类似于 C++这样的语言如何将它们的结构映射到内存。

让我们以用坐标`x`和`y`定义的笛卡尔坐标点为例:

```java
int numberOfPoints = 10;
MemoryLayout pointLayout = MemoryLayout.ofStruct(
  MemoryLayout.ofValueBits(32, ByteOrder.BIG_ENDIAN).withName("x"),
  MemoryLayout.ofValueBits(32, ByteOrder.BIG_ENDIAN).withName("y")
);
SequenceLayout pointsLayout = 
  MemoryLayout.ofSequence(numberOfPoints, pointLayout);
```

这里，我们定义了一个由两个名为`x`和`y`的 32 位值组成的布局。这个布局可以和一个`SequenceLayout`一起使用，产生一个类似于数组的东西，在这个例子中有 10 个索引。

## 4.使用本机内存

### 4.1.`MemoryHandles`

`MemoryHandles` 类让我们构造 VarHandles。**一个`VarHandle`允许访问一个内存段。**

让我们试试这个:

```java
long value = 10;
MemoryAddress memoryAddress = MemorySegment.allocateNative(8).baseAddress();
VarHandle varHandle = MemoryHandles.varHandle(long.class, ByteOrder.nativeOrder());
varHandle.set(memoryAddress, value);

assertThat(varHandle.get(memoryAddress), is(value));
```

在上面的例子中，我们创建了一个 8 字节的`MemorySegment`。我们需要 8 个字节来表示内存中的一个`long`数。然后，我们使用一个`VarHandle`来存储和检索它。

### 4.2.使用带偏移的`MemoryHandles`

我们也可以结合使用偏移量和`MemoryAddress`来访问内存段。这类似于使用索引从数组中获取项目:

```java
VarHandle varHandle = MemoryHandles.varHandle(int.class, ByteOrder.nativeOrder());
try (MemorySegment memorySegment = MemorySegment.allocateNative(100)) {
    MemoryAddress base = memorySegment.baseAddress();
    for(int i=0; i<25; i++) {
        varHandle.set(base.addOffset((i*4)), i);
    }
    for(int i=0; i<25; i++) {
        assertThat(varHandle.get(base.addOffset((i*4))), is(i));
    }
}
```

在上面的例子中，我们将整数 0 到 24 存储在一个内存段中。

首先，我们创建一个 100 字节的`MemorySegment` 。这是因为，在 Java 中，每个整数消耗 4 个字节。因此，要存储 25 个整数值，我们需要 100 个字节(4*25)。

为了访问每个索引，我们使用基址上的`addOffset`设置`varHandle`指向右边的偏移量。

### 4.3.`MemoryLayouts`

**`MemoryLayouts`类定义了各种有用的布局常量**。

例如，在前面的例子中，我们创建了一个`SequenceLayout`:

```java
SequenceLayout sequenceLayout = MemoryLayout.ofSequence(25, 
  MemoryLayout.ofValueBits(64, ByteOrder.nativeOrder()));
```

这可以用`JAVA_LONG`常量更简单地表达:

```java
SequenceLayout sequenceLayout = MemoryLayout.ofSequence(25, MemoryLayouts.JAVA_LONG);
```

### 4.4.`ValueLayout`

**A `ValueLayout`为基本数据类型(如整型和浮点型)建模内存布局。**每个值布局都有大小和字节顺序。我们可以使用`ofValueBits` 方法创建一个`ValueLayout`:

```java
ValueLayout valueLayout = MemoryLayout.ofValueBits(32, ByteOrder.nativeOrder()); 
```

### 4.5.`SequenceLayout`

**A `SequenceLayout` 表示给定布局的重复。**换句话说，这可以被认为是一个元素序列，类似于一个具有已定义元素布局的数组。

例如，我们可以为每个 64 位的 25 个元素创建一个序列布局:

```java
SequenceLayout sequenceLayout = MemoryLayout.ofSequence(25, 
  MemoryLayout.ofValueBits(64, ByteOrder.nativeOrder())); 
```

### 4.6.`GroupLayout`

**一个`GroupLayout` 可以组合多个成员布局**。成员布局可以是相似的类型，也可以是不同类型的组合。

有两种可能的方法来定义组布局。例如，当一个接一个地组织成员布局时，它被定义为`struct.`。另一方面，如果成员布局从相同的起始偏移开始布局，那么它被称为`union`。

让我们用一个`integer`和一个`long`创建一个`struct` 类型的`GroupLayout` :

```java
GroupLayout groupLayout = MemoryLayout.ofStruct(MemoryLayouts.JAVA_INT, MemoryLayouts.JAVA_LONG);
```

我们也可以使用`ofUnion` 方法创建一个`union` 类型的`GroupLayout` :

```java
GroupLayout groupLayout = MemoryLayout.ofUnion(MemoryLayouts.JAVA_INT, MemoryLayouts.JAVA_LONG);
```

第一个是包含每种类型的结构。第二个是可以包含这两种类型的结构。

组布局允许我们创建由多个元素组成的复杂内存布局。例如:

```java
MemoryLayout memoryLayout1 = MemoryLayout.ofValueBits(32, ByteOrder.nativeOrder());
MemoryLayout memoryLayout2 = MemoryLayout.ofStruct(MemoryLayouts.JAVA_LONG, MemoryLayouts.PAD_64);
MemoryLayout.ofStruct(memoryLayout1, memoryLayout2);
```

## 5.分割内存段

我们可以将一个内存段分割成多个更小的块。如果我们想用不同的布局存储值，这就避免了我们必须分配多个块。

让我们试试使用`asSlice`:

```java
MemoryAddress memoryAddress = MemorySegment.allocateNative(12).baseAddress();
MemoryAddress memoryAddress1 = memoryAddress.segment().asSlice(0,4).baseAddress();
MemoryAddress memoryAddress2 = memoryAddress.segment().asSlice(4,4).baseAddress();
MemoryAddress memoryAddress3 = memoryAddress.segment().asSlice(8,4).baseAddress();

VarHandle intHandle = MemoryHandles.varHandle(int.class, ByteOrder.nativeOrder());
intHandle.set(memoryAddress1, Integer.MIN_VALUE);
intHandle.set(memoryAddress2, 0);
intHandle.set(memoryAddress3, Integer.MAX_VALUE);

assertThat(intHandle.get(memoryAddress1), is(Integer.MIN_VALUE));
assertThat(intHandle.get(memoryAddress2), is(0));
assertThat(intHandle.get(memoryAddress3), is(Integer.MAX_VALUE));
```

## 6.结论

在本文中，我们了解了 Java 14 中新的外部内存访问 API。

首先，我们研究了对外部内存访问的需求以及 Java 14 之前的 API 的局限性。然后，我们看到了外部内存访问 API 是如何安全地访问堆内存和非堆内存的。

最后，我们探索了使用 API 在堆上和堆外读写数据。

和往常一样，GitHub 上的[提供了示例的源代码。](https://web.archive.org/web/20221124161003/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-14)