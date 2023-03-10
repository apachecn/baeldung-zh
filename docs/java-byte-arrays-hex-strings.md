# Java 中字节数组和十六进制字符串之间的转换

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-byte-arrays-hex-strings>

## 1.概观

在本教程中，我们将看看将字节数组转换成十六进制`String,`的不同方法，反之亦然。

我们还将理解转换机制，并编写实现来实现这一点。

## 2.在字节和十六进制之间转换

首先，我们来看看字节数和十六进制数之间的转换逻辑。

### 2.1.字节到十六进制

在 Java 中，字节是 8 位有符号整数。因此，我们需要**将每个 4 位段分别转换为十六进制，并将它们串联起来**。因此，转换后我们将得到两个十六进制字符。

例如，我们可以将 45 写成二进制的 0010 1101，十六进制的等价形式是“2d”:

```java
0010 = 2 (base 10) = 2 (base 16)
1101 = 13 (base 10) = d (base 16)

Therefore: 45 = 0010 1101 = 0x2d 
```

让我们用 Java 实现这个简单的逻辑:

```java
public String byteToHex(byte num) {
    char[] hexDigits = new char[2];
    hexDigits[0] = Character.forDigit((num >> 4) & 0xF, 16);
    hexDigits[1] = Character.forDigit((num & 0xF), 16);
    return new String(hexDigits);
}
```

现在，让我们通过分析每个操作来理解上面的代码。首先，我们创建了一个长度为 2 的 char 数组来存储输出:

```java
char[] hexDigits = new char[2];
```

接下来，我们通过右移 4 位来隔离高阶位。然后，我们应用一个掩码来隔离低阶 4 位。需要屏蔽，因为负数在内部表示为正数的[二进制补码](/web/20220908152252/https://www.baeldung.com/cs/two-complement):

```java
hexDigits[0] = Character.forDigit((num >> 4) & 0xF, 16);
```

然后，我们将剩余的 4 位转换为十六进制:

```java
hexDigits[1] = Character.forDigit((num & 0xF), 16);
```

最后，我们从 char 数组中创建一个`String`对象。然后，将此对象作为转换后的十六进制数组返回。

现在，让我们理解这对于负字节 4 是如何工作的:

```java
hexDigits[0]:
1111 1100 >> 4 = 1111 1111 1111 1111 1111 1111 1111 1111
1111 1111 1111 1111 1111 1111 1111 1111 & 0xF = 0000 0000 0000 0000 0000 0000 0000 1111 = 0xf

hexDigits[1]:
1111 1100 & 0xF = 0000 1100 = 0xc

Therefore: -4 (base 10) = 1111 1100 (base 2) = fc (base 16)
```

同样值得注意的是，`Character.` forDigit `()`方法总是返回小写字符。

### 2.2.十六进制到字节

现在，让我们将十六进制数字转换成字节。我们知道，一个字节包含 8 位。因此，**我们需要两个十六进制数字来创建一个字节**。

首先，我们将把每个十六进制数字分别转换成等价的二进制数字。

然后，我们需要连接两个四位段，以获得等效的字节:

```java
Hexadecimal: 2d
2 = 0010 (base 2)
d = 1101 (base 2)

Therefore: 2d = 0010 1101 (base 2) = 45
```

现在，让我们用 Java 编写操作:

```java
public byte hexToByte(String hexString) {
    int firstDigit = toDigit(hexString.charAt(0));
    int secondDigit = toDigit(hexString.charAt(1));
    return (byte) ((firstDigit << 4) + secondDigit);
}

private int toDigit(char hexChar) {
    int digit = Character.digit(hexChar, 16);
    if(digit == -1) {
        throw new IllegalArgumentException(
          "Invalid Hexadecimal Character: "+ hexChar);
    }
    return digit;
}
```

让我们明白这一点，一次一个操作。

首先，我们将十六进制字符转换成整数:

```java
int firstDigit = toDigit(hexString.charAt(0));
int secondDigit = toDigit(hexString.charAt(1));
```

然后，我们将最高有效位左移 4 位。因此，二进制表示在四个最低有效位具有零。

然后，我们添加了最低有效数字:

```java
return (byte) ((firstDigit << 4) + secondDigit);
```

现在，让我们仔细检查一下`toDigit()`方法。我们使用`Character.digit()`方法进行转换。**如果传递给该方法的字符值不是指定基数中的有效数字，则返回-1。**

我们将验证返回值，如果传递了无效值，将抛出异常。

## 3.在字节数组和十六进制之间转换`Strings`

此时，我们知道如何将一个字节转换成十六进制，反之亦然。让我们扩展这个算法，将字节数组转换为十六进制`String`。

### 3.1.字节数组到十六进制`String`

我们需要遍历数组，为每个字节生成十六进制对:

```java
public String encodeHexString(byte[] byteArray) {
    StringBuffer hexStringBuffer = new StringBuffer();
    for (int i = 0; i < byteArray.length; i++) {
        hexStringBuffer.append(byteToHex(byteArray[i]));
    }
    return hexStringBuffer.toString();
}
```

正如我们已经知道的，输出将总是小写。

### 3.2.十六进制字符串到字节数组

首先，我们需要检查十六进制`String`的长度是否为偶数。这是因为奇数长度的十六进制`String`会导致不正确的字节表示。

现在，我们将遍历数组并将每个十六进制对转换为一个字节:

```java
public byte[] decodeHexString(String hexString) {
    if (hexString.length() % 2 == 1) {
        throw new IllegalArgumentException(
          "Invalid hexadecimal String supplied.");
    }

    byte[] bytes = new byte[hexString.length() / 2];
    for (int i = 0; i < hexString.length(); i += 2) {
        bytes[i / 2] = hexToByte(hexString.substring(i, i + 2));
    }
    return bytes;
}
```

## 4.使用`BigInteger`类

我们可以通过传递符号和字节数组来**创建一个类型为`BigInteger` 的对象。**

现在，我们可以在`String`类中定义的静态方法格式的帮助下生成十六进制的`String`:

```java
public String encodeUsingBigIntegerStringFormat(byte[] bytes) {
    BigInteger bigInteger = new BigInteger(1, bytes);
    return String.format(
      "%0" + (bytes.length << 1) + "x", bigInteger);
}
```

提供的格式将生成一个零填充的小写十六进制`String.`我们也可以通过用“X”替换“X”来生成一个大写字符串。

或者，我们可以使用来自`BigInteger`的`toString()`方法。使用`toString()`方法的微妙**区别在于输出没有用前导零填充**:

```java
public String encodeUsingBigIntegerToString(byte[] bytes) {
    BigInteger bigInteger = new BigInteger(1, bytes);
    return bigInteger.toString(16);
}
```

现在，让我们来看看十六进制的`String`到`byte`数组的转换:

```java
public byte[] decodeUsingBigInteger(String hexString) {
    byte[] byteArray = new BigInteger(hexString, 16)
      .toByteArray();
    if (byteArray[0] == 0) {
        byte[] output = new byte[byteArray.length - 1];
        System.arraycopy(
          byteArray, 1, output, 
          0, output.length);
        return output;
    }
    return byteArray;
}
```

**`toByteArray()`方法产生一个附加的符号位**。我们已经编写了处理这个额外位的特定代码。

因此，在使用`BigInteger`类进行转换之前，我们应该了解这些细节。

## 5.使用`DataTypeConverter`类

JAXB 库提供了`DataTypeConverter`类。在 Java 8 之前，这是标准库的一部分。从 Java 9 开始，我们需要向运行时显式添加`java.xml.bind`模块。

让我们看看使用`DataTypeConverter`类的实现:

```java
public String encodeUsingDataTypeConverter(byte[] bytes) {
    return DatatypeConverter.printHexBinary(bytes);
}

public byte[] decodeUsingDataTypeConverter(String hexString) {
    return DatatypeConverter.parseHexBinary(hexString);
}
```

如上图所示，使用`DataTypeConverter`类非常方便。`printHexBinary()`方法的**输出总是大写的**。这个类为数据类型转换提供了一组打印和解析方法。

在选择这种方法之前，我们需要确保该类在运行时可用。

## 6.使用 Apache 的公共编解码器库

我们可以使用 Apache commons-codec 库提供的 [`Hex`](https://web.archive.org/web/20220908152252/https://commons.apache.org/proper/commons-codec/apidocs/org/apache/commons/codec/binary/Hex.html) 类:

```java
public String encodeUsingApacheCommons(byte[] bytes) 
  throws DecoderException {
    return Hex.encodeHexString(bytes);
}

public byte[] decodeUsingApacheCommons(String hexString) 
  throws DecoderException {
    return Hex.decodeHex(hexString);
}
```

`encodeHexString`的**输出总是小写**。

## 7.使用谷歌的番石榴图书馆

让我们来看看 [`BaseEncoding`](https://web.archive.org/web/20220908152252/https://google.github.io/guava/releases/16.0/api/docs/com/google/common/io/BaseEncoding.html) 类是如何用于将字节数组编码和解码为十六进制的`String:`

```java
public String encodeUsingGuava(byte[] bytes) {
    return BaseEncoding.base16().encode(bytes);
}

public byte[] decodeUsingGuava(String hexString) {
    return BaseEncoding.base16()
      .decode(hexString.toUpperCase());
} 
```

**`BaseEncoding`默认使用大写字符**进行编码和解码。如果我们需要使用小写字符，应该使用静态方法[小写](https://web.archive.org/web/20220908152252/https://google.github.io/guava/releases/16.0/api/docs/com/google/common/io/BaseEncoding.html#lowerCase())创建一个新的编码实例。

## 8.结论

在本文中，我们学习了字节数组到十六进制的转换算法`String`。我们还讨论了将字节数组编码为十六进制字符串的各种方法，反之亦然。

不建议添加一个只使用几个实用方法的库。因此，如果我们还没有使用外部库，我们应该使用讨论过的算法。`DataTypeConverter`类是在各种数据类型之间编码/解码的另一种方式。

最后，本教程的完整源代码[可在 GitHub](https://web.archive.org/web/20220908152252/https://github.com/eugenp/tutorials/tree/master/algorithms-modules/algorithms-miscellaneous-5) 上获得。