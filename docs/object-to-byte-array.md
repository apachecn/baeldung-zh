# 在 Java 中将对象转换成字节数组

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/object-to-byte-array>

## 1.概观

在这个简短的教程中，我们将学习如何**将 Java 对象转换成字节数组，反之亦然**。

## 2.使用普通 Java

例如，假设我们有一个`User`类:

```java
public class User implements Serializable {
    private String name;

    @Override
    public String toString() {
        return "User{name=" + name +  "}";
    }

    // getters and setters
}
```

我们可以使用一个`ByteArrayOutputStream`和`ObjectOutputStream`对象将一个对象序列化为一个字节数组。

让我们不要忘记使用 [try-with-resources](/web/20220524020401/https://www.baeldung.com/java-try-with-resources) ，这样我们就不必担心关闭流:

```java
User user = new User();
user.setName("Josh");
try (ByteArrayOutputStream bos = new ByteArrayOutputStream(); 
     ObjectOutputStream oos = new ObjectOutputStream(bos)) {
    oos.writeObject(user);
}
```

然后，我们将使用`ByteArrayInputStream`和`ObjectInputStream`将接收到的字节数组反序列化为一个对象，最后将其转换为`User`:

```java
try (ByteArrayInputStream bis = new ByteArrayInputStream(data);
     ObjectInputStream ois = new ObjectInputStream(bis)) {
    User deserializedUser = (User) ois.readObject();
    System.out.println(deserializedUser);
}
```

注意**我们的`User`类必须实现`Serializable`接口**。否则，上面的代码会抛出一个`NotSerializableException`。

## 3.使用 Apache Commons Lang

我们可以使用来自 [Apache Commons Lang](/web/20220524020401/https://www.baeldung.com/java-commons-lang-3) 库的`SerializationUtils`类来实现同样的目标。

这个类有一个名为`serialize()`的方法，用于将一个对象序列化为一个字节数组:

```java
byte[] data = SerializationUtils.serialize(user);
```

和一个将字节数组反序列化为对象的`deserialize()`方法:

```java
User deserializedUser = SerializationUtils.deserialize(data);
```

**上面的方法有类型为`Serializable.`** 的参数，所以我们的`User`类仍然需要实现`Serializable`接口，就像在我们普通的 Java 例子中一样。

## 4.使用 Spring 框架的`SerializationUtils`类

最后，如果我们的项目已经在使用 Spring 框架，我们可以使用`org.springframework.util`包中的`SerializationUtils`类。方法名与 Apache Commons Lang 库中的方法名相同。

首先，我们可以将我们的`User`对象序列化为一个字节数组:

```java
byte[] data = SerializationUtils.serialize(user);
```

我们可以将结果反序列化回一个`User`对象:

```java
User deserializedUser = SerializationUtils.deserialize(data);
```

**像往常一样，我们的`User`类必须实现`Serializable`接口**，否则当我们运行上面的代码时就会得到一个`NotSerializableException`。

## 5.结论

总之，我们已经学习了三种不同的方法将 Java 对象转换成字节数组，反之亦然。所有这些方法都需要输入对象**实现`Serializable`接口**来完成工作。