# Java 中的位集指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-bitset>

## 1.概观

在本教程中，我们将看到如何使用 [`BitSet` s](https://web.archive.org/web/20220628053548/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/BitSet.html) 来表示一个位向量。

首先，我们将从不使用`boolean[]`背后的基本原理开始。然后在熟悉了`BitSet `的内部之后，我们将仔细看看它的 API。

## 2.比特阵列

为了存储和操作位数组，有人可能会说我们应该使用`boolean[] `作为我们的数据结构。乍一看，这似乎是一个合理的建议。

**然而，`boolean[] `中的每个`boolean `成员通常[消耗一个字节而不是仅仅一个比特](/web/20220628053548/https://www.baeldung.com/jvm-boolean-memory-layout)** 。因此，当我们有严格的内存需求，或者我们只是为了减少内存占用，`boolean[] `远非理想。

更具体地说，让我们看看一个有 1024 个元素的`boolean[] `消耗了多少空间:

```java
boolean[] bits = new boolean[1024];
System.out.println(ClassLayout.parseInstance(bits).toPrintable());
```

理想情况下，我们期望该阵列占用 1024 位内存。然而，[Java 对象布局(JOL)](https://web.archive.org/web/20220628053548/https://search.maven.org/artifact/org.openjdk.jol/jol-core) 揭示了一个完全不同的现实:

```java
[Z object internals:
 OFFSET  SIZE      TYPE DESCRIPTION            VALUE
      0     4           (object header)        01 00 00 00 (00000001 00000000 00000000 00000000) (1)
      4     4           (object header)        00 00 00 00 (00000000 00000000 00000000 00000000) (0)
      8     4           (object header)        7b 12 07 00 (01111011 00010010 00000111 00000000) (463483)
     12     4           (object header)        00 04 00 00 (00000000 00000100 00000000 00000000) (1024)
     16  1024   boolean [Z.                    N/A
Instance size: 1040 bytes
```

如果我们忽略对象头的开销，数组元素将消耗 1024 字节，而不是预期的 1024 位。这比我们预期的内存多了 700%。

****[寻址能力问题和字撕裂](/web/20220628053548/https://www.baeldung.com/jvm-boolean-memory-layout#2-word-tearing)是*布尔*不仅仅是一个单一位的主要原因。****

 **为了解决这个问题，我们可以使用数字数据类型(比如`long`)和逐位操作的组合。这就是`BitSet `的用武之地。

## 3.`BitSet `如何工作

正如我们前面提到的，为了实现每个标志一位的内存使用量，`BitSet ` API 使用了基本数字数据类型和逐位操作的组合。

为了简单起见，假设我们要用一个`byte`来表示八面旗帜。首先，我们用零初始化这个单个`byte`的所有位:

[![initial-bytes](img/7516ad92e9b81e7ba16e53ce1ed79c4c.png)](/web/20220628053548/https://www.baeldung.com/wp-content/uploads/2020/07/initial-bytes.png)

现在，如果我们想将位置 3 的位设置为`true`，**，我们应该首先将数字 1 左移 3:**

[![left-shift](img/3bb7e9aa409b9f0683451465ee275b13.png)](/web/20220628053548/https://www.baeldung.com/wp-content/uploads/2020/07/left-shift.png)

**然后`or `其结果与当前`byte`值**:

[![final-or](img/f0dd81a5f1ea6c100aeb9660df1c8f47.png)](/web/20220628053548/https://www.baeldung.com/wp-content/uploads/2020/07/final-or.png)

如果决定在索引 7 处设置该位，将发生相同的过程:

[![another-set](img/c31a90e82d78c47410961d0f86c80e46.png)](/web/20220628053548/https://www.baeldung.com/wp-content/uploads/2020/07/another-set.png)

如上所示，我们执行了 7 位的左移，并使用`or `操作符将结果与之前的`byte`值相结合。

### 3.1.获取位索引

**为了检查特定的位索引是否设置为`true `，我们将使用`and `操作符**。例如，下面是我们如何检查索引三是否已设置:

1.  对值 1 执行三位左移
2.  `Anding `当前`byte`值的结果
3.  如果结果大于零，那么我们找到了匹配，并且该位索引实际上被设置。否则，所请求的索引是清晰的或者等于`false`

[![get-set](img/847277b388478553deb3633af99beae9.png)](/web/20220628053548/https://www.baeldung.com/wp-content/uploads/2020/07/get-set.png)

上图显示了索引三的 get 操作步骤。然而，如果我们查询一个明确的索引，结果就会不同:

[![get-clear](img/490e518826337d85dd30a05634c64b84.png)](/web/20220628053548/https://www.baeldung.com/wp-content/uploads/2020/07/get-clear.png)

此后

### 3.2.增加存储

目前，我们只能存储 8 位的向量。为了超越这个限制，**我们只需要使用一个`byte`的数组，而不是一个`byte`，就是这样！**

现在，每当我们需要设置、获取或清除一个特定的索引时，我们应该首先找到相应的数组元素。例如，假设我们要设置索引 14:

[![array-set](img/75a38a78a594af5e5e6ea67fcfcf33e4.png)](/web/20220628053548/https://www.baeldung.com/wp-content/uploads/2020/07/array-set-1.png)

如上图所示，在找到正确的数组元素后，我们确实设置了适当的索引。

同样，如果我们想在这里设置一个超过 15 的索引，`BitSet `将首先扩展它的内部数组。只有在扩展数组并复制元素后，它才会设置所请求的位。这有点类似于`[ArrayList](/web/20220628053548/https://www.baeldung.com/java-arraylist-linkedlist) `的内部工作方式。

到目前为止，为了简单起见，我们使用了`byte `数据类型。**然而，`BitSet ` API 在内部**使用了一组`long `值。

## 4.`BitSet ` API

既然我们已经对理论有了足够的了解，是时候看看`BitSet ` API 是什么样子了。

首先，让我们比较一下 1024 位的`BitSet `实例和我们之前看到的`boolean[] `实例的内存占用:

```java
BitSet bitSet = new BitSet(1024);

System.out.println(GraphLayout.parseInstance(bitSet).toPrintable());
```

这将打印出`BitSet `实例的浅层大小及其内部数组的大小:

```java
[[email protected]](/web/20220628053548/https://www.baeldung.com/cdn-cgi/l/email-protection) object externals:
          ADDRESS       SIZE TYPE             PATH         VALUE
        70f97d208         24 java.util.BitSet              (object)
        70f97d220        144 [J               .words       [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
```

如上图，它内部使用了一个 16 元素的`long[] `(16 * 64 位= 1024 位)。总之，**这个实例总共使用了 168 个字节，而`boolean[] `使用了 1024 个字节**。

我们拥有的位数越多，足迹差异就越大。例如，为了存储 1024 * 1024 位，`boolean[] `消耗 1 MB，而`BitSet `实例消耗大约 130 KB。

### 4.1.构造`BitSet` s

创建`BitSet `实例最简单的方法是使用[无参数构造函数](https://web.archive.org/web/20220628053548/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/BitSet.html#%3Cinit%3E()):

```java
BitSet bitSet = new BitSet();
```

**这将创建一个`BitSet `实例，其`long[] `的大小为**。当然，如果需要，它可以自动增长这个数组。

也可以创建一个初始位数为[的`BitSet `](https://web.archive.org/web/20220628053548/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/BitSet.html#%3Cinit%3E(int)):

```java
BitSet bitSet = new BitSet(100_000);
```

这里，内部数组将有足够的元素来保存 100，000 位。当我们已经对要存储的位数有了合理的估计时，这个构造函数就派上了用场。在这样的用例中，**它可以防止或减少数组元素在增长的同时不必要的复制**。

甚至可以从现有的`long[]`、`byte[]`、[、`LongBuffer`、](https://web.archive.org/web/20220628053548/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/LongBuffer.html)和[、`ByteBuffer`、](https://web.archive.org/web/20220628053548/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/ByteBuffer.html)中创建一个`BitSet `。例如，这里我们从给定的`long[]`创建一个`BitSet `实例:

```java
BitSet bitSet = BitSet.valueOf(new long[] { 42, 12 });
```

还有另外三个静态工厂方法的重载版本来支持其他提到的类型。

### 4.2.设置位

我们可以使用`[set(index)](https://web.archive.org/web/20220628053548/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/BitSet.html#set(int)) `方法将特定索引的值设置为`true `:

```java
BitSet bitSet = new BitSet();

bitSet.set(10);
assertThat(bitSet.get(10)).isTrue();
```

和往常一样，这些指数都是从零开始的。**甚至可以使用`[set(fromInclusive, toExclusive)](https://web.archive.org/web/20220628053548/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/BitSet.html#set(int,int)) `方法**将一系列位设置为`true `:

```java
bitSet.set(20, 30);
for (int i = 20; i <= 29; i++) {
    assertThat(bitSet.get(i)).isTrue();
}
assertThat(bitSet.get(30)).isFalse();
```

从方法签名可以明显看出，开始索引是包含性的，结束索引是排他性的。

我们说设置一个索引，通常是指设置为`true`。尽管有这个术语，我们可以使用`[set(index, boolean)](https://web.archive.org/web/20220628053548/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/BitSet.html#set(int,boolean)) `方法为`false `设置一个特定的位索引:

```java
bitSet.set(10, false);
assertThat(bitSet.get(10)).isFalse();
```

此版本还支持设置一系列值:

```java
bitSet.set(20, 30, false);
for (int i = 20; i <= 29; i++) {
    assertThat(bitSet.get(i)).isFalse();
}
```

### 4.3.清除位

我们可以使用`[clear(index)](https://web.archive.org/web/20220628053548/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/BitSet.html#clear(int,int)) `方法简单地清除它，而不是将特定的位索引设置为`false`:

```java
bitSet.set(42);
assertThat(bitSet.get(42)).isTrue();

bitSet.clear(42);
assertThat(bitSet.get(42)).isFalse();
```

此外，我们还可以用`[clear(fromInclusive, toExclusive)](https://web.archive.org/web/20220628053548/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/BitSet.html#clear(int,int)) `重载版本清除一系列位:

```java
bitSet.set(10, 20);
for (int i = 10; i < 20; i++) {
    assertThat(bitSet.get(i)).isTrue();
}

bitSet.clear(10, 20);
for (int i = 10; i < 20; i++) {
    assertThat(bitSet.get(i)).isFalse();
}
```

有趣的是，**如果我们在不传递任何参数的情况下调用这个方法，它将清除所有设置的位**:

```java
bitSet.set(10, 20);
bitSet.clear();
for (int i = 0; i < 100; i++) { 
    assertThat(bitSet.get(i)).isFalse();
}
```

如上图，调用`[clear()](https://web.archive.org/web/20220628053548/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/BitSet.html#clear()) `方法后，所有位都被置零。

### 4.4.获取位

到目前为止，我们相当广泛地使用了`[get(index)](https://web.archive.org/web/20220628053548/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/BitSet.html#get(int)) `方法。**当所请求的位索引被设置时，该方法将返回`true`。否则，它会返回`false`** :

```java
bitSet.set(42);

assertThat(bitSet.get(42)).isTrue();
assertThat(bitSet.get(43)).isFalse();
```

类似于`set `和`clear`，我们可以使用`[get(fromInclusive, toExclusive)](https://web.archive.org/web/20220628053548/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/BitSet.html#get(int,int)) `方法获得一个比特索引范围:

```java
bitSet.set(10, 20);
BitSet newBitSet = bitSet.get(10, 20);
for (int i = 0; i < 10; i++) {
    assertThat(newBitSet.get(i)).isTrue();
}
```

如上图，该方法返回当前方法的[20，30]范围内的另一个`BitSet`。也就是说，`bitSet `变量的索引 20 相当于`newBitSet `变量的索引 0。

### 4.5.翻转比特

**为了否定当前的位索引值，我们可以使用`[flip(index)](https://web.archive.org/web/20220628053548/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/BitSet.html#flip(int)) `方法**。也就是说，它会将`true `值转换为`false `，反之亦然:

```java
bitSet.set(42);
bitSet.flip(42);
assertThat(bitSet.get(42)).isFalse();

bitSet.flip(12);
assertThat(bitSet.get(12)).isTrue();
```

类似地，我们可以使用`[flip(fromInclusive, toExclusive)](https://web.archive.org/web/20220628053548/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/BitSet.html#flip(int,int)) `方法对一系列值实现同样的事情:

```java
bitSet.flip(30, 40);
for (int i = 30; i < 40; i++) {
    assertThat(bitSet.get(i)).isTrue();
}
```

### 4.6.长度

一个`BitSet`有三种类似长度的方法。**`[size()](https://web.archive.org/web/20220628053548/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/BitSet.html#size()) `方法返回内部数组可以表示的位数**。例如，由于无参数构造函数分配了一个只有一个元素的`long[] `数组，那么`size() `将返回 64:

```java
BitSet defaultBitSet = new BitSet();
assertThat(defaultBitSet.size()).isEqualTo(64);
```

用一个 64 位数，我们只能表示 64 位。当然，如果我们显式传递位数，这种情况将会改变:

```java
BitSet bitSet = new BitSet(1024);
assertThat(bitSet.size()).isEqualTo(1024);
```

**此外，`[cardinality()](https://web.archive.org/web/20220628053548/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/BitSet.html#cardinality()) `方法表示`BitSet`** 中的置位位数:

```java
assertThat(bitSet.cardinality()).isEqualTo(0);
bitSet.set(10, 30);
assertThat(bitSet.cardinality()).isEqualTo(30 - 10);
```

首先，该方法返回零，因为所有位都是`false`。在将[10，30]范围设置为`true`之后，`cardinality() `方法调用返回 20。

**此外，`[length()](https://web.archive.org/web/20220628053548/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/BitSet.html#length()) `方法返回最后一个设置位**的索引之后的一个索引:

```java
assertThat(bitSet.length()).isEqualTo(30);
bitSet.set(100);
assertThat(bitSet.length()).isEqualTo(101);
```

一开始，最后设置的索引是 29，所以这个方法返回 30。当我们将索引 100 设置为 true 时，`length() `方法返回 101。**值得一提的是，如果所有位都清零，该方法将返回零**。

最后，当`BitSet`中至少有一个置位位时，`[isEmpty()](https://web.archive.org/web/20220628053548/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/BitSet.html#isEmpty()) `方法返回`false `。否则，它将返回`true`:

```java
assertThat(bitSet.isEmpty()).isFalse();
bitSet.clear();
assertThat(bitSet.isEmpty()).isTrue();
```

### 4.7.与其他`BitSet`结合

**`[intersects(BitSet)](https://web.archive.org/web/20220628053548/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/BitSet.html#intersects(java.util.BitSet)) `方法接受另一个`BitSet `并在两个`BitSet`有共同点**时返回`true `。也就是说，它们在同一索引中至少有一个设置位:

```java
BitSet first = new BitSet();
first.set(5, 10);

BitSet second = new BitSet();
second.set(7, 15);

assertThat(first.intersects(second)).isTrue();
```

在两个`BitSet`中都设置了[7，9]范围，因此该方法返回`true`。

**也可以对两个`BitSet`的**执行逻辑`and `运算:

```java
first.and(second);
assertThat(first.get(7)).isTrue();
assertThat(first.get(8)).isTrue();
assertThat(first.get(9)).isTrue();
assertThat(first.get(10)).isFalse();
```

这将在两个`BitSet`之间执行一个逻辑`and `，并用结果修改`first `变量。类似地，我们也可以对两个`BitSet`执行逻辑`xor `:

```java
first.clear();
first.set(5, 10);

first.xor(second);
for (int i = 5; i < 7; i++) {
    assertThat(first.get(i)).isTrue();
}
for (int i = 10; i < 15; i++) {
    assertThat(first.get(i)).isTrue();
}
```

还有其他方法如 [`andNot(BitSet)`](https://web.archive.org/web/20220628053548/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/BitSet.html#andNot(java.util.BitSet)) 或`[or(BitSet)](https://web.archive.org/web/20220628053548/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/BitSet.html#or(java.util.BitSet)),` 可以对两个`BitSet`执行其他逻辑运算

### 4.8.多方面的

从 Java 8 开始，**有一个`[stream()](https://web.archive.org/web/20220628053548/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/BitSet.html#stream()) `方法来流一个`BitSet`** 的所有设置位。例如:

```java
BitSet bitSet = new BitSet();
bitSet.set(15, 25);

bitSet.stream().forEach(System.out::println);
```

这将把所有设置位打印到控制台。由于这将返回一个 [`IntStream`](https://web.archive.org/web/20220628053548/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/IntStream.html) ，我们可以执行常见的数值运算，如求和、平均、计数等等。例如，这里我们正在计算设置位的数量:

```java
assertThat(bitSet.stream().count()).isEqualTo(10);
```

同样，**`[nextSetBit(fromIndex)](https://web.archive.org/web/20220628053548/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/BitSet.html#nextSetBit(int)) `方法将返回从`fromIndex`** 开始的下一个设置位索引:

```java
assertThat(bitSet.nextSetBit(13)).isEqualTo(15);
```

`fromIndex `本身包含在此计算中。当`BitSet`中没有任何`true `位时，它将返回-1:

```java
assertThat(bitSet.nextSetBit(25)).isEqualTo(-1);
```

同样，**`[nextClearBit(fromIndex)](https://web.archive.org/web/20220628053548/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/BitSet.html#nextClearBit(int)) `返回从`fromIndex`** 开始的下一个清除索引:

```java
assertThat(bitSet.nextClearBit(23)).isEqualTo(25);
```

另一方面，`[previousClearBit(fromIndex)](https://web.archive.org/web/20220628053548/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/BitSet.html#previousClearBit(int)) `返回相反方向上最近的清晰索引的索引:

```java
assertThat(bitSet.previousClearBit(24)).isEqualTo(14);
```

[`previousSetBit(fromIndex)`](https://web.archive.org/web/20220628053548/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/BitSet.html#previousSetBit(int)) 也是如此:

```java
assertThat(bitSet.previousSetBit(29)).isEqualTo(24);
assertThat(bitSet.previousSetBit(14)).isEqualTo(-1);
```

此外，我们可以分别使用`[toByteArray()](https://web.archive.org/web/20220628053548/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/BitSet.html#toByteArray()) `或`[toLongArray()](https://web.archive.org/web/20220628053548/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/BitSet.html#toLongArray()) `方法将`BitSet `转换为`byte[] `或`long[] `:

```java
byte[] bytes = bitSet.toByteArray();
long[] longs = bitSet.toLongArray();
```

## 5.结论

在本教程中，我们看到了如何使用`BitSet` s 来表示位向量。

首先，我们熟悉了不使用`boolean[]`来表示位向量的基本原理。然后我们看到了一个`BitSet `如何在内部工作以及它的 API 是什么样子的。

像往常一样，所有的例子都可以在 GitHub 上找到[。](https://web.archive.org/web/20220628053548/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-3)**