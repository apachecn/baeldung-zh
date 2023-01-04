# Java 可外部化接口指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-externalizable>

## 1。简介

在本教程中，**我们将快速浏览一下 java 的`java.io.Externalizable`接口**。该接口的主要目标是促进自定义序列化和反序列化。

在我们继续之前，请确保您阅读了 Java 文章中的[序列化。下一章是关于如何用这个接口序列化一个 Java 对象。](/web/20221208143839/https://www.baeldung.com/java-serialization)

之后，我们将讨论与`java.io.Serializable`界面相比的主要区别。

## 2。 **`Externalizable`界面**

`Externalizable`从`java.io.Serializable`标记接口延伸。任何实现`Externalizable` 接口的类都应该覆盖`writeExternal()`、`readExternal()`方法。这样我们就可以改变 JVM 的默认序列化行为。

### 2.1。序列化

让我们来看看这个简单的例子:

```java
public class Country implements Externalizable {

    private static final long serialVersionUID = 1L;

    private String name;
    private int code;

    // getters, setters

    @Override
    public void writeExternal(ObjectOutput out) throws IOException {
        out.writeUTF(name);
        out.writeInt(code);
    }

    @Override
    public void readExternal(ObjectInput in) 
      throws IOException, ClassNotFoundException {
        this.name = in.readUTF();
        this.code = in.readInt();
    }
}
```

这里，我们定义了一个类`Country`，它实现了`Externalizable`接口并实现了上面提到的两个方法。

**在`writeExternal()`方法中，我们将对象的属性添加到`ObjectOutput`流中。这有标准的方法，比如用`writeUTF()`来表示`String`，用`writeInt()`来表示 int 值。**

接下来，**为了反序列化对象，我们使用`readUTF(), readInt()`方法从`ObjectInput`流**中读取属性，读取顺序与写入顺序完全相同。

手动添加`serialVersionUID`是一个很好的做法。如果没有，JVM 将自动添加一个。

自动生成的数字取决于编译器。这意味着它可能会导致一个不太可能的`InvalidClassException`。

让我们测试一下我们在上面实现的行为:

```java
@Test
public void whenSerializing_thenUseExternalizable() 
  throws IOException, ClassNotFoundException {

    Country c = new Country();
    c.setCode(374);
    c.setName("Armenia");

    FileOutputStream fileOutputStream
     = new FileOutputStream(OUTPUT_FILE);
    ObjectOutputStream objectOutputStream
     = new ObjectOutputStream(fileOutputStream);
    c.writeExternal(objectOutputStream);

    objectOutputStream.flush();
    objectOutputStream.close();
    fileOutputStream.close();

    FileInputStream fileInputStream
     = new FileInputStream(OUTPUT_FILE);
    ObjectInputStream objectInputStream
     = new ObjectInputStream(fileInputStream);

    Country c2 = new Country();
    c2.readExternal(objectInputStream);

    objectInputStream.close();
    fileInputStream.close();

    assertTrue(c2.getCode() == c.getCode());
    assertTrue(c2.getName().equals(c.getName()));
}
```

在这个例子中，我们首先创建一个`Country`对象并将其写入一个文件。然后，我们从文件中反序列化对象，并验证值是否正确。

打印的`c2`对象的输出:

```java
Country{name='Armenia', code=374}
```

这表明我们已经成功地反序列化了对象。

### 2.2。继承

当一个类从`Serializable` 接口继承时，JVM 也自动从子类中收集所有字段，并使它们可序列化。

请记住，我们也可以将此应用于`Externalizable`。**我们只需要为继承层次的每个子类实现读/写方法。**

让我们看看下面的`Region`类，它扩展了上一节中的`Country`类:

```java
public class Region extends Country implements Externalizable {

    private static final long serialVersionUID = 1L;

    private String climate;
    private Double population;

    // getters, setters

    @Override
    public void writeExternal(ObjectOutput out) throws IOException {
        super.writeExternal(out);
        out.writeUTF(climate);
    }

    @Override
    public void readExternal(ObjectInput in) 
      throws IOException, ClassNotFoundException {

        super.readExternal(in);
        this.climate = in.readUTF();
    }
}
```

这里，我们添加了两个附加属性，并序列化了第一个属性。

注意，**我们还在序列化器方法中调用了`super.writeExternal(out), super.readExternal(in)`来保存/恢复父类字段**。

让我们用以下数据运行单元测试:

```java
Region r = new Region();
r.setCode(374);
r.setName("Armenia");
r.setClimate("Mediterranean");
r.setPopulation(120.000);
```

下面是反序列化的对象:

```java
Region{
  country='Country{
    name='Armenia',
    code=374}'
  climate='Mediterranean', 
  population=null
}
```

注意，**因为我们没有序列化`Region`类中的`population` 字段，所以该属性的值是`null.`**

## 3。 **`Externalizable` vs `Serializable`**

让我们来看看这两种界面之间的主要区别:

*   **序列化责任**

这里的关键区别是我们如何处理序列化过程。当一个类实现了`java.io.Serializable`接口时，JVM 承担序列化类实例的全部责任。**在`Externalizable,`的情况下，程序员应该负责整个序列化和反序列化过程。**

*   **用例**

如果我们需要序列化整个对象，`Serializable`接口是更好的选择。另一方面，**对于自定义序列化，我们可以使用`Externalizable`** 来控制进程。

*   **性能**

`java.io.Serializable`接口使用反射和元数据，这会导致相对较慢的性能。相比之下， **`Externalizable`接口让您可以完全控制序列化过程。**

*   **阅读顺序**

**使用`Externalizable`时，必须按照写入的顺序读取所有字段状态。**否则，我们会得到一个异常。

例如，如果我们改变了`Country`类中`code`和`name`属性的读取顺序，就会抛出一个`java.io.EOFException`。

同时，`Serializable`接口没有这个要求。

*   **自定义序列化**

我们可以通过用`transient`关键字标记字段，用`Serializable`接口实现自定义序列化。**JVM 不会序列化特定的字段，但是它会用默认值**将字段添加到文件存储中。这就是为什么在自定义序列化的情况下使用`Externalizable`是一个好的实践。

## 4。结论

在这个关于`Externalizable`界面的简短指南中，我们讨论了主要特性、优点和简单使用的示例。我们也和`Serializable`接口做了对比。

像往常一样，该教程的完整源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221208143839/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-serialization)