# 爪哇 UUID 旅游指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-uuid>

## 1。概述

UUID (全球唯一标识符)，也称为 GUID(全球唯一标识符)，表示**一个 128 位长的值，在所有实际用途中都是唯一的。**UUID 的标准表示法使用十六进制数字(八进制):

```java
123e4567-e89b-12d3-a456-556642440000
```

UUID 由十六进制数字(每个数字 4 个字符)和 4 个“-”符号组成，这使得它的**长度等于 36 个字符。**

零 UUID 是 UUID 的一种特殊形式，其中所有位都被设置为零。

在本教程中，我们将看看 Java 中的`UUID`类。首先，我们将看到如何使用类本身。然后我们将看看不同类型的 UUIDs，以及如何在 Java 中生成它们。

## 延伸阅读:

## [Java 中的字符序列与字符串](/web/20220819035016/https://www.baeldung.com/java-char-sequence-string)

Learn the differences between CharSequence and String.[Read more](/web/20220819035016/https://www.baeldung.com/java-char-sequence-string) →

## 在 Java 中使用字符串上的 char[]数组来操作密码？

Explore several reasons why we shouldn't use Strings for storing passwords and use char[] arrays instead.[Read more](/web/20220819035016/https://www.baeldung.com/java-storing-passwords) →

## [Java 字符串池指南](/web/20220819035016/https://www.baeldung.com/java-string-pool)

Learn how the JVM optimizes the amount of memory allocated to String storage in the Java String Pool.[Read more](/web/20220819035016/https://www.baeldung.com/java-string-pool) →

## 2.`UUID`类

UUID 类只有一个构造函数:

```java
UUID uuid = new UUID(long mostSignificant64Bits, long leastSignificant64Bits);
```

如果我们想使用这个构造函数，我们需要提供两个长值。然而，它要求我们自己为 UUID 构建位模式。

为了方便起见，有三种静态方法来创建 UUID。

第一种方法从给定的字节数组创建第 3 版 UUID:

```java
UUID uuid = UUID.nameUUIDFromBytes(byte[] bytes); 
```

**其次，`randomUUID()`方法创建了版本 4 的 UUID。这是创造 UUID** 最便捷的方式:

```java
UUID uuid = UUID.randomUUID(); 
```

第三个静态方法返回给定 UUID 的字符串表示形式的 UUID 对象:

```java
UUID uuid = UUID.fromString(String uuidHexDigitString); 
```

现在让我们看看 UUID 是如何构成的。

## 3。结构

让我们以 UUID 为例:

```java
123e4567-e89b-42d3-a456-556642440000
xxxxxxxx-xxxx-Bxxx-Axxx-xxxxxxxxxxxx
```

### 3.1.UUID 变体

`**A**`代表决定 UUID 布局的变量。UUID 中的所有其他位取决于变量字段中的位的设置。变量由 A 的三个最高有效位决定:

```java
 MSB1    MSB2    MSB3
   0       X       X     reserved (0)
   1       0       X     current variant (2)
   1       1       0     reserved for Microsoft (6)
   1       1       1     reserved for future (7)
```

所提到的 UUID 中`**A**`的值为“a”。“a”(= 10xx)的二进制等价物将变量显示为 2。

### 3.2.UUID 版本

`**B**`代表版本。提到的 UUID 中的版本(`**B**`的值)是 4。

Java 提供了获取 UUID 变体和版本的方法:

```java
UUID uuid = UUID.randomUUID();
int variant = uuid.variant();
int version = uuid.version();
```

变体 2 UUIDs 有五个不同的版本:基于时间的(UUIDv1)、DCE 安全的(UUIDv2)、基于名称的(UUIDv3 和 UUIDv5)和随机的(UUIDv4)。

Java 提供了 v3 和 v4 的实现，但也提供了用于生成任何类型的 UUID 的`constructor`:

```java
UUID uuid = new UUID(long mostSigBits, long leastSigBits);
```

## 4.UUID 版本

### 4.1。版本 1

UUID 版本 1 基于当前时间戳，从 1582 年 10 月 15 日开始以 100 纳秒为单位进行测量，并与创建 UUID 的设备的 MAC 地址相连。

如果担心隐私问题，也可以用随机的 48 位数字代替 MAC 地址来生成 UUID 版本 1。在本文中，我们将研究这种替代方案。

首先，我们将生成 64 个最低和最高有效位作为长值:

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

然后，我们可以将这两个值传递给 UUID 的构造函数:

```java
public static UUID generateType1UUID() {

    long most64SigBits = get64MostSignificantBitsForVersion1();
    long least64SigBits = get64LeastSignificantBitsForVersion1();

    return new UUID(most64SigBits, least64SigBits);
}
```

### 4.2。版本 2

版本 2 基于时间戳和 MAC 地址。然而， [RFC 4122](https://web.archive.org/web/20220819035016/https://tools.ietf.org/html/rfc4122) 并没有指定确切的生成细节，所以我们不会在本文中查看实现。

### 4.3。版本 3 和 5

UUIDs 是使用名称空间和名称的散列生成的。名称空间标识符是 UUIDs，如域名系统(DNS)、对象标识符(oid)、URL 等。

```java
UUID = hash(NAMESPACE_IDENTIFIER + NAME)
```

UUIDv3 和 UUIDv5 之间的唯一区别是哈希算法— v3 使用 MD5 (128 位)，而 v5 使用 SHA-1 (160 位)。

简而言之，我们将产生的哈希截断为 128 位，然后替换版本的 4 位和变体的 2 位。

让我们生成第三类 UUID:

```java
byte[] nameSpaceBytes = bytesFromUUID(namespace);
byte[] nameBytes = name.getBytes("UTF-8");
byte[] result = joinBytes(nameSpaceBytes, nameBytes);

UUID uuid = UUID.nameUUIDFromBytes(result);
```

这里，需要注意的是，名称空间的十六进制字符串首先需要转换为字节数组。

最后，Java 没有提供类型 5 的实现。检查 UUIDv5 的源代码库。

### 4.4。版本 4

UUIDv4 实现使用随机数作为来源。Java 实现是`SecureRandom`，用一个不可预知的值作为种子生成随机数，减少碰撞的几率。

让我们生成版本 4 UUID:

```java
UUID uuid = UUID.randomUUID();
```

让我们使用“SHA-256”和一个随机 UUID 生成一个唯一的密钥:

```java
MessageDigest salt = MessageDigest.getInstance("SHA-256");
salt.update(UUID.randomUUID().toString().getBytes("UTF-8"));
String digest = bytesToHex(salt.digest());
```

## 5。结论

在本文中，我们看到了 UUID 是如何构造的，以及有哪些变体和版本。

我们还了解了 Java 为哪些版本提供了开箱即用的实现，并查看了生成其他版本的代码示例。

和往常一样，GitHub 上的[提供了实现的源代码。](https://web.archive.org/web/20220819035016/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-uuid/)*