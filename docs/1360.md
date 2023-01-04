# 清除 StringBuilder 或 StringBuffer

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-clear-stringbuilder-stringbuffer>

## 1.概观

在本教程中，我们将介绍几种清除 [`StringBuilder`或`StringBuffer`](/web/20220824120057/https://www.baeldung.com/java-string-builder-string-buffer) 的方法，然后对它们进行详细说明。

## 2.清除 a `StringBuilder`

### 2.1.使用 `setLength`方法

**方法`setLength` 更新`StringBuilder`的内部长度。**当操作`StringBuilder`T3 时，长度之后的所有条目都被忽略。因此，用 0 调用它会清除其内容:

```
@Test
void whenSetLengthToZero_ThenStringBuilderIsCleared() {
    StringBuilder stringBuilder = new StringBuilder();
    stringBuilder.append("Hello World");
    int initialCapacity = stringBuilder.capacity();
    stringBuilder.setLength(0);
    assertEquals("", stringBuilder.toString());
    assertEquals(initialCapacity, stringBuilder.capacity();
}
```

让我们注意，在我们调用了`setLength`方法之后，`StringBuilder`的容量保持不变。

### 2.2.使用`delete`方法

**`delete`方法在后台使用 [`System.arraycopy`](/web/20220824120057/https://www.baeldung.com/java-array-copy) 。起始索引之前或结束索引之后的所有索引都被复制到同一个`StringBuilder`。**

因此，如果我们调用起始索引为 0 且结束索引等于`StringBuilder`长度的`delete`，我们将复制:

*   0 之前的索引:没有。
*   `stringBuilder.length()`之后的索引:没有。

因此，`StringBuilder`的所有内容都被删除:

```
@Test
void whenDeleteAll_ThenStringBuilderIsCleared() {
    StringBuilder stringBuilder = new StringBuilder();
    stringBuilder.append("Hello World");
    int initialCapacity = stringBuilder.capacity();
    stringBuilder.delete(0, stringBuilder.length());
    assertEquals("", stringBuilder.toString());
    assertEquals(initialCapacity, stringBuilder.capacity();
}
```

与`setLength`方法一样，对象容量在删除其内容后保持不变。我们还要强调的是，在这个过程中没有涉及到新对象的创建。

## 3.清除 a `StringBuffer`

**所有适用于`StringBuilder` 的方法与`StringBuffer`的工作方式相同。**此外，所有关于客体能力的言论仍然有效。
让我们用`setLength` 法展示一个例子:

```
@Test
void whenSetLengthToZero_ThenStringBufferIsCleared() {
    StringBuffer stringBuffer = new StringBuffer();
    stringBuffer.append("Hello World");
    int initialCapacity = stringBuffer.capacity();
    stringBuffer.setLength(0);
    assertEquals("", stringBuffer.toString());
    assertEquals(initialCapacity, stringBuffer.capacity();
}
```

 `也可以使用`delete`方法:

```
@Test
void whenDeleteAll_ThenStringBufferIsCleared() {
    StringBuffer stringBuffer = new StringBuffer();
    stringBuffer.append("Hello World");
    int initialCapacity = stringBuffer.capacity();
    stringBuffer.delete(0, stringBuffer.length());
    assertEquals("", stringBuffer.toString());
    assertEquals(initialCapacity, stringBuffer.capacity();
}
```

## 4.表演

让我们和 [JMH](/web/20220824120057/https://www.baeldung.com/java-microbenchmark-harness) 做一个快速的性能比较。让我们比较一下`StringBuilder`的三种方法:

```
@State(Scope.Benchmark)
public static class MyState {
    final String HELLO = "Hello World";
    final StringBuilder sb = new StringBuilder().append(HELLO);
}

@Benchmark
public void evaluateSetLength(Blackhole blackhole, MyState state) {
    state.sb.setLength(0);
    blackhole.consume(state.sb.toString());
}

@Benchmark
public void evaluateDelete(Blackhole blackhole, MyState state) {
    state.sb.delete(0, state.sb.length());
    blackhole.consume(state.sb.toString());
}
```

我们已经测量了每秒的运算次数。该基准导致以下结果:

```
Benchmark                  Mode   Cnt         Score          Error  Units
evaluateDelete             thrpt   25  67943684.417 ± 18116791.770  ops/s
evaluateSetLength          thrpt   25  37310891.158 ±   994382.978  ops/s
```

正如我们所见，`delete`似乎是两种方法中耗时更少的一种，几乎是前者的两倍。

## 5.结论

在本文中，我们详细介绍了三种清除`StringBuilder`或`StringBuffer`的方法。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20220824120057/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-apis)`