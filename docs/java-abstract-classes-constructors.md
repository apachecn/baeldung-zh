# Java 抽象类中的构造函数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-abstract-classes-constructors>

## 1.概观

抽象类和构造函数可能看起来不兼容。**构造函数是类被实例化时调用的方法**，**抽象类不能被实例化**。听起来很违反直觉，对吧？

在本文中，我们将看到为什么抽象类可以有构造函数，以及使用它们如何在子类实例化中提供好处。

## 2.默认构造函数

当一个类没有声明任何构造函数时，编译器会为我们创建一个默认的构造函数。抽象类也是如此。即使没有显式的构造函数，抽象类也会有一个默认的构造函数。

在抽象类中，它的后代可以使用`super()`调用抽象默认构造函数:

```java
public abstract class AbstractClass {
    // compiler creates a default constructor
}

public class ConcreteClass extends AbstractClass {

    public ConcreteClass() {
        super();
    }
}
```

## 3.无参数构造函数

我们可以在抽象类中声明一个不带参数的构造函数。它将覆盖默认的构造函数，任何子类的创建都将在构造链中首先调用它。

让我们用一个抽象类的两个子类来验证这个行为:

```java
public abstract class AbstractClass {
    public AbstractClass() {
        System.out.println("Initializing AbstractClass");
    }
}

public class ConcreteClassA extends AbstractClass {
}

public class ConcreteClassB extends AbstractClass {
    public ConcreteClassB() {
        System.out.println("Initializing ConcreteClassB");
    }
}
```

让我们看看调用 `new ConcreateClassA()`时得到的输出:

```java
Initializing AbstractClass
```

而调用`new ConcreteClassB()`的输出将是:

```java
Initializing AbstractClass
Initializing ConcreteClassB
```

### 3.1.安全初始化

声明不带参数的抽象构造函数有助于安全初始化。

下面的`Counter`类是计算自然数的超类。我们需要它的值从零开始。

让我们看看如何使用无参数构造函数来确保安全初始化:

```java
public abstract class Counter {

    int value;

    public Counter() {
        this.value = 0;
    }

    abstract int increment();
}
```

我们的`SimpleCounter`子类用`++`操作符实现了`increment()`方法。它在每次调用时将`value`递增 1:

```java
public class SimpleCounter extends Counter {

    @Override
    int increment() {
        return ++value;
    }
}
```

注意`SimpleCounter `没有声明任何构造函数。它的创建依赖于默认情况下调用的计数器的无参数构造函数。

下面的单元测试演示了构造函数如何安全地初始化`value`属性:

```java
@Test
void givenNoArgAbstractConstructor_whenSubclassCreation_thenCalled() {
    Counter counter = new SimpleCounter();

    assertNotNull(counter);
    assertEquals(0, counter.value);
}
```

### 3.2.阻止访问

我们的`Counter `初始化工作正常，但是让我们想象我们不希望子类覆盖这个安全初始化。

首先，我们需要将构造函数设为私有，以防止子类拥有访问权限:

```java
private Counter() {
    this.value = 0;
    System.out.println("Counter No-Arguments constructor");
}
```

其次，让我们为子类创建另一个构造函数来调用:

```java
public Counter(int value) {
    this.value = value;
    System.out.println("Parametrized Counter constructor");
}
```

最后，我们的`SimpleCounter`需要覆盖参数化的构造函数，否则，它不会编译:

```java
public class SimpleCounter extends Counter {

    public SimpleCounter(int value) {
        super(value);
    }

    // concrete methods
}
```

注意编译器期望我们在这个构造函数上调用`super(value)`，以限制对我们的`private`无参数构造函数的访问。

## 4.参数化构造函数

抽象类中构造函数最常见的用法之一是避免冗余。让我们用 cars 创建一个例子，看看我们如何利用参数化的构造函数。

我们从抽象的`Car`类开始，表示所有类型的汽车。我们还需要一个`distance`属性来知道它移动了多少:

```java
public abstract class Car {

    int distance;

    public Car(int distance) {
        this.distance = distance;
    }
}
```

我们的超类看起来不错，但是我们不希望用非零值初始化`distance`属性。我们还想防止子类改变`distance`属性或者覆盖参数化的构造函数。

让我们看看如何限制对`distance`的访问，并使用构造函数安全地初始化它:

```java
public abstract class Car {

    private int distance;

    private Car(int distance) {
        this.distance = distance;
    }

    public Car() {
        this(0);
        System.out.println("Car default constructor");
    }

    // getters
}
```

现在，我们的`distance`属性和参数化构造函数是私有的。有一个公共默认构造函数`Car()`，它委托私有构造函数来初始化`distance`。

为了使用我们的`distance`属性，让我们添加一些行为来获取和显示汽车的基本信息:

```java
abstract String getInformation();

protected void display() {
    String info = new StringBuilder(getInformation())
      .append("\nDistance: " + getDistance())
      .toString();
    System.out.println(info);
}
```

所有的子类都需要提供一个`getInformation()`的实现，`display()`方法会用它来打印所有的细节。

现在让我们创建`ElectricCar`和`FuelCar`子类:

```java
public class ElectricCar extends Car {
    int chargingTime;

    public ElectricCar(int chargingTime) {
        this.chargingTime = chargingTime;
    }

    @Override
    String getInformation() {
        return new StringBuilder("Electric Car")
          .append("\nCharging Time: " + chargingTime)
          .toString();
    }
}

public class FuelCar extends Car {
    String fuel;

    public FuelCar(String fuel) {
        this.fuel = fuel;
    }

    @Override
    String getInformation() {
        return new StringBuilder("Fuel Car")
          .append("\nFuel type: " + fuel)
          .toString();
    }
}
```

让我们看看这些子类的作用:

```java
ElectricCar electricCar = new ElectricCar(8);
electricCar.display();

FuelCar fuelCar = new FuelCar("Gasoline");
fuelCar.display();
```

产生的输出如下所示:

```java
Car default constructor
Electric Car
Charging Time: 8
Distance: 0

Car default constructor
Fuel Car
Fuel type: Gasoline
Distance: 0
```

## 5.结论

像 Java 中的其他类一样，抽象类可以有构造函数，即使它们只是从具体的子类中被调用。

在本文中，我们从抽象类的角度介绍了每种类型的构造函数——它们如何与 concreate 子类相关，以及我们如何在实际用例中使用它们。

和往常一样，代码样本可以在 GitHub 上找到[。](https://web.archive.org/web/20220525121135/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-oop-constructors)