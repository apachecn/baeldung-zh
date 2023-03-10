# 用 Jackson 反序列化不可变对象

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jackson-deserialize-immutable-objects>

## 1.概观

在这个快速教程中，我们将展示用 [Jackson](/web/20221106121625/https://www.baeldung.com/jackson) JSON 处理库反序列化不可变 Java 对象的两种不同方式。

## 2.为什么我们使用不可变对象？

一个[不可变的对象](/web/20221106121625/https://www.baeldung.com/java-immutable-object)是一个**从它创建的那一刻起就保持其状态不变的对象**。这意味着无论最终用户调用对象的哪个方法，**对象的行为都是一样的**。

当我们设计一个必须在多线程环境中工作的**系统时，不可变对象就派上了用场，因为不变性通常保证线程安全。**

另一方面，当我们需要处理来自外部来源的输入时，不可变对象是有用的。例如，它可以是用户输入或来自存储器的一些数据。在这种情况下，保存接收到的数据并保护其免受意外或无意的更改可能至关重要。

让我们看看如何反序列化一个不可变的对象。

## 3.公共构造函数

让我们考虑一下`Employee`的类结构。它有两个必需的字段:`id`和`name`，因此我们定义了一个**公共全参数构造函数**，它有一组与对象字段集匹配的参数:

```java
public class Employee {

    private final long id;
    private final String name;

    public Employee(long id, String name) {
        this.id = id;
        this.name = name;
    }

    // getters
}
```

这样，我们将在创建时初始化对象的所有字段。字段声明中的最终修饰符不允许我们在将来更改它们的值。要使这个对象可反序列化，我们只需向这个构造函数添加几个[注释](/web/20221106121625/https://www.baeldung.com/jackson-annotations):

```java
@JsonCreator(mode = JsonCreator.Mode.PROPERTIES)
public Employee(@JsonProperty("id") long id, @JsonProperty("name") String name) {
    this.id = id;
    this.name = name;
}
```

让我们仔细看看我们刚刚添加的注释。

首先，`@JsonCreator`告诉 Jackson 反序列化器**使用指定的构造函数进行反序列化**。

有两种模式可用作该注释的参数—`PROPERTIES`和`DELEGATING`。

当我们声明一个全参数构造函数时，`PROPERTIES`是最合适的，而对于单参数构造函数来说，`DELEGATING`可能是有用的。

之后，**我们需要用`@JsonProperty`注释每个构造函数参数**,声明各自属性的名称作为注释值。在这一步我们应该非常小心，因为所有的**属性名必须与我们在序列化过程中使用的**相匹配。

让我们来看一个简单的单元测试，它涵盖了一个`Employee`对象的反序列化:

```java
String json = "{\"name\":\"Frank\",\"id\":5000}";
Employee employee = new ObjectMapper().readValue(json, Employee.class);

assertEquals("Frank", employee.getName());
assertEquals(5000, employee.getId());
```

## 4.私人建造师和营造师

有时，一个对象有一组可选字段。让我们考虑另一个类结构，`Person`，它有一个可选的`age`字段:

```java
public class Person {
    private final String name;
    private final Integer age;

    // getters
}
```

当我们有大量这样的字段时，**创建一个公共构造函数可能会变得很麻烦**。换句话说，我们需要为构造函数声明许多参数，并用`@JsonProperty`注释对每个参数进行注释。因此，许多重复的声明会使我们的代码变得臃肿，难以阅读。

这是一个经典的[构建器模式](/web/20221106121625/https://www.baeldung.com/creational-design-patterns)来拯救的情况。让我们看看如何在反序列化中利用它的力量。首先，让我们声明**一个私有的所有参数构造函数和一个`Builder`类**:

```java
private Person(String name, Integer age) {
    this.name = name;
    this.age = age;
}

static class Builder {
    String name;
    Integer age;

    Builder withName(String name) {
        this.name = name;
        return this;
    }

    Builder withAge(Integer age) {
        this.age = age;
        return this;
    }

    public Person build() {
        return new Person(name, age);
    } 
}
```

为了让 Jackson 反序列化器使用这个`Builder`，我们只需要在代码中添加两个注释。首先，我们需要用`@JsonDeserialize`注释来标记我们的类，用一个构建器类的**全限定域名来传递一个`builder`参数。**

之后，我们需要将构建器类本身注释为`@JsonPOJOBuilder`:

```java
@JsonDeserialize(builder = Person.Builder.class)
public class Person {
    //...

    @JsonPOJOBuilder
    static class Builder {
        //...
    }
}
```

请注意，我们可以自定义构建过程中使用的方法的名称。

参数`buildMethodName`默认为“`build”`，代表当构建器准备生成新对象时我们调用的**方法的名称。**

另一个参数`withPrefix`，代表我们添加到负责设置属性的构建器方法中的**前缀。该参数的默认值为`“with”`。这就是为什么我们在示例中没有指定这些参数。**

让我们来看一个简单的单元测试，它涵盖了一个`Person`对象的反序列化:

```java
String json = "{\"name\":\"Frank\",\"age\":50}";
Person person = new ObjectMapper().readValue(json, Person.class);

assertEquals("Frank", person.getName());
assertEquals(50, person.getAge().intValue());
```

## 5.结论

在这篇短文中，我们看到了如何使用 Jackson 库反序列化不可变对象。

与本文相关的所有代码都可以在 [GitHub](https://web.archive.org/web/20221106121625/https://github.com/eugenp/tutorials/tree/master/jackson-modules/jackson-conversions) 上找到。