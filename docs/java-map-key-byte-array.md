# 在 Java 中使用字节数组作为映射键

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-map-key-byte-array>

## 1.介绍

在本教程中，我们将学习如何使用一个字节数组作为 [`HashMap`](/web/20221126220806/https://www.baeldung.com/java-hashmap) 中的一个键。不幸的是，由于`HashMap`的工作方式，我们不能直接这么做。我们将调查这是为什么，并寻找几种方法来解决这个问题。

## 2.为`HashMap`设计一把好钥匙

### 2.1.`HashMap`如何工作

`HashMap`使用哈希的[机制来存储和检索自身的值。**当我们调用`put(key, value)`方法时，`HashMap`根据键的`hashCode()`方法计算散列码。**该散列用于标识最终存储值的桶:](/web/20221126220806/https://www.baeldung.com/java-hashmap#internals-hashmap)

```java
public V put(K key, V value) {
    if (key == null)
        return putForNullKey(value);
    int hash = hash(key.hashCode());
    int i = indexFor(hash, table.length);
    for (Entry e = table[i]; e != null; e = e.next) {
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }

    modCount++;
    addEntry(hash, key, value, i);
    return null;
}
```

当我们使用`get(key)`方法获取一个值时，会涉及到一个类似的过程。该密钥用于计算哈希代码，然后找到存储桶。**然后使用`equals()`方法检查桶中的每个条目是否相等。**最后，返回匹配条目的值:

```java
public V get(Object key) {
    if (key == null)
        return getForNullKey();
    int hash = hash(key.hashCode());
    for (Entry e = table[indexFor(hash, table.length)]; e != null; e = e.next) {
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k)))
            return e.value;
    }
    return null;
}
```

### 2.2.`equals`()与`hashCode`()之间的合同

[`equals`和`hashCode`方法都有应该遵守的契约](/web/20221126220806/https://www.baeldung.com/java-equals-hashcode-contracts)。在`HashMaps`的上下文中，有一个方面特别重要:**彼此相等的对象必须返回相同的`hashCode`** 。然而，返回相同`hashCode`的对象不需要彼此相等。这就是为什么我们可以在一个桶中存储多个值。

### 2.3.不变

**键在`HashMap`中的 `hashCode`不应该改变。**虽然这不是强制性的，但是强烈建议键不可变。如果一个对象是不可变的，那么不管`hashCode`方法的实现如何，它的`hashCode`都没有机会改变。

默认情况下，哈希是基于对象的所有字段计算的。如果我们想要一个可变的键，我们需要覆盖`hashCode`方法以确保可变的字段不在它的计算中使用。为了维护契约，我们还需要更改`equals`方法。

### 2.4.有意义的平等

为了能够成功地从地图中检索值，等式必须有意义。在大多数情况下，我们需要能够创建一个新的 key 对象，它将等于 map 中的某个现有 key。出于这个原因，对象标识在这种情况下不是很有用。

这也是为什么使用原始字节数组不是一个真正的选择的主要原因。Java 中的数组使用对象标识来确定相等性。如果我们用字节数组作为键创建`HashMap`,我们将能够仅使用完全相同的数组对象来检索值。

让我们用一个字节数组作为键创建一个简单的实现:

```java
byte[] key1 = {1, 2, 3};
byte[] key2 = {1, 2, 3};
Map<byte[], String> map = new HashMap<>();
map.put(key1, "value1");
map.put(key2, "value2");
```

我们不仅有两个条目具有几乎相同的键，而且我们不能使用新创建的具有相同值的数组来检索任何内容:

```java
String retrievedValue1 = map.get(key1);
String retrievedValue2 = map.get(key2);
String retrievedValue3 = map.get(new byte[]{1, 2, 3});

assertThat(retrievedValue1).isEqualTo("value1");
assertThat(retrievedValue2).isEqualTo("value2");
assertThat(retrievedValue3).isNull();
```

## 3.使用现有容器

我们可以使用现有的类来代替字节数组，这些类的等式实现是基于内容的，而不是基于对象标识的。

### 3.1.`String`

`String`相等是基于字符数组的内容:

```java
public boolean equals(Object anObject) {
    if (this == anObject) {
        return true;
    }
    if (anObject instanceof String) {
        String anotherString = (String)anObject;
        int n = count;
        if (n == anotherString.count) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = offset;
            int j = anotherString.offset;
            while (n-- != 0) {
                if (v1[i++] != v2[j++])
                   return false;
            }
            return true;
        }
    }
    return false;
}
```

`String`也是不可变的，基于字节数组创建`String`相当简单。我们可以使用 [`Base64`](https://web.archive.org/web/20221126220806/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Base64.html) 方案轻松地对`String`进行编码和解码:

```java
String key1 = Base64.getEncoder().encodeToString(new byte[]{1, 2, 3});
String key2 = Base64.getEncoder().encodeToString(new byte[]{1, 2, 3});
```

现在我们可以用`String`创建一个`HashMap`作为键，而不是字节数组。我们将以类似于前一个例子的方式将值放入`Map`:

```java
Map<String, String> map = new HashMap<>();
map.put(key1, "value1");
map.put(key2, "value2");
```

然后我们可以从地图中检索一个值。对于这两个键，我们将得到相同的第二个值。我们还可以检查密钥是否真的彼此相等:

```java
String retrievedValue1 = map.get(key1);
String retrievedValue2 = map.get(key2);

assertThat(key1).isEqualTo(key2);
assertThat(retrievedValue1).isEqualTo("value2");
assertThat(retrievedValue2).isEqualTo("value2");
```

### 3.2.列表

与`String`类似，`List#equals`方法将检查它的每个元素是否相等。如果这些元素有一个合理的`equals()`方法并且是不可变的，`List`将作为`HashMap`键正确工作。我们只需要**确保我们使用的是不可变的`List`实现**:

```java
List<Byte> key1 = ImmutableList.of((byte)1, (byte)2, (byte)3);
List<Byte> key2 = ImmutableList.of((byte)1, (byte)2, (byte)3);
Map<List<Byte>, String> map = new HashMap<>();
map.put(key1, "value1");
map.put(key2, "value2");

assertThat(map.get(key1)).isEqualTo(map.get(key2));
```

请注意，`Byte`对象的`List`将比`byte`原语数组占用更多的内存。因此，尽管这种解决方案很方便，但对于大多数情况来说并不可行。

## 4.实现自定义容器

我们还可以实现自己的包装器来完全控制哈希代码的计算和相等性。这样，我们可以确保解决方案速度快，并且不会占用太多内存。

让我们创建一个具有最后一个私有数组字段的类。它没有 setter，它的 getter 将创建一个防御性副本以确保完全不变性:

```java
public final class BytesKey {
    private final byte[] array;

    public BytesKey(byte[] array) {
        this.array = array;
    }

    public byte[] getArray() {
        return array.clone();
    }
}
```

我们还需要实现自己的`equals`和`hashCode`方法。幸运的是，我们可以使用`Arrays`实用程序类来完成这两项任务:

```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;
    BytesKey bytesKey = (BytesKey) o;
    return Arrays.equals(array, bytesKey.array);
}

@Override
public int hashCode() {
    return Arrays.hashCode(array);
}
```

最后，我们可以将我们的包装器用作一个`HashMap`中的键:

```java
BytesKey key1 = new BytesKey(new byte[]{1, 2, 3});
BytesKey key2 = new BytesKey(new byte[]{1, 2, 3});
Map<BytesKey, String> map = new HashMap<>();
map.put(key1, "value1");
map.put(key2, "value2");
```

然后，我们可以使用任何一个声明的键或者使用一个动态创建的键来检索第二个值:

```java
String retrievedValue1 = map.get(key1);
String retrievedValue2 = map.get(key2);
String retrievedValue3 = map.get(new BytesKey(new byte[]{1, 2, 3}));

assertThat(retrievedValue1).isEqualTo("value2");
assertThat(retrievedValue2).isEqualTo("value2");
assertThat(retrievedValue3).isEqualTo("value2");
```

## 5.结论

在本教程中，我们研究了使用`byte`数组作为`HashMap`中的键的不同问题和解决方案。首先，我们研究了为什么不能使用数组作为键。然后，我们使用一些内置容器来缓解这个问题，最后，实现我们自己的包装器。

像往常一样，本教程的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221126220806/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-maps-5)