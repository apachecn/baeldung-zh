# Java 中字节数组和 UUID 之间的转换

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-byte-array-to-uuid>

## 1.概观

在这个简短的教程中，我们将看到如何在 Java 中的**字节数组和 [`UUID`](/web/20220923232236/https://www.baeldung.com/java-uuid)** 之间进行转换。

## 2.将`UUID`转换为字节数组

在普通 Java 中，我们可以很容易地将`UUID`转换成字节数组:

```
public static byte[] convertUUIDToBytes(UUID uuid) {
    ByteBuffer bb = ByteBuffer.wrap(new byte[16]);
    bb.putLong(uuid.getMostSignificantBits());
    bb.putLong(uuid.getLeastSignificantBits());
    return bb.array();
}
```

## 3.将字节数组转换为`UUID`

将一个字节数组转换成`UUID`也很简单:

```
public static UUID convertBytesToUUID(byte[] bytes) {
    ByteBuffer byteBuffer = ByteBuffer.wrap(bytes);
    long high = byteBuffer.getLong();
    long low = byteBuffer.getLong();
    return new UUID(high, low);
}
```

## 4.测试我们的方法

让我们测试一下我们的方法:

```
UUID uuid = UUID.randomUUID();
System.out.println("Original UUID: " + uuid);

byte[] bytes = convertUUIDToBytes(uuid);
System.out.println("Converted byte array: " + Arrays.toString(bytes));

UUID uuidNew = convertBytesToUUID(bytes);
System.out.println("Converted UUID: " + uuidNew);
```

结果将类似于:

```
Original UUID: bd9c7f32-8010-4cfe-97c0-82371e3276fa
Converted byte array: [-67, -100, 127, 50, -128, 16, 76, -2, -105, -64, -126, 55, 30, 50, 118, -6]
Converted UUID: bd9c7f32-8010-4cfe-97c0-82371e3276fa
```

## 5.结论

在这个快速教程中，我们学习了如何在 Java 中的字节数组和 T2 之间进行转换。

和往常一样，本文的示例代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220923232236/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-8-2)