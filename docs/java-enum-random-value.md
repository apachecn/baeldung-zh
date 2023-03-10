# 从枚举生成随机值

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-enum-random-value>

## 1.概观

在本教程中，我们将学习如何从一个`[enum](/web/20221012233249/https://www.baeldung.com/a-guide-to-java-enums)`生成一个随机值。

## 2.使用`static` 方法随机选择`Enum`值

首先，我们将创建一个从特定的`enum`集合中返回随机生成值的`static`函数。 **`Enum`值代表一组常数；然而，我们仍然可以在`enum`类体中声明`static` 方法。** **我们将利用一个`static` 方法作为助手来生成一个随机的`enum`值。**

我们在`enum`类体内声明了一个方法，它是`static`，并返回一个`enum`值。这个方法将从一个 [`Random`](/web/20221012233249/https://www.baeldung.com/java-generating-random-numbers)) 对象中调用`nextInt()` ，我们将这个方法命名为`randomDirection()`:

```java
public enum Direction {
    EAST, WEST, SOUTH, NORTH;

    private static final Random PRNG = new Random();

    public static Direction randomDirection()  {
        Direction[] directions = values();
        return directions[PRNG.nextInt(directions.length)];
    }
}
```

在`randomDirection()`内部，我们用整数参数调用方法`nextInt()` 。`nextInt()` 方法返回一个随机数来访问`directions` 数组；因此，我们需要通过向`nextInt()`传递一个`bound`参数来确保整数不超出数组的界限。`bound` 参数是方向的总数，我们知道不会超过数组的大小。

此外，每次调用`randomDirection()` 方法时，`values()`方法都会创建一个`enum`值的副本。我们可以通过创建一个在生成随机索引后访问的`final` 成员变量列表来提高性能:

```java
private static final Direction[] directions = values();
```

现在，`randomDirection()` 方法将如下所示:

```java
public static Direction randomDirection() { 
    return directions[PRNG.nextInt(directions.length)]; 
}
```

最后，我们可以通过调用方法生成一个随机的`Direction`:

```java
Direction direction = Direction.randomDirection();
```

## 3.带有泛型的随机`Enum`值

类似地，我们可以使用泛型来生成一个随机的`enum`值。**通过使用泛型，我们创建了一个类，它接受任何类型的`enum`数据来生成一个随机值:**

```java
public class RandomEnumGenerator<T extends Enum<T>> {
    private static final Random PRNG = new Random();
    private final T[] values;

    public RandomEnumGenerator(Class<T> e) {
        values = e.getEnumConstants();
    }

    public T randomEnum() {
        return values[PRNG.nextInt(values.length)];
    }
}
```

注意`randomEnum()` 方法与前一个例子中的`randomDirection()`方法是多么相似。不同之处在于`RandomEnumGenerator` 类有一个构造函数，它期望一个枚举类型，从中获取常量值。

我们可以使用`RandomEnumGenerator`类生成一个随机方向，如下所示:

```java
RandomEnumGenerator reg = new RandomEnumGenerator(Direction.class);
Direction direction = (Direction) reg.randomEnum();
```

这里，我们使用上一节中的`Direction` enum 类。`RandomEnumGenerator` 接受这个类，`direction`对象将引用`Direction`类中的一个常量值。

## 4。结论

在本教程中，我们学习了如何从一个`enum`中获取一个随机值。我们介绍了两种方法:首先，我们在`enum`类中使用一个`static` 方法，它生成一个随机值，这个随机值严格限制在声明该方法的`enum`类中。此外，我们看到了如何通过缓存常量值来提高性能。最后，我们通过使用一个接受任何类型的`enum`的类来利用泛型，以便获得一个随机值。

和往常一样，本文的完整代码示例可以在 GitHub 的[中找到。](https://web.archive.org/web/20221012233249/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-oop-types-2)