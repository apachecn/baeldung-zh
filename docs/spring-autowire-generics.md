# 泛型类型的 Spring 自动布线

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-autowire-generics>

## 1.概观

在本教程中，我们将看到如何通过[通用参数](/web/20220628150125/https://www.baeldung.com/java-generics)注入 Spring beans。

## 2.Spring 3.2 中的自动连接泛型。

从 3.2 版本开始，Spring 支持泛型类型的注入。

假设我们有一个叫做`Vehicle `的抽象类和一个叫做`Car:`的具体子类

```java
public abstract class Vehicle {
    private String name;
    private String manufacturer;

    // ... getters, setters etc
}
```

```java
public class Car extends Vehicle {
    private String engineType;

    // ... getters, setters etc
}
```

假设我们想要将类型为`Vehicle`的对象列表注入到某个处理程序类中:

```java
@Autowired
private List<Vehicle> vehicles;
```

**Spring 将所有的`Vehicle`实例 beans 自动连接到这个列表中。**我们如何通过 Java 或 XML 配置实例化这些 beans 并不重要。

我们也可以使用限定符来获得特定的`Vehicle`类型的 beans。然后我们创建`@CarQualifier`，并用`@Qualifier`对其进行注释:

```java
@Target({
  ElementType.FIELD, 
  ElementType.METHOD,
  ElementType.TYPE, 
  ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface CarQualifier {
}
```

现在我们可以在我们的列表中使用这个注释来获得一些特定的`Vehicles`:

```java
@Autowired
@CarQualifier
private List<Vehicle> vehicles;
```

**在这种情况下，我们可以创建几个`Vehicle`beans，但是 Spring 只会将那些带有`@CarQualifier`的 bean 注入到上面的列表中:**

```java
public class CustomConfiguration {
    @Bean
    @CarQualifier
    public Car getMercedes() {
        return new Car("E280", "Mercedes", "Diesel");
    }
}
```

## 3.Spring 4.0 中的自动连接泛型。

假设我们有另一个叫做`Motorcycle`的`Vehicle`子类:

```java
public class Motorcycle extends Vehicle {
    private boolean twoWheeler;
    //... getters, setters etc
}
```

现在，如果我们只想将`Car`bean 注入到我们的列表中，而不想注入`Motorcycle `bean，我们可以通过使用特定的子类作为类型参数来实现:

```java
@Autowired
private List<Car> vehicles; 
```

从 4.0 版本开始，Spring 允许我们使用泛型类型作为限定符，而不需要显式注释。

在 Spring 4.0 之前，上面的代码在没有显式限定符的情况下不能处理`Vehicle` `.`的多个子类的 beans，我们会收到一个`NonUniqueBeanDefinitionException`。

## 4.`ResolvableType`

**泛型自动连接特性在后台的`ResolvableType`类的帮助下工作。**

它是在 Spring 4.0 中引入的，用于封装 Java 类型，处理对超类型、接口、泛型参数的访问，并最终解析为一个类:

```java
ResolvableType vehiclesType = ResolvableType.forField(getClass().getDeclaredField("vehicles"));
System.out.println(vehiclesType);

ResolvableType type = vehiclesType.getGeneric();
System.out.println(type);

Class<?> aClass = type.resolve();
System.out.println(aClass);
```

上述代码的输出将显示相应的简单类型和一般类型:

```java
java.util.List<com.example.model.Vehicle>
com.example.model.Vehicle
class com.example.model.Vehicle 
```

## 5.结论

泛型类型的注入是一个强大的特性，它节省了开发人员分配显式限定符的工作量，使代码更干净、更容易理解。

和往常一样，代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220628150125/https://github.com/eugenp/tutorials/tree/master/spring-di)