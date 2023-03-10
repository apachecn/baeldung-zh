# Kryo 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/kryo>

## 1。概述

Kryo 是一个专注于速度、效率和用户友好的 API 的 Java 序列化框架。

在本文中，我们将探索 Kryo 框架的关键特性，并实现示例来展示其功能。

## 2。Maven 依赖关系

我们需要做的第一件事是将`kryo`依赖项添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>com.esotericsoftware</groupId>
    <artifactId>kryo</artifactId>
    <version>4.0.1</version>
</dependency>
```

这个神器的最新版本可以在 [Maven Central](https://web.archive.org/web/20220824142037/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.esotericsoftware%22%20AND%20a%3A%22kryo%22) 上找到。

## 3 .低温基础

让我们先来看看 Kryo 是如何工作的，以及我们如何用它来序列化和反序列化对象。

### 3.1。简介

框架提供了`Kryo`类作为其所有功能的主要入口点。

这个类编排序列化过程，并将类映射到`Serializer` 实例，这些实例处理将对象的图形转换成字节表示的细节。

一旦字节准备好了，就用一个`Output`对象把它们写到一个流中。这样，它们可以存储在文件、数据库中或通过网络传输。

稍后，当需要对象时，使用一个`Input`实例来读取这些字节，并将它们解码成 Java 对象。

### 3.2。序列化对象

在深入示例之前，让我们首先创建一个实用方法来初始化一些变量，我们将在本文的每个测试用例中使用这些变量:

```java
@Before
public void init() {
    kryo = new Kryo();
    output = new Output(new FileOutputStream("file.dat"));
    input = new Input(new FileInputStream("file.dat"));
}
```

现在，我们可以看看使用 Kryo 读写一个对象是多么容易:

```java
@Test
public void givenObject_whenSerializing_thenReadCorrectly() {
    Object someObject = "Some string";

    kryo.writeClassAndObject(output, someObject);
    output.close();

    Object theObject = kryo.readClassAndObject(input);
    input.close();

    assertEquals(theObject, "Some string");
}
```

注意对`close()`方法的调用。这是必需的，因为`Output`和`Input`类分别继承自`OutputStream`和`InputStream`。

序列化多个对象同样简单:

```java
@Test
public void givenObjects_whenSerializing_thenReadCorrectly() {
    String someString = "Multiple Objects";
    Date someDate = new Date(915170400000L);

    kryo.writeObject(output, someString);
    kryo.writeObject(output, someDate);
    output.close();

    String readString = kryo.readObject(input, String.class);
    Date readDate = kryo.readObject(input, Date.class);
    input.close();

    assertEquals(readString, "Multiple Objects");
    assertEquals(readDate.getTime(), 915170400000L);
}
```

注意，我们将适当的类传递给了`readObject()`方法，这使得我们的代码是无强制转换的。

## 4。串行器

在这一节中，我们将展示哪些`Serializers`已经可用，然后我们将创建自己的。

### 4.1。默认序列化程序

当 Kryo 序列化一个对象时，它会创建一个先前注册的`Serializer`类的实例来完成到字节的转换。这些被称为默认序列化器，可以在我们不做任何设置的情况下使用。

该库已经提供了几个处理原语、列表、映射、枚举等的序列化程序。如果没有为给定的类找到序列化程序，则使用`FieldSerializer`来处理几乎任何类型的对象。

让我们看看这个是什么样子的。首先，让我们创建一个`Person`类:

```java
public class Person {
    private String name = "John Doe";
    private int age = 18;
    private Date birthDate = new Date(933191282821L);

    // standard constructors, getters, and setters
}
```

现在，让我们从这个类中编写一个对象，然后读回它:

```java
@Test
public void givenPerson_whenSerializing_thenReadCorrectly() {
    Person person = new Person();

    kryo.writeObject(output, person);
    output.close();

    Person readPerson = kryo.readObject(input, Person.class);
    input.close();

    assertEquals(readPerson.getName(), "John Doe");
}
```

注意，我们不需要指定任何东西来序列化一个`Person`对象，因为已经为我们自动创建了一个`FieldSerializer`。

### 4.2。自定义序列化程序

如果我们需要对序列化过程进行更多的控制，我们有两个选择；我们可以编写自己的`Serializer`类并向 Kryo 注册，或者让类自己处理序列化。

为了演示第一个选项，让我们创建一个扩展`Serializer`的类:

```java
public class PersonSerializer extends Serializer<Person> {

    public void write(Kryo kryo, Output output, Person object) {
        output.writeString(object.getName());
        output.writeLong(object.getBirthDate().getTime());
    }

    public Person read(Kryo kryo, Input input, Class<Person> type) {
        Person person = new Person();
        person.setName(input.readString());
        long birthDate = input.readLong();
        person.setBirthDate(new Date(birthDate));
        person.setAge(calculateAge(birthDate));
        return person;
    }

    private int calculateAge(long birthDate) {
        // Some custom logic
        return 18;
    }
}
```

现在，让我们来测试一下:

```java
@Test
public void givenPerson_whenUsingCustomSerializer_thenReadCorrectly() {
    Person person = new Person();
    person.setAge(0);

    kryo.register(Person.class, new PersonSerializer());
    kryo.writeObject(output, person);
    output.close();

    Person readPerson = kryo.readObject(input, Person.class);
    input.close();

    assertEquals(readPerson.getName(), "John Doe");
    assertEquals(readPerson.getAge(), 18);
}
```

请注意，`age`字段等于 18，即使我们之前将其设置为 0。

我们还可以使用`@DefaultSerializer`注释来让 Kryo 知道，每当它需要处理一个`Person`对象时，我们都要使用`PersonSerializer`。这有助于避免调用`register()`方法:

```java
@DefaultSerializer(PersonSerializer.class)
public class Person implements KryoSerializable {
    // ...
}
```

对于第二个选项，让我们修改我们的`Person`类来扩展`KryoSerializable`接口:

```java
public class Person implements KryoSerializable {
    // ...

    public void write(Kryo kryo, Output output) {
        output.writeString(name);
        // ...
    }

    public void read(Kryo kryo, Input input) {
        name = input.readString();
        // ...
    }
}
```

由于该选项的测试用例与前一个相同，因此这里不包括。但是，您可以在本文的源代码中找到它。

### 4.3。Java 串行器

在个别情况下，Kryo 不能序列化一个类。如果发生这种情况，并且不能编写定制的序列化程序，我们可以使用标准的 Java 序列化机制，使用一个`JavaSerializer`。这要求该类照常实现`Serializable`接口。

下面是一个使用上述序列化程序的示例:

```java
public class ComplexObject implements Serializable {
    private String name = "Bael";

    // standard getters and setters
}
```

```java
@Test
public void givenJavaSerializable_whenSerializing_thenReadCorrectly() {
    ComplexClass complexObject = new ComplexClass();
    kryo.register(ComplexClass.class, new JavaSerializer());

    kryo.writeObject(output, complexObject);
    output.close();

    ComplexClass readComplexObject = kryo.readObject(input, ComplexClass.class);
    input.close();

    assertEquals(readComplexObject.getName(), "Bael");
}
```

## 5。结论

在本教程中，我们探索了 Kryo 库最显著的特性。

我们序列化了多个简单对象，并使用`FieldSerializer`类来处理一个自定义对象。我们还创建了一个定制的序列化器，并演示了如何在需要时退回到标准的 Java 序列化机制。

和往常一样，本文的完整源代码可以在 Github 上找到[。](https://web.archive.org/web/20220824142037/https://github.com/eugenp/tutorials/tree/master/libraries-data-io)