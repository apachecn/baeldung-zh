# 在 Java 中生成字母数字 UUID 字符串

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-generate-alphanumeric-uuid>

## 1.概观

[UUID](/web/20220626071559/https://www.baeldung.com/java-uuid) (全球唯一标识符)，也称为 GUID(全球唯一标识符)，是一个 128 位的值，在所有实际用途中都是唯一的。**与大多数其他编号方案不同，它们的唯一性不依赖于中央注册机构或生成它们的各方之间的协调**。

在本教程中，我们将看到在 Java 中生成 UUID 标识符的两种不同的实现方法。

## 2.结构

让我们来看一个 UUID 的例子，后面是 UUID 的规范表示:

```java
123e4567-e89b-42d3-a456-556642440000
xxxxxxxx-xxxx-Bxxx-Axxx-xxxxxxxxxxxx
```

标准表示由 32 个十六进制(16 进制)数字组成，以 8-4-4-4-12 的形式显示在由连字符分隔的五个组中，总共 36 个字符(32 个十六进制字符和 4 个连字符)。

零 UUID 是 UUID 的一种特殊形式，其中所有位都是零。

### 2.1.变体

在上面的标准表示中， **`A`表示 UUID 变体**，它决定了 UUID 的布局。UUID 中的所有其他位取决于变量字段中的位的设置。

变量由`A`的三个最高有效位决定:

```java
 MSB1    MSB2    MSB3
   0       X       X     reserved (0)
   1       0       X     current variant (2)
   1       1       0     reserved for Microsoft (6)
   1       1       1     reserved for future (7)
```

所提到的 UUID 中`A`的值为“a”。“a”(= 10xx)的二进制等价物将变量显示为 2。

### 2.1.版本

再次查看标准表示， **`B`表示版本**。版本字段**保存描述给定 UUID** 的类型的值。上面示例 UUID 中的版本(`B`的值)是 4。

UUIDs 有五种不同的基本类型:

1.  版本 1(基于时间):基于当前时间戳，从 1582 年 10 月 15 日开始以 100 纳秒为单位进行测量，并与创建 UUID 的设备的 MAC 地址相连。
2.  版本 2(DCE-分布式计算环境):使用当前时间，以及本地机器上网络接口的 MAC 地址(或节点)。此外，第 2 版 UUID 将时间字段的低位替换为本地标识符，例如创建 UUID 的本地帐户的用户 ID 或组 ID。
3.  版本 3(基于名称):UUIDs 是使用名称空间和名称的散列生成的。名称空间标识符是 UUIDs，如域名系统(DNS)、对象标识符(oid)和 URL。
4.  版本 4(随机生成):在这个版本中，UUID 标识符是随机生成的，不包含任何关于它们的创建时间或生成它们的机器的信息。
5.  版本 5(基于名称，使用 SHA-1):使用与版本 3 相同的方法生成，不同之处在于哈希算法。这个版本使用命名空间标识符和名称的 SHA-1 (160 位)散列。

## 3.`UUID`类

Java 有一个内置的实现来管理 UUID 标识符，不管我们是想要随机生成 uuid 还是使用构造函数来创建它们。

`UUID`类**有一个单独的构造函数**:

```java
UUID uuid = new UUID(long mostSignificant64Bits, long leastSignificant64Bits);
```

如果我们想使用这个构造函数，我们需要提供两个`long`值。然而，它要求我们自己为 UUID 构建位模式。

为了方便起见，有**三个静态方法来创建一个`UUID`** 。

第一种方法从给定的字节数组创建版本 3 UUID:

```java
UUID uuid = UUID.nameUUIDFromBytes(byte[] bytes);
```

其次，`randomUUID()`方法创建了版本 4 的 UUID。这是创建`UUID`实例最方便的方式:

```java
UUID uuid = UUID.randomUUID();
```

给定给定 UUID 的字符串表示，第三个静态方法返回一个`UUID`对象:

```java
UUID uuid = UUID.fromString(String uuidHexDigitString);
```

现在让我们看看一些不使用内置的`UUID`类来生成 UUIDs 的实现。

## 4 .实现

我们将根据需求将实现分为两类。第一类是只需要唯一的标识符，为此，`UUIDv1`和`UUIDv4`是最好的选择。在第二类中，如果我们需要总是从一个给定的名字生成相同的 UUID，我们将需要一个`UUIDv3`或`UUIDv5`。

因为 RFC 4122 没有指定确切的生成细节，所以在本文中我们不会查看`UUIDv2`的实现。

现在让我们看看我们提到的类别的实现。

### 4.1.版本 1 和 4

首先，如果隐私是一个问题，`UUIDv1`也可以用一个随机的 48 位数字代替 MAC 地址来生成。在本文中，我们将研究这种替代方案。

首先，我们将生成 64 个最低和最高有效位作为`long`值:

```java
private static long get64LeastSignificantBitsForVersion1() {
    Random random = new Random();
    long random63BitLong = random.nextLong() & 0x3FFFFFFFFFFFFFFFL;
    long variant3BitFlag = 0x8000000000000000L;
    return random63BitLong + variant3BitFlag;
}

private static long get64MostSignificantBitsForVersion1() {
    LocalDateTime start = LocalDateTime.of(1582, 10, 15, 0, 0, 0);
    Duration duration = Duration.between(start, LocalDateTime.now());
    long seconds = duration.getSeconds();
    long nanos = duration.getNano();
    long timeForUuidIn100Nanos = seconds * 10000000 + nanos * 100;
    long least12SignificatBitOfTime = (timeForUuidIn100Nanos & 0x000000000000FFFFL) >> 4;
    long version = 1 << 12;
    return 
      (timeForUuidIn100Nanos & 0xFFFFFFFFFFFF0000L) + version + least12SignificatBitOfTime;
}
```

然后我们可以将这两个值传递给`UUID`的构造函数:

```java
public static UUID generateType1UUID() {

    long most64SigBits = get64MostSignificantBitsForVersion1();
    long least64SigBits = get64LeastSignificantBitsForVersion1();

    return new UUID(most64SigBits, least64SigBits);
}
```

我们现在来看看如何生成 UUIDv4。该实现使用随机数作为来源。Java 实现是`SecureRandom`，它使用一个不可预测的值作为种子来生成随机数，目的是为了减少冲突的几率。

让我们生成一个版本 4 `UUID`:

```java
UUID uuid = UUID.randomUUID();
```

然后，让我们使用“SHA-256”和一个随机的`UUID`生成一个唯一的密钥:

```java
MessageDigest salt = MessageDigest.getInstance("SHA-256");
salt.update(UUID.randomUUID().toString().getBytes("UTF-8"));
String digest = bytesToHex(salt.digest());
```

### 4.2.版本 3 和 5

UUIDs 是使用名称空间和名称的散列生成的。名称空间标识符是 UUIDs，如域名系统(DNS)、对象标识符(oid)和 URL。让我们看看算法的伪代码:

```java
UUID = hash(NAMESPACE_IDENTIFIER + NAME)
```

`UUIDv3`和`UUIDv5`的唯一区别是哈希算法——v3 使用 MD5 (128 位)，而 v5 使用 SHA-1 (160 位)。

对于`UUIDv3`,我们将使用来自`UUID`类的方法`nameUUIDFromBytes()`,它接受一个字节数组并应用 MD5 散列。

因此，让我们首先从名称空间和特定名称中提取字节表示，并将它们连接到一个数组中，以将其发送给 UUID api:

```java
byte[] nameSpaceBytes = bytesFromUUID(namespace);
byte[] nameBytes = name.getBytes("UTF-8");
byte[] result = joinBytes(nameSpaceBytes, nameBytes);
```

最后一步是将我们从前面的过程中得到的结果传递给`nameUUIDFromBytes()`方法。该方法还将设置变量和版本字段:

```java
UUID uuid = UUID.nameUUIDFromBytes(result);
```

现在让我们看看`UUIDv5`的实现。需要注意的是，Java 没有提供生成版本 5 的内置实现。

让我们检查代码以生成最低和最高有效位，同样作为`long`值:

```java
public static long getLeastAndMostSignificantBitsVersion5(final byte[] src, final int offset, final ByteOrder order) {
    long ans = 0;
    if (order == ByteOrder.BIG_ENDIAN) {
        for (int i = offset; i < offset + 8; i += 1) {
            ans <<= 8;
            ans |= src[i] & 0xffL;
        }
    } else {
        for (int i = offset + 7; i >= offset; i -= 1) {
            ans <<= 8;
            ans |= src[i] & 0xffL;
        }
    }
    return ans;
}
```

现在，我们需要定义一个方法，该方法需要一个名称来生成 UUID。该方法将使用在`UUID`类中定义的默认构造函数:

```java
private static UUID generateType5UUID(String name) { 
    byte[] bytes = name.getBytes(StandardCharsets.UTF_8);
    MessageDigest md = MessageDigest.getInstance("SHA-1");
    byte[] hash = md.digest(bytes);
    long msb = getLeastAndMostSignificantBitsVersion5(hash, 0, ByteOrder.BIG_ENDIAN);
    long lsb = getLeastAndMostSignificantBitsVersion5(hash, 8, ByteOrder.BIG_ENDIAN);
    msb &= ~(0xfL << 12);
    msb |= ((long) 5) << 12;
    lsb &= ~(0x3L << 62);
    lsb |= 2L << 62;
    return new UUID(msb, lsb);
}
```

## 5.结论

在本文中，我们看到了关于 UUID 标识符的主要概念，以及如何使用内置类生成它们。然后，我们看到了不同版本的 UUIDs 及其应用范围的一些有效实现。

和往常一样，本文的完整代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220626071559/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-uuid/)