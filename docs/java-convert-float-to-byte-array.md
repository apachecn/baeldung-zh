# 在 Java 中将浮点数转换成字节数组

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-convert-float-to-byte-array>

## 1.概观

在这个快速教程中，我们将探索几个使用 Java 将浮点数转换成字节数组的例子，反之亦然。

如果我们将 int 或 long 转换为 byte 数组，这就很简单了，因为 Java 位操作符只对整数类型有效。但是，对于 float，我们需要使用另一层转换。

例如，我们可以使用由`java.nio`包的`the Float`类或`ByteBuffer`类提供的 API。

## 2.浮点到字节数组转换

众所周知，Java 中浮点的大小是 32 位，类似于 int。所以我们可以使用 Java 的`Float`类中可用的`floatToIntBits or floatToRawIntBits`函数。然后移位这些位以返回一个字节数组。点击[此处](https://web.archive.org/web/20221129011911/https://docs.oracle.com/javase/tutorial/java/nutsandbolts/op3.html)了解更多关于移位操作的信息。

两者的区别在于`floatToRawIntBits`也保留了非数字(NaN)值。这里移位是通过一种叫做[缩小原始转换](https://web.archive.org/web/20221129011911/https://docs.oracle.com/javase/specs/jls/se10/html/jls-5.html#jls-5.1.3)的技术完成的。

首先让我们看一下使用 Float 类函数的代码:

```java
public static byte[] floatToByteArray(float value) {
    int intBits =  Float.floatToIntBits(value);
    return new byte[] {
      (byte) (intBits >> 24), (byte) (intBits >> 16), (byte) (intBits >> 8), (byte) (intBits) };
}
```

其次是一种简洁的转换方式，使用`ByteBuffer`:

```java
ByteBuffer.allocate(4).putFloat(value).array();
```

## 3.字节数组到浮点的转换

现在让我们使用`Float`类函数`intBitsToFloat`将一个字节数组转换成一个浮点数。

然而，我们需要首先使用左移将字节数组转换为 int 位:

```java
public static float byteArrayToFloat(byte[] bytes) {
    int intBits = 
      bytes[0] << 24 | (bytes[1] & 0xFF) << 16 | (bytes[2] & 0xFF) << 8 | (bytes[3] & 0xFF);
    return Float.intBitsToFloat(intBits);  
}
```

使用`ByteBuffer`将一个字节数组转换成一个浮点数就像这样简单:

```java
ByteBuffer.wrap(bytes).getFloat(); 
```

## 4.单元测试

让我们来看看简单的单元测试实现案例:

```java
public void givenAFloat_thenConvertToByteArray() {
    assertArrayEquals(new byte[] { 63, -116, -52, -51}, floatToByteArray(1.1f));
}

@Test
public void givenAByteArray_thenConvertToFloat() {
   assertEquals(1.1f, byteArrayToFloat(new byte[] { 63, -116, -52, -51}), 0);
}
```

## 5.结论

我们已经看到了不同的浮点到字节的转换方式，反之亦然。

类提供函数作为这种转换的变通方法。然而，`ByteBuffer`提供了一种简洁的方式来做到这一点。因此，我建议尽可能使用它。

这些实现和单元测试用例的完整源代码可以在 [GitHub 项目](https://web.archive.org/web/20221129011911/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-arrays-convert)中找到。