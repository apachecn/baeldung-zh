# 什么是 serialVersionUID？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-serial-version-uid>

## 1。概述

`serialVersionUID`属性是用于序列化/反序列化`[Serializable](https://web.archive.org/web/20230101050541/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/Serializable.html)`类的对象的标识符。

在这个快速教程中，我们将讨论什么是`serialVersionUID`以及如何通过例子来使用它。

## 2。串行版本 UID

**简单地说，我们使用`serialVersionUID`属性来记住`Serializable`类的版本，以验证一个加载的类和序列化的对象是兼容的。**

不同类的`serialVersionUID`属性是独立的。因此，不同的类没有必要具有唯一的值。

接下来，我们通过一些例子来学习如何使用`serialVersionUID`。

让我们从创建一个可序列化的类并声明一个`serialVersionUID`标识符开始:

```java
public class AppleProduct implements Serializable {

    private static final long serialVersionUID = 1234567L;

    public String headphonePort;
    public String thunderboltPort;
}
```

接下来，我们需要两个实用程序类:一个将一个`AppleProduct`对象序列化为一个`String,`，另一个从那个`String:`反序列化对象

```java
public class SerializationUtility {

    public static void main(String[] args) {
        AppleProduct macBook = new AppleProduct();
        macBook.headphonePort = "headphonePort2020";
        macBook.thunderboltPort = "thunderboltPort2020";

        String serializedObj = serializeObjectToString(macBook);

        System.out.println("Serialized AppleProduct object to string:");
        System.out.println(serializedObj);
    }

    public static String serializeObjectToString(Serializable o) {
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        ObjectOutputStream oos = new ObjectOutputStream(baos);
        oos.writeObject(o);
        oos.close();

        return Base64.getEncoder().encodeToString(baos.toByteArray());
    }
}
```

```java
public class DeserializationUtility {

    public static void main(String[] args) {

        String serializedObj = ... // ommited for clarity
        System.out.println(
          "Deserializing AppleProduct...");

        AppleProduct deserializedObj = (AppleProduct) deSerializeObjectFromString(
          serializedObj);

        System.out.println(
          "Headphone port of AppleProduct:"
            + deserializedObj.getHeadphonePort());
        System.out.println(
          "Thunderbolt port of AppleProduct:"
           + deserializedObj.getThunderboltPort());
    }

    public static Object deSerializeObjectFromString(String s)
      throws IOException, ClassNotFoundException {

        byte[] data = Base64.getDecoder().decode(s);
        ObjectInputStream ois = new ObjectInputStream(
          new ByteArrayInputStream(data));
        Object o = ois.readObject();
        ois.close();
        return o;
    }
}
```

我们从运行`SerializationUtility.java`开始，它将`AppleProduct`对象保存(序列化)到一个使用 [Base64](https://web.archive.org/web/20230101050541/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Base64.html) 编码字节的`String`实例`e,`中。

然后，使用那个`String`作为反序列化方法的参数，我们运行`DeserializationUtility.java,`，它从给定的`String.`重新组装(反序列化)了`AppleProduct`对象

生成的输出应该如下所示:

```java
Serialized AppleProduct object to string:
rO0ABXNyACljb20uYmFlbGR1bmcuZGVzZXJpYWxpemF0aW9uLkFwcGxlUHJvZHVjdAAAAAAAEta
HAgADTAANaGVhZHBob25lUG9ydHQAEkxqYXZhL2xhbmcvU3RyaW5nO0wADmxpZ2h0ZW5pbmdQb3
J0cQB+AAFMAA90aHVuZGVyYm9sdFBvcnRxAH4AAXhwdAARaGVhZHBob25lUG9ydDIwMjBwdAATd
Gh1bmRlcmJvbHRQb3J0MjAyMA==
```

```java
Deserializing AppleProduct...
Headphone port of AppleProduct:headphonePort2020
Thunderbolt port of AppleProduct:thunderboltPort2020
```

**现在，让我们修改`AppleProduct.java,`中的`serialVersionUID`** **常量，并重新尝试反序列化**先前生成的同一字符串中的`AppleProduct`对象。重新运行`DeserializationUtility.java`应该会产生这个输出。

```java
Deserializing AppleProduct...
Exception in thread "main" java.io.InvalidClassException: com.baeldung.deserialization.AppleProduct; local class incompatible: stream classdesc serialVersionUID = 1234567, local class serialVersionUID = 7654321
	at java.io.ObjectStreamClass.initNonProxy(ObjectStreamClass.java:616)
	at java.io.ObjectInputStream.readNonProxyDesc(ObjectInputStream.java:1630)
	at java.io.ObjectInputStream.readClassDesc(ObjectInputStream.java:1521)
	at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:1781)
	at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1353)
	at java.io.ObjectInputStream.readObject(ObjectInputStream.java:373)
	at com.baeldung.deserialization.DeserializationUtility.deSerializeObjectFromString(DeserializationUtility.java:24)
	at com.baeldung.deserialization.DeserializationUtility.main(DeserializationUtility.java:15)
```

通过改变类的`serialVersionUID`，我们修改了它的版本/状态。结果，在反序列化期间没有找到兼容的类，并且抛出了一个`**InvalidClassException**`。

如果`Serializable`类中没有提供`serialVersionUID`，JVM 将自动生成一个。然而，**提供`serialVersionUID`值并在类发生变化后更新它是一个好习惯，这样我们就可以控制序列化/反序列化过程**。我们将在后面的部分中仔细研究它。

## 3。兼容变更

假设我们需要在现有的`AppleProduct`类中添加一个新字段`lightningPort`:

```java
public class AppleProduct implements Serializable {
//...
    public String lightningPort;
}
```

由于我们只是添加了一个新字段，**不需要对`serialVersionUID` 做任何更改**。这是因为，**在反序列化过程中，`null`将被指定为`lightningPort`字段**的默认值。

让我们修改我们的`DeserializationUtility`类来打印这个新字段的值:

```java
System.out.println("LightningPort port of AppleProduct:"
  + deserializedObj.getLightningPort());
```

现在，当我们重新运行`DeserializationUtility`类时，我们将看到类似如下的输出:

```java
Deserializing AppleProduct...
Headphone port of AppleProduct:headphonePort2020
Thunderbolt port of AppleProduct:thunderboltPort2020
Lightning port of AppleProduct:null
```

## 4.默认串行版本

**如果我们没有为一个`Serializable `类定义一个`serialVersionUID `状态，那么 Java 会根据类本身的一些属性定义一个状态，比如类名、实例字段等等。**

让我们定义一个简单的`Serializable `类:

```java
public class DefaultSerial implements Serializable {
}
```

如果我们序列化此类的实例，如下所示:

```java
DefaultSerial instance = new DefaultSerial();
System.out.println(SerializationUtility.serializeObjectToString(instance));
```

这将打印序列化二进制文件的 Base64 摘要:

```java
rO0ABXNyACpjb20uYmFlbGR1bmcuZGVzZXJpYWxpemF0aW9uLkRlZmF1bHRTZXJpYWx9iVz3Lz/mdAIAAHhw
```

就像之前一样，我们应该能够从摘要中反序列化这个实例:

```java
String digest = "rO0ABXNyACpjb20uYmFlbGR1bmcuZGVzZXJpY" 
  + "WxpemF0aW9uLkRlZmF1bHRTZXJpYWx9iVz3Lz/mdAIAAHhw";
DefaultSerial instance = (DefaultSerial) DeserializationUtility.deSerializeObjectFromString(digest);
```

**但是，对此类的一些更改可能会破坏序列化兼容性。**例如，如果我们给这个类添加一个`private `字段:

```java
public class DefaultSerial implements Serializable {
    private String name;
}
```

然后尝试将相同的 Base64 摘要反序列化为一个类实例，我们将得到一个`InvalidClassException:`

```java
Exception in thread "main" java.io.InvalidClassException: 
  com.baeldung.deserialization.DefaultSerial; local class incompatible: 
  stream classdesc serialVersionUID = 9045863543269746292, 
  local class serialVersionUID = -2692722436255640434
```

**由于这种不想要的不兼容性，在`Serializable`类中声明一个`serialVersionUID `总是一个好主意。**这样我们可以随着类本身的发展保持或发展版本。

## 5。结论

在这篇简短的文章中，我们演示了如何使用`serialVersionUID`常量来简化序列化数据的版本控制。

和往常一样，本文中使用的代码示例可以在 GitHub 上找到[。](https://web.archive.org/web/20230101050541/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-serialization/)