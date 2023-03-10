# Java 中的序列化验证

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-validate-serializable>

## 1.概观

在这个快速教程中，**我们将演示如何在 Java** 中验证一个可序列化的对象。

## 2.序列化和反序列化

[序列化](/web/20221208143921/https://www.baeldung.com/java-serialization)是**将对象的状态转换成字节流**的过程。序列化对象主要用于 Hibernate、RMI、JPA、EJB 和 JMS 技术中的。

转换方向，反序列化是使用字节流在内存中重新创建实际 Java 对象的反向过程。 这个过程常用来持久化对象。

## 3.序列化验证

我们可以使用多种方法来验证序列化。下面我们来看几个。

### 3.1.验证`implements`序列化

确定一个对象是否可序列化的最简单的方法是用**检查该对象是否是`java.io.Serializable`或`java.io.Externalizable`T3 的实例。然而，这个方法不能保证我们可以序列化一个对象。**

假设我们有一个没有实现`Serializable`接口的`Address`对象:

```java
public class Address {
    private int houseNumber;

    //getters and setters
}
```

尝试序列化一个`Address`对象时，可能会出现一个`NotSerializableException`:

```java
@Test(expected = NotSerializableException.class)
public void whenSerializing_ThenThrowsError() throws IOException {
    Address address = new Address();
    address.setHouseNumber(10);
    FileOutputStream fileOutputStream = new FileOutputStream("yofile.txt");
    try (ObjectOutputStream objectOutputStream = new ObjectOutputStream(fileOutputStream)) {
        objectOutputStream.writeObject(address);
    }
}
```

现在，假设我们有一个实现了`Serializable`接口的`Person`对象:

```java
public class Person implements Serializable {
    private static final long serialVersionUID = 1L;
    private int age;
    private String name;

    // getters and setters
}
```

在这种情况下，我们将能够序列化和反序列化以重新创建对象:

```java
Person p = new Person();
p.setAge(20);
p.setName("Joe");
FileOutputStream fileOutputStream = new FileOutputStream("yofile.txt");
try (ObjectOutputStream objectOutputStream = new ObjectOutputStream(fileOutputStream)) {
    objectOutputStream.writeObject(p);
}

FileInputStream fileInputStream = new FileInputStream("yofile.txt");
try ( ObjectInputStream objectInputStream = new ObjectInputStream(fileInputStream)) {
    Person p2 = (Person) objectInputStream.readObject();
    assertEquals(p2.getAge(), p.getAge());
    assertEquals(p2.getName(), p.getName());;
}
```

### 3.2 .Apache common〔t0〕

另一种验证对象序列化的方法是利用 Apache Commons [`SerializationUtils`](https://web.archive.org/web/20221208143921/https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/SerializationUtils.html) 中的`serialize`方法。此方法不接受不可序列化的对象。

如果我们试图通过显式类型转换来编译代码，从而序列化不可序列化的`Address`对象，会怎么样？在`runtime`，我们会遇到一个`ClassCastException`:

```java
Address address = new Address();
address.setHouseNumber(10);
SerializationUtils.serialize((Serializable) address);
```

让我们使用上面的方法来验证可序列化的`Person` 对象:

```java
Person p = new Person();
p.setAge(20);
p.setName("Joe");
byte[] serialize = SerializationUtils.serialize(p);
Person p2 = (Person)SerializationUtils.deserialize(serialize);
assertEquals(p2.getAge(), p.getAge());
assertEquals(p2.getName(), p.getName());
```

### 3.3.弹簧芯`SerializationUtils`

我们现在来看看来自 [spring-core](https://web.archive.org/web/20221208143921/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework%22%20AND%20a%3A%22spring-core%22) 的 [`SerializationUtils`](https://web.archive.org/web/20221208143921/https://www.javadoc.io/doc/org.springframework/spring-core/5.0.8.RELEASE/org/springframework/util/SerializationUtils.html) 方法，它类似于来自 Apache Commons 的方法。这个方法也不接受不可序列化的`Address` 对象。

这样的代码将在运行时抛出一个`ClassCastException` :

```java
Address address = new Address();
address.setHouseNumber(10);
org.springframework.util.SerializationUtils.serialize((Serializable) address);
```

让我们尝试使用可序列化的`Person`对象:

```java
Person p = new Person();
p.setAge(20);
p.setName("Joe");
byte[] serialize = org.springframework.util.SerializationUtils.serialize(p);
Person p2 = (Person)org.springframework.util.SerializationUtils.deserialize(serialize);
assertEquals(p2.getAge(), p.getAge());
assertEquals(p2.getName(), p.getName());
```

### 3.4.自定义序列化实用程序

作为第三种选择，我们将创建自己的定制实用程序，根据我们的需求进行序列化或反序列化。为了演示这一点，我们将为序列化和反序列化编写两个独立的方法。

第一个是序列化过程的对象验证示例:

```java
public static  byte[] serialize(T obj) throws IOException {
    ByteArrayOutputStream baos = new ByteArrayOutputStream();
    ObjectOutputStream oos = new ObjectOutputStream(baos);
    oos.writeObject(obj);
    oos.close();
    return baos.toByteArray();
}
```

我们还将编写一个方法来执行反序列化过程:

```java
public static  T deserialize(byte[] b, Class cl) throws IOException, ClassNotFoundException {
    ByteArrayInputStream bais = new ByteArrayInputStream(b);
    ObjectInputStream ois = new ObjectInputStream(bais);
    Object o = ois.readObject();
    return cl.cast(o);
}
```

此外，我们可以创建一个实用方法，该方法将`Class`作为参数，如果对象是可序列化的，则返回`true`。该方法假设原语和接口是隐式可序列化的，同时验证输入类是否可以分配给`Serializable`。此外，我们在验证过程中排除了`transient`和`static`字段。

让我们实现这个方法:

```java
public static boolean isSerializable(Class<?> it) {
    boolean serializable = it.isPrimitive() || it.isInterface() || Serializable.class.isAssignableFrom(it);
    if (!serializable) {
        return false;
    }
    Field[] declaredFields = it.getDeclaredFields();
    for (Field field : declaredFields) {
        if (Modifier.isVolatile(field.getModifiers()) || Modifier.isTransient(field.getModifiers()) || 
          Modifier.isStatic(field.getModifiers())) {
            continue;
        }
        Class<?> fieldType = field.getType();
        if (!isSerializable(fieldType)) {
            return false;
        }
    }
    return true;
}
```

现在让我们验证我们的实用方法:

```java
assertFalse(MySerializationUtils.isSerializable(Address.class));
assertTrue(MySerializationUtils.isSerializable(Person.class));
assertTrue(MySerializationUtils.isSerializable(Integer.class));
```

## 4.结论

在本文中，我们研究了几种确定对象是否可序列化的方法。我们还演示了一个自定义实现来完成同样的任务。

按照惯例，本教程中使用的所有代码示例都可以从 GitHub 上的[处获得。](https://web.archive.org/web/20221208143921/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-serialization)