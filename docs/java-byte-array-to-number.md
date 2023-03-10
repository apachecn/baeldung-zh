# 在 Java 中将字节数组转换为数字表示

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-byte-array-to-number>

## 1.概观

在本教程中，我们将探索不同的方法将一个`byte`数组转换成一个数值(`int`、`long`、`float`、`double`)，反之亦然。

字节是计算机存储和处理信息的基本单位。Java 语言中定义的[原语类型](/web/20221008082518/https://www.baeldung.com/java-primitives)是同时操作多个字节的一种便捷方式。因此，`byte`数组和基元类型之间存在固有的转换关系。

由于`short`和`char`类型只包含两个字节，所以不需要太多关注。因此，我们将关注于`byte`数组与`int`、`long`、`float`和`double`类型之间的转换。

## 2.使用移位运算符

将`byte`数组转换成数值的最直接的方法是使用[移位操作符](/web/20221008082518/https://www.baeldung.com/java-bitwise-operators#bitwise-shift-operators)。

### 2.1.到`int`和`long`的字节数组

当将一个`byte`数组转换成一个`int`值时，我们使用`<<`(左移)运算符:

```java
int value = 0;
for (byte b : bytes) {
    value = (value << 8) + (b & 0xFF);
}
```

通常，上述代码片段中的`bytes`数组的长度应该等于或小于四。那是因为一个`int`值占据了四个字节。否则，就会导致`int`范围溢出。

为了验证转换的正确性，让我们定义两个常数:

```java
byte[] INT_BYTE_ARRAY = new byte[] {
    (byte) 0xCA, (byte) 0xFE, (byte) 0xBA, (byte) 0xBE
};
int INT_VALUE = 0xCAFEBABE;
```

如果我们仔细观察这两个常数，`INT_BYTE_ARRAY`和`INT_VALUE`，我们会发现它们是十六进制数`0xCAFEBABE`的不同表示。

然后，让我们检查一下这个转换是否正确:

```java
int value = convertByteArrayToIntUsingShiftOperator(INT_BYTE_ARRAY);

assertEquals(INT_VALUE, value);
```

类似地，当将一个`byte`数组转换成一个`long`值时，我们可以重用上面的代码片段，只需做两处修改:`value`的类型是`long`并且`bytes`的长度应该等于或小于 8。

### 2.2.`int`和`long`到字节数组

当将一个`int`值转换为一个`byte`数组时，我们可以使用`>>`(有符号右移)或`>>>`(无符号右移)运算符:

```java
byte[] bytes = new byte[Integer.BYTES];
int length = bytes.length;
for (int i = 0; i < length; i++) {
    bytes[length - i - 1] = (byte) (value & 0xFF);
    value >>= 8;
}
```

在上面的代码片段中，我们可以用`>>>`操作符替换`>>`操作符。这是因为我们只使用了`value`参数最初包含的字节。因此，带符号扩展或零扩展的右移不会影响最终结果。

然后，我们可以检查上述转换的正确性:

```java
byte[] bytes = convertIntToByteArrayUsingShiftOperator(INT_VALUE);

assertArrayEquals(INT_BYTE_ARRAY, bytes);
```

当将一个`long`值转换为一个`byte`数组时，我们只需要将`Integer.BYTES`转换为`Long.BYTES`，并确保`value`的类型为`long`。

### 2.3.到`float`和`double`的字节数组

**当[将一个`byte`数组转换成一个`float`](/web/20221008082518/https://www.baeldung.com/java-convert-float-to-byte-array) 数组时，我们利用了`Float.intBitsToFloat()`方法**:

```java
// convert bytes to int
int intValue = 0;
for (byte b : bytes) {
    intValue = (intValue << 8) + (b & 0xFF);
}

// convert int to float
float value = Float.intBitsToFloat(intValue);
```

从上面的代码片段中，我们可以了解到一个`byte`数组不能直接转换成一个`float`值。基本上，它需要两个独立的步骤:首先，我们将一个`byte`数组转换成一个`int`值，然后我们将相同的位模式解释成一个`float`值。

为了验证转换的正确性，让我们定义两个常数:

```java
byte[] FLOAT_BYTE_ARRAY = new byte[] {
    (byte) 0x40, (byte) 0x48, (byte) 0xF5, (byte) 0xC3
};
float FLOAT_VALUE = 3.14F;
```

然后，让我们检查一下这个转换是否正确:

```java
float value = convertByteArrayToFloatUsingShiftOperator(FLOAT_BYTE_ARRAY);

assertEquals(Float.floatToIntBits(FLOAT_VALUE), Float.floatToIntBits(value));
```

同样，**我们可以利用中间的`long`值和`Double.longBitsToDouble()`方法将`byte`数组转换成`double`值**。

### 2.4.`float`和`double`到字节数组

**当把一个`float`转换成一个`byte`数组时，我们可以利用`Float.floatToIntBits()`方法**:

```java
// convert float to int
int intValue = Float.floatToIntBits(value);

// convert int to bytes
byte[] bytes = new byte[Float.BYTES];
int length = bytes.length;
for (int i = 0; i < length; i++) {
    bytes[length - i - 1] = (byte) (intValue & 0xFF);
    intValue >>= 8;
}
```

然后，让我们检查一下这个转换是否正确:

```java
byte[] bytes = convertFloatToByteArrayUsingShiftOperator(FLOAT_VALUE);

assertArrayEquals(FLOAT_BYTE_ARRAY, bytes);
```

以此类推，**我们可以利用`Double.doubleToLongBits()`方法将一个`double`值转换成一个`byte`数组**。

## 3.使用`ByteBuffer`

**`java.nio.ByteBuffer`类提供了一种简洁、统一的方式来在`byte`数组和数值** ( `int`、`long`、`float`、`double`)之间进行转换。

### 3.1.字节数组到数值

现在，我们使用`ByteBuffer`类将一个`byte`数组转换成一个`int`值:

```java
ByteBuffer buffer = ByteBuffer.allocate(Integer.BYTES);
buffer.put(bytes);
buffer.rewind();
int value = buffer.getInt();
```

然后，我们使用`ByteBuffer`类将一个`int`值转换成一个`byte`数组:

```java
ByteBuffer buffer = ByteBuffer.allocate(Integer.BYTES);
buffer.putInt(value);
buffer.rewind();
byte[] bytes = buffer.array();
```

我们应该注意，上面两个代码片段遵循相同的模式:

*   首先，我们使用`ByteBuffer.allocate(int)`方法获得一个具有指定容量的`ByteBuffer`对象。
*   然后，我们将原始值(一个`byte`数组或一个`int`值)放入`ByteBuffer`对象，如`buffer.put(bytes)`和`buffer.putInt(value)`方法。
*   之后，我们将`ByteBuffer`对象的位置重置为零，这样我们就可以从头开始读取。
*   最后，我们使用`buffer.getInt()`和`buffer.array()`等方法从`ByteBuffer`对象中获取目标值。

这种模式非常通用，它支持`long`、`float`和`double`类型的转换。我们需要做的唯一修改是与类型相关的信息。

### 3.2.使用现有的字节数组

此外，`ByteBuffer.wrap(byte[])`方法允许我们重用现有的`byte`数组，而无需创建新的数组:

```java
ByteBuffer.wrap(bytes).getFloat();
```

但是我们也要注意，上面的`bytes`变量的长度等于或者大于目标类型的大小(`Float.BYTES`)。否则，就会抛出`BufferUnderflowException`。

## 4.使用`BigInteger`

[`java.math.BigInteger`](/web/20221008082518/https://www.baeldung.com/java-biginteger) 类的主要用途是表示大型数值，否则这些数值不适合原始数据类型。**尽管我们可以用它在一个`byte`数组和一个原始值之间进行转换，但是对于这种目的来说，使用`BigInteger`还是有点麻烦。**

### 4.1.到`int`和`long`的字节数组

现在，让我们使用`BigInteger`类将一个`byte`数组转换成一个`int`值:

```java
int value = new BigInteger(bytes).intValue();
```

类似地，`BigInteger`类有一个`longValue()`方法将一个`byte`数组转换成一个`long`值:

```java
long value = new BigInteger(bytes).longValue();
```

此外，`BigInteger`类还有一个`intValueExact()`方法和一个`longValueExact()`方法。**这两种方法都要小心使用**:如果`BigInteger`对象分别超出了`int`或`long`类型的范围，这两种方法都会抛出一个`ArithmeticException`。

当将一个`int`或一个`long`值转换成一个`byte`数组时，我们可以使用相同的代码片段:

```java
byte[] bytes = BigInteger.valueOf(value).toByteArray();
```

然而，`BigInteger`类的`toByteArray()`方法返回最小数量的字节，不一定是四个或八个字节。

### 4.2.到`float`和`double`的字节数组

虽然`BigInteger`类有一个`floatValue()`方法，但是我们不能像预期的那样用它将一个`byte`数组转换成一个`float`值。那么，我们该怎么办呢？我们可以使用一个`int`值作为中间步骤，将一个`byte`数组转换成一个`float`值:

```java
int intValue = new BigInteger(bytes).intValue();
float value = Float.intBitsToFloat(intValue);
```

同样，我们可以将一个`float`值转换成一个`byte`数组:

```java
int intValue = Float.floatToIntBits(value);
byte[] bytes = BigInteger.valueOf(intValue).toByteArray();
```

同样，通过利用`Double.longBitsToDouble()`和`Double.doubleToLongBits()`方法，我们可以使用`BigInteger`类在`byte`数组和`double`值之间进行转换。

## 5.用番石榴

番石榴库为我们提供了进行这种转换的便捷方法。

### 5.1.到`int`和`long`的字节数组

在 Guava 中，`com.google.common.primitives`包中的`Ints`类包含一个`fromByteArray()`方法。因此，我们很容易将一个`byte`数组转换成一个`int`值:

```java
int value = Ints.fromByteArray(bytes);
```

`Ints`类也有一个`toByteArray()`方法，可以用来将一个`int`值转换成一个`byte`数组:

```java
byte[] bytes = Ints.toByteArray(value);
```

并且，`Longs`类在使用上类似于`Ints`类:

```java
long value = Longs.fromByteArray(bytes);
byte[] bytes = Longs.toByteArray(value);
```

此外，如果我们检查`fromByteArray()`和`toByteArray()`方法的源代码，我们可以发现**这两种方法都使用移位操作符来完成它们的任务**。

### 5.2.到`float`和`double`的字节数组

在同一个包中还存在`Floats`和`Doubles`类。但是，这两个类都不支持`fromByteArray()`和`toByteArray()`方法。

但是，我们可以利用`Float.intBitsToFloat()`、`Float.floatToIntBits()`、`Double.longBitsToDouble()`和`Double.doubleToLongBits()`方法来完成`byte`数组和`float`或`double`值之间的转换。为了简洁起见，我们在这里省略了代码。

## 6.使用公共语言

当我们使用 [Apache Commons Lang 3](/web/20221008082518/https://www.baeldung.com/java-commons-lang-3) 时，做这种转换有点复杂。这是因为**Commons Lang 库默认使用 little-endian `byte`数组**。但是，我们上面提到的`byte`数组都是大端顺序。因此，我们需要将大端`byte`数组转换成小端`byte`数组，反之亦然。

### 6.1.到`int`和`long`的字节数组

`org.apache.commons.lang3`包中的`Conversion`类提供了`byteArrayToInt()`和`intToByteArray()`方法。

现在，让我们将一个`byte`数组转换成一个`int`值:

```java
byte[] copyBytes = Arrays.copyOf(bytes, bytes.length);
ArrayUtils.reverse(copyBytes);
int value = Conversion.byteArrayToInt(copyBytes, 0, 0, 0, copyBytes.length);
```

**在上面的代码中，我们复制了原来的`bytes`变量。这是因为有时候，我们不想改变原来的`byte`数组的内容。**

然后，让我们将一个`int`值转换成一个`byte`数组:

```java
byte[] bytes = new byte[Integer.BYTES];
Conversion.intToByteArray(value, 0, bytes, 0, bytes.length);
ArrayUtils.reverse(bytes);
```

`Conversion`类还定义了`byteArrayToLong()`和`longToByteArray()`方法。并且，我们可以使用这两种方法在一个`byte`数组和一个`long`值之间进行转换。

### 6.2.到`float`和`double`的字节数组

然而，`Conversion`类并不直接提供相应的方法来转换`float`或`double`值。

同样，我们需要一个中间值`int`或`long`在一个`byte`数组和一个`float`或`double`值之间进行转换。

## 7.结论

在本文中，我们通过移位运算符、`ByteBuffer`和`BigInteger`说明了使用普通 Java 将`byte`数组转换为数值的各种方法。然后，我们看到了使用 Guava 和 Apache Commons Lang 的相应转换。

像往常一样，本教程的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221008082518/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-arrays-convert)