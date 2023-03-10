# Java 序列化简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-serialization>

## 1。简介

序列化是将对象的状态转换成字节流；反序列化则相反。换句话说，序列化是将 Java 对象转换成静态字节流(序列)，然后我们可以将它保存到数据库或通过网络传输。

## 2.序列化和反序列化

序列化过程是独立于实例的；例如，我们可以在一个平台上序列化对象，在另一个平台上反序列化它们。**适合序列化的类需要实现一个特殊的标记接口，** `Serializable. `

`ObjectInputStream`和`ObjectOutputStream`都是高级类，分别扩展了`java.io.InputStream`和`java.io.OutputStream,`。`ObjectOutputStream`可以将对象的基本类型和图形作为字节流写入`OutputStream` 。然后我们可以使用`ObjectInputStream`来读取这些流。

`ObjectOutputStream`中最重要的方法是:

```java
public final void writeObject(Object o) throws IOException;
```

此方法接受一个可序列化的对象，并将其转换为字节序列(流)。同样，`ObjectInputStream`中最重要的方法是:

```java
public final Object readObject() 
  throws IOException, ClassNotFoundException;
```

该方法可以读取字节流，并将其转换回 Java 对象。然后可以将它强制转换回原始对象。

让我们用一个`Person`类来说明序列化。注意**静态字段属于一个类(相对于一个对象)，并且没有被序列化**。另外，请注意，我们可以使用关键字`transient`在序列化过程中忽略类字段:

```java
public class Person implements Serializable {
    private static final long serialVersionUID = 1L;
    static String country = "ITALY";
    private int age;
    private String name;
    transient int height;

    // getters and setters
}
```

下面的测试显示了一个将类型为`Person`的对象保存到本地文件，然后将值读回的例子:

```java
@Test 
public void whenSerializingAndDeserializing_ThenObjectIsTheSame() () 
  throws IOException, ClassNotFoundException { 
    Person person = new Person();
    person.setAge(20);
    person.setName("Joe");

    FileOutputStream fileOutputStream
      = new FileOutputStream("yourfile.txt");
    ObjectOutputStream objectOutputStream 
      = new ObjectOutputStream(fileOutputStream);
    objectOutputStream.writeObject(person);
    objectOutputStream.flush();
    objectOutputStream.close();

    FileInputStream fileInputStream
      = new FileInputStream("yourfile.txt");
    ObjectInputStream objectInputStream
      = new ObjectInputStream(fileInputStream);
    Person p2 = (Person) objectInputStream.readObject();
    objectInputStream.close(); 

    assertTrue(p2.getAge() == person.getAge());
    assertTrue(p2.getName().equals(person.getName()));
}
```

我们使用`ObjectOutputStream`通过`FileOutputStream`将这个对象的状态保存到一个文件中。文件`“yourfile.txt”`被创建在项目目录中。然后使用`FileInputStream.`加载这个文件，T4 拾取这个流并将其转换成一个名为`p2`的新对象。

最后，我们将测试加载对象的状态，并确保它与原始对象的状态相匹配。

注意，我们必须显式地将加载的对象转换成一个`Person`类型。

## 3。Java 序列化警告

关于 Java 中的序列化有一些注意事项。

### 3.1。继承和组合

当一个类实现了 `java.io.Serializable`接口时，它的所有子类也是可序列化的。相反，当一个对象引用另一个对象时，这些对象必须单独实现`Serializable`接口，否则将抛出 *NotSerializableException* :

```java
public class Person implements Serializable {
    private int age;
    private String name;
    private Address country; // must be serializable too
} 
```

如果可序列化对象中的一个字段由对象数组组成，那么所有这些对象也必须是可序列化的，否则将抛出一个 *NotSerializableException* 。

### 3.2。串行版本 UID

**JVM 将版本号(`long`)与每个可序列化的类相关联。**我们使用它来验证保存和加载的对象具有相同的属性，因此在序列化上是兼容的。

大多数 ide 可以自动生成这个数字，它基于类名、属性和相关的访问修饰符。任何改变都会导致不同的数字，并可能导致`InvalidClassException`。

如果一个可序列化的类没有声明一个`serialVersionUID`，JVM 将在运行时自动生成一个。然而，强烈建议每个类声明它的`serialVersionUID,`,因为生成的类依赖于编译器，因此可能会导致意外的`InvalidClassExceptions`。

### 3.3。Java 中的自定义序列化

Java 指定了序列化对象的默认方式，但是 Java 类可以覆盖这种默认行为。当尝试序列化具有某些不可序列化属性的对象时，自定义序列化可能特别有用。我们可以通过在我们想要序列化的类中提供两个方法来做到这一点:

```java
private void writeObject(ObjectOutputStream out) throws IOException;
```

和

```java
private void readObject(ObjectInputStream in) 
  throws IOException, ClassNotFoundException;
```

使用这些方法，我们可以将不可序列化的属性序列化为其他可以序列化的形式:

```java
public class Employee implements Serializable {
    private static final long serialVersionUID = 1L;
    private transient Address address;
    private Person person;

    // setters and getters

    private void writeObject(ObjectOutputStream oos) 
      throws IOException {
        oos.defaultWriteObject();
        oos.writeObject(address.getHouseNumber());
    }

    private void readObject(ObjectInputStream ois) 
      throws ClassNotFoundException, IOException {
        ois.defaultReadObject();
        Integer houseNumber = (Integer) ois.readObject();
        Address a = new Address();
        a.setHouseNumber(houseNumber);
        this.setAddress(a);
    }
}
```

```java
public class Address {
    private int houseNumber;

    // setters and getters
}
```

我们可以运行以下单元测试来测试这种自定义序列化:

```java
@Test
public void whenCustomSerializingAndDeserializing_ThenObjectIsTheSame() 
  throws IOException, ClassNotFoundException {
    Person p = new Person();
    p.setAge(20);
    p.setName("Joe");

    Address a = new Address();
    a.setHouseNumber(1);

    Employee e = new Employee();
    e.setPerson(p);
    e.setAddress(a);

    FileOutputStream fileOutputStream
      = new FileOutputStream("yourfile2.txt");
    ObjectOutputStream objectOutputStream 
      = new ObjectOutputStream(fileOutputStream);
    objectOutputStream.writeObject(e);
    objectOutputStream.flush();
    objectOutputStream.close();

    FileInputStream fileInputStream 
      = new FileInputStream("yourfile2.txt");
    ObjectInputStream objectInputStream 
      = new ObjectInputStream(fileInputStream);
    Employee e2 = (Employee) objectInputStream.readObject();
    objectInputStream.close();

    assertTrue(
      e2.getPerson().getAge() == e.getPerson().getAge());
    assertTrue(
      e2.getAddress().getHouseNumber() == e.getAddress().getHouseNumber());
}
```

在这段代码中，我们可以看到如何通过使用自定义序列化来序列化`Address`来保存一些不可序列化的属性。注意，我们必须将不可序列化的属性标记为`transient`以避免 *NotSerializableException。*

## 4。结论

在这篇简短的文章中，我们回顾了 Java 序列化，讨论了注意事项，并学习了如何进行自定义序列化。

和往常一样，本文中使用的源代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20221104165328/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-serialization)