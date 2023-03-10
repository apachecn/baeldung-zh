# 杰克逊的遗产

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jackson-inheritance>

## 1。概述

在这篇文章中，我们将看看如何在 Jackson 中使用类层次结构。

两个典型的用例是包含子类型元数据和忽略从超类继承的属性。我们将描述这两种情况以及需要对亚型进行特殊治疗的几种情况。

## 2。包含子类型信息

在序列化和反序列化数据对象时，有两种方法可以添加类型信息，即全局默认类型和每个类的注释。

### 2.1。全局默认输入

下面的三个 Java 类将用来说明类型元数据的全局包含。

`Vehicle`超类:

```java
public abstract class Vehicle {
    private String make;
    private String model;

    protected Vehicle(String make, String model) {
        this.make = make;
        this.model = model;
    }

    // no-arg constructor, getters and setters
}
```

`Car`子类:

```java
public class Car extends Vehicle {
    private int seatingCapacity;
    private double topSpeed;

    public Car(String make, String model, int seatingCapacity, double topSpeed) {
        super(make, model);
        this.seatingCapacity = seatingCapacity;
        this.topSpeed = topSpeed;
    }

    // no-arg constructor, getters and setters
}
```

`Truck`子类:

```java
public class Truck extends Vehicle {
    private double payloadCapacity;

    public Truck(String make, String model, double payloadCapacity) {
        super(make, model);
        this.payloadCapacity = payloadCapacity;
    }

    // no-arg constructor, getters and setters
}
```

全局默认类型允许类型信息通过在一个`ObjectMapper`对象上启用它来声明一次。然后，该类型元数据将应用于所有指定的类型。因此，使用这种方法添加类型元数据非常方便，尤其是当涉及大量类型时。缺点是它使用完全限定的 Java 类型名作为类型标识符，因此不适合与非 Java 系统交互，并且只适用于几种预定义的类型。

上面显示的`Vehicle`结构用于填充`Fleet`类的一个实例:

```java
public class Fleet {
    private List<Vehicle> vehicles;

    // getters and setters
}
```

为了嵌入类型元数据，我们需要在`ObjectMapper`对象上启用类型功能，该功能稍后将用于数据对象的序列化和反序列化:

```java
ObjectMapper.activateDefaultTyping(PolymorphicTypeValidator ptv, 
  ObjectMapper.DefaultTyping applicability, JsonTypeInfo.As includeAs)
```

参数`PolymorphicTypeValidator`用于根据指定的标准验证要反序列化的实际子类型是否有效。此外，`applicability`参数确定需要类型信息的类型，而`includeAs`参数是类型元数据包含的机制。此外，还提供了`activateDefaultTyping`方法的两种其他变体:

*   `ObjectMapper.activateDefaultTyping(PolymorphicTypeValidator ptv, ObjectMapper.DefaultTyping applicability)`:允许呼叫者指定`validator`和`applicability`，同时使用`WRAPPER_ARRAY` 作为`includeAs`的默认值
*   `ObjectMapper.activateDefaultTyping(PolymorphicTypeValidator ptv):` 允许呼叫者指定`validator`，同时使用`OBJECT_AND_NON_CONCRETE` 作为`applicability`的默认值，使用`WRAPPER_ARRAY` 作为`includeAs`的默认值

让我们看看它是如何工作的。首先，我们需要创建一个验证器:

```java
PolymorphicTypeValidator ptv = BasicPolymorphicTypeValidator.builder()
  .allowIfSubType("com.baeldung.jackson.inheritance")
  .allowIfSubType("java.util.ArrayList")
  .build();
```

接下来，让我们创建一个`ObjectMapper`对象，并使用上面的验证器激活它的默认类型:

```java
ObjectMapper mapper = new ObjectMapper();
mapper.activateDefaultTyping(ptv, ObjectMapper.DefaultTyping.NON_FINAL);
```

下一步是实例化和填充本小节开头介绍的数据结构。完成这项工作的代码将在后面的小节中重用。为了方便和重用，我们将其命名为**车辆实例化块**。

```java
Car car = new Car("Mercedes-Benz", "S500", 5, 250.0);
Truck truck = new Truck("Isuzu", "NQR", 7500.0);

List<Vehicle> vehicles = new ArrayList<>();
vehicles.add(car);
vehicles.add(truck);

Fleet serializedFleet = new Fleet();
serializedFleet.setVehicles(vehicles);
```

这些填充的对象将被序列化:

```java
String jsonDataString = mapper.writeValueAsString(serializedFleet);
```

产生的 JSON 字符串:

```java
{
    "vehicles": 
    [
        "java.util.ArrayList",
        [
            [
                "com.baeldung.jackson.inheritance.Car",
                {
                    "make": "Mercedes-Benz",
                    "model": "S500",
                    "seatingCapacity": 5,
                    "topSpeed": 250.0
                }
            ],

            [
                "com.baeldung.jackson.inheritance.Truck",
                {
                    "make": "Isuzu",
                    "model": "NQR",
                    "payloadCapacity": 7500.0
                }
            ]
        ]
    ]
}
```

在反序列化过程中，从 JSON 字符串中恢复对象，并保留类型数据:

```java
Fleet deserializedFleet = mapper.readValue(jsonDataString, Fleet.class);
```

重新创建的对象将具有与序列化前相同的具体子类型:

```java
assertThat(deserializedFleet.getVehicles().get(0), instanceOf(Car.class));
assertThat(deserializedFleet.getVehicles().get(1), instanceOf(Truck.class));
```

### 2.2。每类注释

每类注释是一种包含类型信息的强大方法，对于需要大量定制的复杂用例非常有用。然而，这只能以复杂化为代价来实现。如果类型信息是以两种方式配置的，则每类注释将覆盖全局默认类型。

为了使用这个方法，超类型应该用`@JsonTypeInfo`和其他几个相关的注释进行注释。这一小节将使用一个类似于前面例子中的`Vehicle`结构的数据模型来说明每个类的注释。唯一的变化是在`Vehicle`抽象类上增加了注释，如下所示:

```java
@JsonTypeInfo(
  use = JsonTypeInfo.Id.NAME, 
  include = JsonTypeInfo.As.PROPERTY, 
  property = "type")
@JsonSubTypes({ 
  @Type(value = Car.class, name = "car"), 
  @Type(value = Truck.class, name = "truck") 
})
public abstract class Vehicle {
    // fields, constructors, getters and setters
}
```

使用前面小节中介绍的**车辆实例化块**创建数据对象，然后序列化:

```java
String jsonDataString = mapper.writeValueAsString(serializedFleet);
```

序列化产生以下 JSON 结构:

```java
{
    "vehicles": 
    [
        {
            "type": "car",
            "make": "Mercedes-Benz",
            "model": "S500",
            "seatingCapacity": 5,
            "topSpeed": 250.0
        },

        {
            "type": "truck",
            "make": "Isuzu",
            "model": "NQR",
            "payloadCapacity": 7500.0
        }
    ]
}
```

该字符串用于重新创建数据对象:

```java
Fleet deserializedFleet = mapper.readValue(jsonDataString, Fleet.class);
```

最后，对整个过程进行了验证:

```java
assertThat(deserializedFleet.getVehicles().get(0), instanceOf(Car.class));
assertThat(deserializedFleet.getVehicles().get(1), instanceOf(Truck.class));
```

## 3。忽略超类型的属性

有时，在序列化或反序列化过程中，需要忽略从超类继承的一些属性。这可以通过三种方法之一来实现:注释、混合和注释自省。

### 3.1。注释

有两个常用的 Jackson 注释来忽略属性，分别是`@JsonIgnore`和`@JsonIgnoreProperties`。前者直接应用于类型成员，告诉 Jackson 在序列化或反序列化时忽略相应的属性。后者用于任何级别，包括类型和类型成员，以列出应该忽略的属性。

`@JsonIgnoreProperties`比另一个更强大，因为它允许我们忽略从我们无法控制的超类型继承的属性，比如外部库中的类型。此外，这个注释允许我们一次忽略许多属性，这在某些情况下可以产生更容易理解的代码。

下面的类结构用于演示注释的用法:

```java
public abstract class Vehicle {
    private String make;
    private String model;

    protected Vehicle(String make, String model) {
        this.make = make;
        this.model = model;
    }

    // no-arg constructor, getters and setters
}

@JsonIgnoreProperties({ "model", "seatingCapacity" })
public abstract class Car extends Vehicle {
    private int seatingCapacity;

    @JsonIgnore
    private double topSpeed;

    protected Car(String make, String model, int seatingCapacity, double topSpeed) {
        super(make, model);
        this.seatingCapacity = seatingCapacity;
        this.topSpeed = topSpeed;
    }

    // no-arg constructor, getters and setters
}

public class Sedan extends Car {
    public Sedan(String make, String model, int seatingCapacity, double topSpeed) {
        super(make, model, seatingCapacity, topSpeed);
    }

    // no-arg constructor
}

public class Crossover extends Car {
    private double towingCapacity;

    public Crossover(String make, String model, int seatingCapacity, 
      double topSpeed, double towingCapacity) {
        super(make, model, seatingCapacity, topSpeed);
        this.towingCapacity = towingCapacity;
    }

    // no-arg constructor, getters and setters
}
```

如你所见，`@JsonIgnore`告诉杰克森忽略`Car.topSpeed`属性，而`@JsonIgnoreProperties`忽略`Vehicle.model`和`Car.seatingCapacity`属性。

下面的测试验证了这两种注释的行为。首先，我们需要实例化`ObjectMapper`和数据类，然后使用那个`ObjectMapper`实例来序列化数据对象:

```java
ObjectMapper mapper = new ObjectMapper();

Sedan sedan = new Sedan("Mercedes-Benz", "S500", 5, 250.0);
Crossover crossover = new Crossover("BMW", "X6", 5, 250.0, 6000.0);

List<Vehicle> vehicles = new ArrayList<>();
vehicles.add(sedan);
vehicles.add(crossover);

String jsonDataString = mapper.writeValueAsString(vehicles);
```

`jsonDataString`包含以下 JSON 数组:

```java
[
    {
        "make": "Mercedes-Benz"
    },
    {
        "make": "BMW",
        "towingCapacity": 6000.0
    }
]
```

最后，我们将证明结果 JSON 字符串中是否存在各种属性名:

```java
assertThat(jsonDataString, containsString("make"));
assertThat(jsonDataString, not(containsString("model")));
assertThat(jsonDataString, not(containsString("seatingCapacity")));
assertThat(jsonDataString, not(containsString("topSpeed")));
assertThat(jsonDataString, containsString("towingCapacity"));
```

### 3.2。混音

mix-in 允许我们应用行为(比如在序列化和反序列化时忽略属性)，而不需要直接对类应用注释。这在处理第三方类时特别有用，因为我们不能直接修改代码。

这一小节重用了上一小节中介绍的类继承链，除了在`Car`类上的`@JsonIgnore`和`@JsonIgnoreProperties`注释已经被移除:

```java
public abstract class Car extends Vehicle {
    private int seatingCapacity;
    private double topSpeed;

    // fields, constructors, getters and setters
}
```

为了演示 mix-in 的操作，我们将忽略`Vehicle.make`和`Car.topSpeed`属性，然后使用一个测试来确保一切按预期工作。

第一步是声明一个混合类型:

```java
private abstract class CarMixIn {
    @JsonIgnore
    public String make;
    @JsonIgnore
    public String topSpeed;
}
```

接下来，mix-in 通过一个`ObjectMapper`对象绑定到一个数据类:

```java
ObjectMapper mapper = new ObjectMapper();
mapper.addMixIn(Car.class, CarMixIn.class);
```

之后，我们实例化数据对象并将它们序列化为一个字符串:

```java
Sedan sedan = new Sedan("Mercedes-Benz", "S500", 5, 250.0);
Crossover crossover = new Crossover("BMW", "X6", 5, 250.0, 6000.0);

List<Vehicle> vehicles = new ArrayList<>();
vehicles.add(sedan);
vehicles.add(crossover);

String jsonDataString = mapper.writeValueAsString(vehicles);
```

`jsonDataString`现在包含以下 JSON:

```java
[
    {
        "model": "S500",
        "seatingCapacity": 5
    },
    {
        "model": "X6",
        "seatingCapacity": 5,
        "towingCapacity": 6000.0
    }
]
```

最后，让我们验证结果:

```java
assertThat(jsonDataString, not(containsString("make")));
assertThat(jsonDataString, containsString("model"));
assertThat(jsonDataString, containsString("seatingCapacity"));
assertThat(jsonDataString, not(containsString("topSpeed")));
assertThat(jsonDataString, containsString("towingCapacity"));
```

### 3.3。注释自省

注释自省是忽略超类型属性的最强大的方法，因为它允许使用`AnnotationIntrospector.hasIgnoreMarker` API 进行详细的定制。

这一小节使用与前一小节相同的类层次结构。在这个用例中，我们将要求 Jackson 忽略`Vehicle.model`、`Crossover.towingCapacity`以及在`Car`类中声明的所有属性。让我们从声明一个扩展了`JacksonAnnotationIntrospector`接口的类开始:

```java
class IgnoranceIntrospector extends JacksonAnnotationIntrospector {
    public boolean hasIgnoreMarker(AnnotatedMember m) {
        return m.getDeclaringClass() == Vehicle.class && m.getName() == "model" 
          || m.getDeclaringClass() == Car.class 
          || m.getName() == "towingCapacity" 
          || super.hasIgnoreMarker(m);
    }
}
```

内省器将忽略任何与方法中定义的条件集匹配的属性(也就是说，它将把它们视为通过其他方法之一被标记为忽略的属性)。

下一步是用一个`ObjectMapper`对象注册一个`IgnoranceIntrospector`类的实例:

```java
ObjectMapper mapper = new ObjectMapper();
mapper.setAnnotationIntrospector(new IgnoranceIntrospector());
```

现在，我们以与 3.2 节相同的方式创建和序列化数据对象。新生成的字符串的内容是:

```java
[
    {
        "make": "Mercedes-Benz"
    },
    {
        "make": "BMW"
    }
]
```

最后，我们将验证自省器是否按预期工作:

```java
assertThat(jsonDataString, containsString("make"));
assertThat(jsonDataString, not(containsString("model")));
assertThat(jsonDataString, not(containsString("seatingCapacity")));
assertThat(jsonDataString, not(containsString("topSpeed")));
assertThat(jsonDataString, not(containsString("towingCapacity")));
```

## 4。子类型处理场景

本节将讨论与子类处理相关的两个有趣的场景。

### 4.1。子类型之间的转换

Jackson 允许将一个对象转换成不同于原始类型的类型。事实上，这种转换可能发生在任何兼容的类型之间，但是当用于同一接口或类的两个子类型之间以保护值和功能时，它是最有帮助的。

为了演示从一种类型到另一种类型的转换，我们将重用第 2 节中的`Vehicle`层次结构，在`Car`和`Truck`中的属性上添加了`@JsonIgnore`注释以避免不兼容。

```java
public class Car extends Vehicle {
    @JsonIgnore
    private int seatingCapacity;

    @JsonIgnore
    private double topSpeed;

    // constructors, getters and setters
}

public class Truck extends Vehicle {
    @JsonIgnore
    private double payloadCapacity;

    // constructors, getters and setters
}
```

下面的代码将验证转换是否成功，以及新对象是否保留了旧对象的数据值:

```java
ObjectMapper mapper = new ObjectMapper();

Car car = new Car("Mercedes-Benz", "S500", 5, 250.0);
Truck truck = mapper.convertValue(car, Truck.class);

assertEquals("Mercedes-Benz", truck.getMake());
assertEquals("S500", truck.getModel());
```

### 4.2。不带无参数构造函数的反序列化

默认情况下，Jackson 通过使用无参数构造函数来重新创建数据对象。这在某些情况下是不方便的，比如当一个类有非默认的构造函数，用户不得不写无参数的构造函数来满足 Jackson 的要求。在类层次结构中，这甚至更麻烦，因为必须将无参数构造函数添加到类和继承链中所有更高的类中。在这些情况下， **creator methods** 就来帮忙了。

这一节将使用与第 2 节相似的对象结构，只是对构造函数做了一些修改。具体来说，所有无参数的构造函数都被丢弃，具体子类型的构造函数用`@JsonCreator`和`@JsonProperty`标注，使它们成为 creator 方法。

```java
public class Car extends Vehicle {

    @JsonCreator
    public Car(
      @JsonProperty("make") String make, 
      @JsonProperty("model") String model, 
      @JsonProperty("seating") int seatingCapacity, 
      @JsonProperty("topSpeed") double topSpeed) {
        super(make, model);
        this.seatingCapacity = seatingCapacity;
            this.topSpeed = topSpeed;
    }

    // fields, getters and setters
}

public class Truck extends Vehicle {

    @JsonCreator
    public Truck(
      @JsonProperty("make") String make, 
      @JsonProperty("model") String model, 
      @JsonProperty("payload") double payloadCapacity) {
        super(make, model);
        this.payloadCapacity = payloadCapacity;
    }

    // fields, getters and setters
}
```

测试将验证 Jackson 可以处理缺少无参数构造函数的对象:

```java
ObjectMapper mapper = new ObjectMapper();
mapper.enableDefaultTyping();

Car car = new Car("Mercedes-Benz", "S500", 5, 250.0);
Truck truck = new Truck("Isuzu", "NQR", 7500.0);

List<Vehicle> vehicles = new ArrayList<>();
vehicles.add(car);
vehicles.add(truck);

Fleet serializedFleet = new Fleet();
serializedFleet.setVehicles(vehicles);

String jsonDataString = mapper.writeValueAsString(serializedFleet);
mapper.readValue(jsonDataString, Fleet.class);
```

## 5。结论

本教程涵盖了几个有趣的用例来展示 Jackson 对类型继承的支持，重点是多态和忽略超类型属性。

所有这些例子和代码片段的实现都可以在 GitHub 项目中找到。