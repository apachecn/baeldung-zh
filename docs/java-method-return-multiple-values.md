# 如何从一个 Java 方法返回多个值

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-method-return-multiple-values>

## 1.概观

在本教程中，我们将学习从一个 Java 方法返回多个值的不同方法。

首先，我们将返回数组和集合。然后，我们将展示如何对复杂数据使用容器类，并学习如何创建通用元组类。

最后，我们将看到如何使用第三方库返回多个值的例子。

## 2.使用数组

**数组可用于返回原始和引用数据类型**。

例如，下面的`getCoordinates`方法返回两个`double`值的数组:

```java
double[] getCoordinatesDoubleArray() {

    double[] coordinates = new double[2];

    coordinates[0] = 10;
    coordinates[1] = 12.5;

    return coordinates;
}
```

**如果我们想返回一个不同引用类型的数组，我们可以使用一个公共的父类型作为数组的类型**:

```java
Number[] getCoordinatesNumberArray() {

    Number[] coordinates = new Number[2];

    coordinates[0] = 10;   // Integer
    coordinates[1] = 12.5; // Double

    return coordinates;
}
```

这里我们定义了类型为`Number`的`coordinates`数组，因为它是`Integer`和`Double`元素之间的公共类。

## 3.使用集合

使用泛型 [Java 集合](/web/20220913170115/https://www.baeldung.com/java-collections)、**，我们可以返回一个通用类型**的多个值。

集合框架有一系列的类和接口。然而，在本节中，我们将把讨论限制在`List`和`Map`接口上。

### 3.1.返回列表中相似类型的值

首先，让我们使用`List<Number>`重写前面的数组示例:

```java
List<Number> getCoordinatesList() {

    List<Number> coordinates = new ArrayList<>();

    coordinates.add(10);  // Integer
    coordinates.add(12.5);  // Double

    return coordinates;
}
```

像`Number[]`一样，`List<Number>`集合保存一系列混合类型的元素，它们都是相同的公共类型。

### 3.2.返回地图中的命名值

如果我们想要命名集合中的每个条目，可以使用`Map`来代替:

```java
Map<String, Number> getCoordinatesMap() {

    Map<String, Number> coordinates = new HashMap<>();

    coordinates.put("longitude", 10);
    coordinates.put("latitude", 12.5);

    return coordinates;
}
```

`getCoordinatesMap`方法的用户可以使用`longitude”`或`latitude”`键和`Map#get`方法来检索相应的值。

## 4.使用容器类

与数组和集合不同，**容器类(POJOs)可以用不同的数据类型包装多个字段**。

例如，下面的`Coordinates`类有两种不同的数据类型，`double`和`String`:

```java
public class Coordinates {

    private double longitude;
    private double latitude;
    private String placeName;

    public Coordinates(double longitude, double latitude, String placeName) {

        this.longitude = longitude;
        this.latitude = latitude;
        this.placeName = placeName;
    }

    // getters and setters
}
```

使用像`Coordinates`这样的容器类使我们能够用有意义的名字对复杂的数据类型建模。

下一步是实例化并返回`Coordinates`的实例:

```java
Coordinates getCoordinates() {

    double longitude = 10;
    double latitude = 12.5;
    String placeName = "home";

    return new Coordinates(longitude, latitude, placeName);
}
```

我们应该注意到**建议我们把数据类做成像`Coordinates`**这样不可变的。通过这样做，我们创建了简单的、线程安全的、可共享的对象。

## 5.使用元组

像容器一样，元组存储不同类型的字段。然而，它们的不同之处在于它们不是特定于应用的。

当我们使用它们来描述我们希望它们处理的类型时，它们是专门化的，但是它们是包含一定数量的值的通用容器。这意味着我们不需要编写定制代码来拥有它们，我们可以使用一个库，或者创建一个公共的单一实现。

一个元组可以是任意数量的字段，通常被称为`Tuple` n，其中 n 是字段的数量。例如，Tuple2 是两个字段的元组，Tuple3 是三个字段的元组，以此类推。

为了证明元组的重要性，让我们考虑下面的例子。假设我们想要找出一个`Coordinates`点和一个`List<Coordinates>`内所有其他点之间的距离。然后，我们需要返回最远的坐标对象，以及距离。

让我们首先创建一个通用的双字段元组:

```java
public class Tuple2<K, V> {

    private K first;
    private V second;

    public Tuple2(K first, V second){
        this.first = first;
        this.second = second;
    }

    // getters and setters
}
```

接下来，让我们实现我们的逻辑并使用一个`Tuple2<Coordinates, Double>`实例来包装结果:

```java
Tuple2<Coordinates, Double> getMostDistantPoint(List<Coordinates> coordinatesList, 
                                                       Coordinates target) {

    return coordinatesList.stream()
      .map(coor -> new Tuple2<>(coor, coor.calculateDistance(target)))
      .max((d1, d2) -> Double.compare(d1.getSecond(), d2.getSecond())) // compare distances
      .get();
}
```

**在前面的例子中使用`Tuple2<Coordinates, Double>`使我们不用创建一个单独的容器类来一次性使用这个特殊的方法**。

像容器一样，**元组应该是不可变的**。此外，由于它们的通用性质，**我们应该在内部使用元组，而不是作为我们公共 API 的一部分**。

## 6.第三方库

一些第三方库实现了不可变的`Pair`或`Triple`类型。 [Apache Commons Lang](https://web.archive.org/web/20220913170115/https://commons.apache.org/proper/commons-lang/) 和 [javatuples](https://web.archive.org/web/20220913170115/http://www.javatuples.org/index.html) 就是最好的例子。一旦我们在应用程序中将这些库作为依赖项，我们就可以直接使用这些库提供的`Pair`或`Triple`类型，而不是自己创建它们。

让我们看一个使用 Apache Commons Lang 返回一个`Pair`或`Triple`对象的例子。

在我们进一步之前，让我们在我们的`pom.xml:`中添加 [`commons-lang3`](https://web.archive.org/web/20220913170115/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.commons%22%20AND%20a%3A%22commons-lang3%22) 依赖项

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

### 6.1.`ImmutablePair`来自 Apache Commons Lang

Apache Commons Lang 中的`[ImmutablePair](https://web.archive.org/web/20220913170115/https://commons.apache.org/proper/commons-lang/javadocs/api-3.9/org/apache/commons/lang3/tuple/ImmutablePair.html)`类型正是我们想要的:一种不可变类型，其用法非常简单。

它包含两个字段:`left`和`right`。让我们看看如何让我们的`getMostDistantPoint`方法返回一个`ImmutablePair`类型的对象:

```java
ImmutablePair<Coordinates, Double> getMostDistantPoint(
  List<Coordinates> coordinatesList, Coordinates target) {
    return coordinatesList.stream()
      .map(coordinates -> ImmutablePair.of(coordinates, coordinates.calculateDistance(target)))
      .max(Comparator.comparingDouble(Pair::getRight))
      .get();
}
```

### 6.2.`ImmutableTriple`来自 Apache Commons Lang

[`ImmutableTriple`](https://web.archive.org/web/20220913170115/https://commons.apache.org/proper/commons-lang/javadocs/api-3.9/org/apache/commons/lang3/tuple/ImmutableTriple.html) 和`ImmutablePair`很像。唯一不同的是，顾名思义，一个`ImmutableTriple`包含三个字段:`left`、`middle,`和`right.`

现在，让我们添加一个新的方法到我们的坐标计算中来展示如何使用`ImmutableTriple`类型。

我们将遍历`List<Coordinates>`中的所有点，找出到给定目标点的`min`、`avg,`和`max`距离。

让我们看看如何使用`ImmutableTriple`类用一个方法返回这三个值:

```java
ImmutableTriple<Double, Double, Double> getMinAvgMaxTriple(
  List<Coordinates> coordinatesList, Coordinates target) {
    List<Double> distanceList = coordinatesList.stream()
      .map(coordinates -> coordinates.calculateDistance(target))
      .collect(Collectors.toList());
    Double minDistance = distanceList.stream().mapToDouble(Double::doubleValue).min().getAsDouble();
    Double avgDistance = distanceList.stream().mapToDouble(Double::doubleValue).average().orElse(0.0D);
    Double maxDistance = distanceList.stream().mapToDouble(Double::doubleValue).max().getAsDouble();

    return ImmutableTriple.of(minDistance, avgDistance, maxDistance);
}
```

## 7.结论

在本文中，我们学习了如何使用数组、集合、容器和元组从一个方法中返回多个值。我们可以在简单的情况下使用数组和集合，因为它们包装了单一的数据类型。

另一方面，容器和元组在创建复杂类型时很有用，容器提供了更好的可读性。

我们还了解到一些第三方库已经实现了 pair 和 triple 类型，并从 Apache Commons Lang 库中看到了一些例子。

和往常一样，这篇文章的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220913170115/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-2)