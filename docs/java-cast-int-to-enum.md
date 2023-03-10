# 在 Java 中将 int 强制转换为 Enum

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-cast-int-to-enum>

## 1.概观

在本教程中，我们将简要介绍在 Java 中将`int` 转换为[枚举](/web/20220628143636/https://www.baeldung.com/a-guide-to-java-enums)值的不同方法。虽然没有直接的造型方法，但有几种近似的方法。

## 2.使用`Enum` # `values`

首先，让我们看看如何使用 [`Enum`的`values`方法](https://web.archive.org/web/20220628143636/https://docs.oracle.com/javase/tutorial/java/javaOO/enum.html)来解决这个问题。

让我们首先创建一个枚举`PizzaStatus`来定义一个比萨饼订单的状态:

```java
public enum PizzaStatus {
    ORDERED(5),
    READY(2),
    DELIVERED(0);

    private int timeToDelivery;

    PizzaStatus (int timeToDelivery) {
        this.timeToDelivery = timeToDelivery;
    }

    // Method that gets the timeToDelivery variable.
}
```

我们将每个常量枚举值与`timeToDelivery`字段相关联。当定义常量枚举时，我们将`timeToDelivery`字段传递给构造函数。

**[`static`](/web/20220628143636/https://www.baeldung.com/java-static) `values`方法返回一个数组，该数组包含枚举的所有值，按其声明的顺序排列。**因此，我们可以用`timeToDelivery`的整数值来得到相应的 enum 值:

```java
int timeToDeliveryForOrderedPizzaStatus = 5;

PizzaStatus pizzaOrderedStatus = null;

for (PizzaStatus pizzaStatus : PizzaStatus.values()) {
    if (pizzaStatus.getTimeToDelivery() == timeToDeliveryForOrderedPizzaStatus) {
        pizzaOrderedStatus = pizzaStatus;
    }
}

assertThat(pizzaOrderedStatus).isEqualTo(PizzaStatus.ORDERED);
```

这里，我们使用一个由`PizzaStatus.values()`返回的数组来根据`timeToDelivery`属性找到一个匹配值。

然而，这种方法相当冗长。此外，**也是低效的，因为每次我们想要获取相应的`PizzaStatus`，我们都需要迭代`PizzaStatus.values()`。**

### 2.1.使用 Java 8 `Stream`

让我们看看如何使用 Java 8 方法找到匹配的`PizzaStatus`:

```java
int timeToDeliveryForOrderedPizzaStatus = 5;

Optional<PizzaStatus> pizzaStatus = Arrays.stream(PizzaStatus.values())
  .filter(p -> p.getTimeToDelivery() == timeToDeliveryForOrderedPizzaStatus)
  .findFirst();

assertThat(pizzaStatus).hasValue(PizzaStatus.ORDERED);
```

这段代码看起来比使用`for`循环的代码更简洁。然而，我们仍然在每次需要获取匹配的枚举时迭代`PizzaStatus.values()`。

还要注意，在这种方法中，我们得到的是 **`Optional<PizzaStatus>`，而不是直接得到的`PizzaStatus`实例**。

## 3.使用`Map`

接下来，让我们使用 Java 的`Map`数据结构和`values`方法来获取与交付整数值的时间相对应的枚举值。

**在这种方法中，在初始化映射**时，`values`方法只被调用一次。此外，因为我们使用的是 map，所以我们不需要在每次需要获取对应于交付时间的 enum 值时迭代这些值。

我们在内部使用一个静态映射`timeToDeliveryToEnumValuesMapping`,它处理交付时间到相应 enum 值的映射。

此外，`Enum`类的`values`方法提供了所有的枚举值。**在`static`块中，我们遍历枚举值的数组，并将它们和相应的时间一起添加到映射中，以将整数值作为关键字:**

```java
private static Map<Integer, PizzaStatus> timeToDeliveryToEnumValuesMapping = new HashMap<>();

static {
    for (PizzaStatus pizzaStatus : PizzaStatus.values()) {
        timeToDeliveryToEnumValuesMapping.put(
          pizzaStatus.getTimeToDelivery(),
          pizzaStatus
        );
    }
}
```

最后，我们创建一个`static`方法，它将`timeToDelivery`整数作为参数。该方法使用静态映射`timeToDeliveryToEnumValuesMapping`返回相应的枚举值:

```java
public static PizzaStatus castIntToEnum(int timeToDelivery) {
    return timeToDeliveryToEnumValuesMapping.get(timeToDelivery);
}
```

通过使用静态映射和静态方法，我们获取与交付整数值的时间相对应的枚举值。

## 4.结论

总之，我们研究了几种获取与整数值相对应的枚举值的变通方法。

和往常一样，所有这些代码示例都可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220628143636/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-2)