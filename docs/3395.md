# 在 Java 中使用 Pairs

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-pairs>

## 1。概述

在这个快速教程中，我们讨论了非常有用的编程概念，称为`Pair`。`Pairs`提供了一种处理简单键值关联的便捷方式，当我们想从一个方法中返回两个值时，这种方式特别有用。

核心 Java 库中有一个简单的`Pair`实现。除此之外，某些第三方库，如 Apache Commons 和 Vavr，已经在其各自的 API 中公开了这一功能。

## 延伸阅读:

## [幕后的 Java 散列表](/web/20220808151525/https://www.baeldung.com/java-hashmap-advanced)

A quick and practical guide to Hashmap's internals[Read more](/web/20220808151525/https://www.baeldung.com/java-hashmap-advanced) →

## [在 Java 中迭代地图](/web/20220808151525/https://www.baeldung.com/java-iterate-map)

Learn different ways of iterating through the entries of a Map in Java.[Read more](/web/20220808151525/https://www.baeldung.com/java-iterate-map) →

## [Java–合并多个集合](/web/20220808151525/https://www.baeldung.com/java-combine-multiple-collections)

A quick and practical guide to combining multiple collections in Java[Read more](/web/20220808151525/https://www.baeldung.com/java-combine-multiple-collections) →

## 2。核心 Java 实现

### 2.1。`Pair`班

我们可以在`javafx.util`包中找到`Pair`类。这个类的构造函数有两个参数，一个键和它对应的值:

```
Pair<Integer, String> pair = new Pair<>(1, "One");
Integer key = pair.getKey();
String value = pair.getValue(); 
```

这个例子说明了一个使用 Pair 概念的简单的`Integer`到`String`的映射。

如图所示，`pair`对象中的键是通过调用`getKey()`方法来检索的，而值是通过调用`getValue().`来检索的

### 2.2。`AbstractMap.SimpleEntry`和`AbstractMap.SimpleImmutableEntry`

`SimpleEntry`被定义为`AbstractMap`类中的嵌套类。要创建这种类型的对象，我们可以向构造函数提供一个键和值:

```
AbstractMap.SimpleEntry<Integer, String> entry 
  = new AbstractMap.SimpleEntry<>(1, "one");
Integer key = entry.getKey();
String value = entry.getValue();
```

可以通过标准的 getter 和 setter 方法访问键和值。

此外，`AbstractMap`类还包含一个表示不可变对的嵌套类，即`SimpleImmutableEntry`类:

```
AbstractMap.SimpleImmutableEntry<Integer, String> entry
  = new AbstractMap.SimpleImmutableEntry<>(1, "one");
```

这与可变 pair 类的工作方式类似，只是 pair 的值不能改变。试图这样做将导致`UnsupportedOperationException`。

## 3 .Apache common〔t1〕

在 Apache Commons 库中，我们可以在`org.apache.commons.lang3.tuple` 包中找到`Pair`类。这是一个抽象类，所以不能直接实例化。

这里我们可以找到代表不可变和可变对的两个子类，`Imm` `utablePair` 和`MutablePair.`

这两种实现都可以访问键/值 getter/setter 方法:

```
ImmutablePair<Integer, String> pair = new ImmutablePair<>(2, "Two");
Integer key = pair.getKey();
String value = pair.getValue();
```

**不出所料，试图在`ImmutablePair` 上调用`setValue()` 会导致`UnsupportedOperationException.`**

但是，该操作对可变实现完全有效:

```
Pair<Integer, String> pair = new MutablePair<>(3, "Three");
pair.setValue("New Three"); 
```

## 4。Vavr

在 Vavr 库中，pair 功能由不可变的`Tuple2`类提供:

```
Tuple2<Integer, String> pair = new Tuple2<>(4, "Four");
Integer key = pair._1();
String value = pair._2(); 
```

在这个实现中，我们不能在创建后修改对象，所以变异方法返回一个新的实例，该实例包含所提供的更改:

```
tuplePair = pair.update2("New Four"); 
```

## 5。备选方案 I–简单容器类

无论是根据用户偏好还是在没有任何上述库的情况下，pair 功能的标准解决方法是创建一个简单的容器类来包装所需的返回值。

这里最大的优点是能够提供我们的名字，这有助于避免用同一个类表示不同的域对象:

```
public class CustomPair {
    private String key;
    private String value;

    // standard getters and setters
}
```

## 6。 **方案二——阵列**

另一种常见的解决方法是使用具有两个元素的简单数组来实现类似的结果:

```
private Object[] getPair() {
    // ...
    return new Object[] {key, value};
}
```

通常，键位于数组的索引零处，而其对应的值位于索引一处。

## 7。结论

在本文中，我们讨论了 Java 中的`Pairs`概念，以及 core Java 和其他第三方库中可用的不同实现。

和往常一样，这篇文章的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220808151525/https://github.com/eugenp/tutorials/tree/master/libraries-4)