# Java 中的密封类和接口

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-sealed-classes-interfaces>

## 1.概观

Java SE 17 的发布引入了密封类( [JEP 409](https://web.archive.org/web/20220816172057/https://openjdk.org/jeps/409) )。

这个特性是关于在 Java 中实现更细粒度的继承控制。密封允许类和接口定义它们允许的子类型。

换句话说，一个类或一个接口现在可以定义哪些类可以实现或扩展它。对于领域建模和增加库的安全性来说，这是一个有用的特性。

## 2.动机

类层次结构使我们能够通过继承来重用代码。但是，类层次结构也可以有其他用途。代码重用是伟大的，但并不总是我们的主要目标。

### 2.1.建模可能性

类层次结构的另一个目的是对领域中存在的各种可能性进行建模。

作为一个例子，设想一个只适用于汽车和卡车，而不适用于摩托车的业务域。在 Java 中创建`Vehicle`抽象类时，我们应该只允许`Car`和`Truck`类扩展它。通过这种方式，我们希望确保在我们的领域内不会出现滥用`Vehicle`抽象类的情况。

**在这个例子中，我们更感兴趣的是处理已知子类的代码的清晰性，而不是防御所有未知子类**。

在版本 15 之前(在版本 15 中，密封类作为预览版引入)，Java 假定代码重用始终是一个目标。每个类都可以被任意数量的子类扩展。

### 2.2.包私有方法

在早期版本中，Java 在继承控制方面提供了有限的选项。

一个[最终类](/web/20220816172057/https://www.baeldung.com/java-final)不能有子类。一个[包-私有类](/web/20220816172057/https://www.baeldung.com/java-access-modifiers)在同一个包中只能有子类。

使用包私有方法，如果不允许用户扩展抽象类，用户就不能访问它:

```java
public class Vehicles {

    abstract static class Vehicle {

        private final String registrationNumber;

        public Vehicle(String registrationNumber) {
            this.registrationNumber = registrationNumber;
        }

        public String getRegistrationNumber() {
            return registrationNumber;
        }

    }

    public static final class Car extends Vehicle {

        private final int numberOfSeats;

        public Car(int numberOfSeats, String registrationNumber) {
            super(registrationNumber);
            this.numberOfSeats = numberOfSeats;
        }

        public int getNumberOfSeats() {
            return numberOfSeats;
        }

    }

    public static final class Truck extends Vehicle {

        private final int loadCapacity;

        public Truck(int loadCapacity, String registrationNumber) {
            super(registrationNumber);
            this.loadCapacity = loadCapacity;
        }

        public int getLoadCapacity() {
            return loadCapacity;
        }

    }

}
```

### 2.3.超类可访问，不可扩展

用一组子类开发的超类应该能够记录它的预期用途，而不是约束它的子类。同样，限制子类不应该限制其超类的可访问性。

因此，密封类背后的主要动机是让超类有可能被广泛访问，但不能被广泛扩展。

## 3.创造

密封特性在 Java 中引入了几个新的修饰符和子句:`sealed, non-sealed,`和`permits`。

### 3.1.密封接口

要密封一个接口，我们可以对它的声明应用`sealed`修饰符。然后，`permits`子句指定了允许实现密封接口的类:

```java
public sealed interface Service permits Car, Truck {

    int getMaxServiceIntervalInMonths();

    default int getMaxDistanceBetweenServicesInKilometers() {
        return 100000;
    }

}
```

### 3.2.密封类

类似于接口，我们可以通过应用相同的`sealed`修饰符来密封类。`permits`子句应定义在任何`extends`或`implements` 子句之后；

```java
public abstract sealed class Vehicle permits Car, Truck {

    protected final String registrationNumber;

    public Vehicle(String registrationNumber) {
        this.registrationNumber = registrationNumber;
    }

    public String getRegistrationNumber() {
        return registrationNumber;
    }

}
```

允许的子类必须定义一个修饰符。可以将[声明为`final`](/web/20220816172057/https://www.baeldung.com/java-final) 以防止任何进一步的扩展:

```java
public final class Truck extends Vehicle implements Service {

    private final int loadCapacity;

    public Truck(int loadCapacity, String registrationNumber) {
        super(registrationNumber);
        this.loadCapacity = loadCapacity;
    }

    public int getLoadCapacity() {
        return loadCapacity;
    }

    @Override
    public int getMaxServiceIntervalInMonths() {
        return 18;
    }

}
```

允许的子类也可以声明为`sealed`。然而，如果我们声明它为`non-sealed,` ，那么它是开放扩展的:

```java
public non-sealed class Car extends Vehicle implements Service {

    private final int numberOfSeats;

    public Car(int numberOfSeats, String registrationNumber) {
        super(registrationNumber);
        this.numberOfSeats = numberOfSeats;
    }

    public int getNumberOfSeats() {
        return numberOfSeats;
    }

    @Override
    public int getMaxServiceIntervalInMonths() {
        return 12;
    }

}
```

### 3.4.限制

密封类对其允许的子类施加了三个重要的约束:

1.  所有允许的子类必须与密封类属于同一个模块。
2.  每个允许的子类必须显式扩展密封类。
3.  每个允许的子类必须定义一个修饰符:`final`、`sealed`或`non-sealed.`

## 4.使用

### 4.1.传统的方式

当密封一个类时，我们使客户端代码能够清楚地推理出所有允许的子类。

推理子类的传统方法是使用一组`if-else`语句和`instanceof` 检查:

```java
if (vehicle instanceof Car) {
    return ((Car) vehicle).getNumberOfSeats();
} else if (vehicle instanceof Truck) {
    return ((Truck) vehicle).getLoadCapacity();
} else {
    throw new RuntimeException("Unknown instance of Vehicle");
}
```

### 4.2.模式匹配

通过应用[模式匹配](/web/20220816172057/https://www.baeldung.com/java-pattern-matching-instanceof)，我们可以避免额外的类转换，但是我们仍然需要一组 i `f-else`语句:

```java
if (vehicle instanceof Car car) {
    return car.getNumberOfSeats();
} else if (vehicle instanceof Truck truck) {
    return truck.getLoadCapacity();
} else {
    throw new RuntimeException("Unknown instance of Vehicle");
}
```

使用 i `f-else` 使得编译器很难确定我们覆盖了所有允许的子类。出于这个原因，我们正在抛出一个`RuntimeException`。

在 Java 的未来版本中，客户端代码将能够使用一个`switch` 语句来代替 i `f-else` ( [JEP 375](https://web.archive.org/web/20220816172057/https://openjdk.java.net/jeps/375) )。

通过使用[类型测试模式](https://web.archive.org/web/20220816172057/https://openjdk.java.net/jeps/8213076)，编译器将能够检查每个允许的子类是否被覆盖。因此，将不再需要`default`条款/案例。

## 4.和睦相处

现在让我们看看密封类与其他 Java 语言特性的兼容性，比如记录和反射 API。

### 4.1.记录

密封类可以很好地处理[记录](/web/20220816172057/https://www.baeldung.com/java-record-keyword)。因为记录是隐式最终的，所以密封的层次结构更加简洁。让我们尝试使用记录重写我们的类示例:

```java
public sealed interface Vehicle permits Car, Truck {

    String getRegistrationNumber();

}

public record Car(int numberOfSeats, String registrationNumber) implements Vehicle {

    @Override
    public String getRegistrationNumber() {
        return registrationNumber;
    }

    public int getNumberOfSeats() {
        return numberOfSeats;
    }

}

public record Truck(int loadCapacity, String registrationNumber) implements Vehicle {

    @Override
    public String getRegistrationNumber() {
        return registrationNumber;
    }

    public int getLoadCapacity() {
        return loadCapacity;
    }

}
```

### 4.2.反射

密封类也受[反射 API](/web/20220816172057/https://www.baeldung.com/java-reflection) 支持，其中两个公共方法被添加到了`java.lang.Class:`

*   如果给定的类或接口是密封的，`isSealed`方法返回`true`。
*   方法`getPermittedSubclasses`返回代表所有允许的子类的对象数组。

我们可以利用这些方法来创建基于我们的示例的断言:

```java
Assertions.assertThat(truck.getClass().isSealed()).isEqualTo(false);
Assertions.assertThat(truck.getClass().getSuperclass().isSealed()).isEqualTo(true);
Assertions.assertThat(truck.getClass().getSuperclass().getPermittedSubclasses())
  .contains(ClassDesc.of(truck.getClass().getCanonicalName()));
```

## 5.结论

在本文中，我们探索了 Java SE 17 中的一个新特性——密封类和接口。我们讨论了密封类和接口的创建和使用，以及它们的约束和与其他语言特性的兼容性。

在示例中，我们介绍了密封接口和密封类的创建、密封类的使用(有和没有模式匹配)，以及密封类与记录和反射 API 的兼容性。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220816172057/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-17)