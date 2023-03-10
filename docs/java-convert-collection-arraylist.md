# 在 Java 中将集合转换为数组列表

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-convert-collection-arraylist>

## 1。概述

将 Java 集合从一种类型转换成另一种类型是一项常见的编程任务。在本教程中，我们将把任何类型的`Collection`转换成`ArrayList`。

在整个教程中，我们将假设我们已经有了一个`Foo`对象的集合。从那里，我们将使用各种方法创建一个`ArrayList `。

## 2。定义我们的示例

但是在继续之前，让我们对输入和输出进行建模。

我们的源可以是任何类型的集合，所以我们将使用`Collection`接口来声明它:

```java
Collection<Foo> srcCollection; 
```

我们需要生成一个具有相同元素类型的`ArrayList`:

```java
ArrayList<Foo> newList;
```

## 3。使用 ArrayList 构造函数

将集合复制到新集合的最简单方法是使用它的构造函数。

在我们之前的[数组列表](/web/20221128045127/https://www.baeldung.com/java-arraylist)指南中，我们了解到`ArrayList`构造函数可以接受一个集合参数:

```java
ArrayList<Foo> newList = new ArrayList<>(srcCollection);
```

*   新的`ArrayList`包含源集合中 Foo 元素的浅层副本。
*   该顺序与源集合中的顺序相同。

构造函数的简单性使得它在大多数情况下都是一个很好的选择。

## 4。使用流 API

现在，**让我们利用 Streams API 从现有的集合** :中创建一个数组列表

```java
ArrayList<Foo> newList = srcCollection.stream().collect(toCollection(ArrayList::new));
```

在这个片段中:

*   我们从源集合中获取流，并应用`collect()`操作符来创建一个`List`
*   我们指定`ArrayList::new`来获得我们想要的列表类型
*   这段代码也将产生一个浅层拷贝。

如果我们不关心确切的`List`类型，我们可以简化:

```java
List<Foo> newList = srcCollection.stream().collect(toList());
```

注意`toCollection()`和`toList()`是从`Collectors`静态导入的。要了解更多，请参考我们关于 Java 8 的收集器的[指南。](/web/20221128045127/https://www.baeldung.com/java-8-collectors)

## 5。深层复制

在我们提到“浅抄”之前。我们的意思是，新列表中的**元素与源集合中仍然存在的`Foo`实例**完全相同。因此，我们通过引用将`Foo`复制到了`newList`。

如果我们在任一集合中修改一个`Foo`实例的内容，那么这个**修改将在两个集合**中反映出来。因此，如果我们想要修改集合`without`中的元素，修改另一个，我们需要执行“深度复制”

为了深度复制一个`Foo`，我们**为每个元素**创建一个全新的`Foo`实例。因此，所有的`Foo` 字段都需要复制到新的实例中。

让我们定义我们的`Foo`类，以便它知道如何深度复制自己:

```java
public class Foo {

    private int id;
    private String name;
    private Foo parent;

    public Foo(int id, String name, Foo parent) {
        this.id = id;
        this.name = name;
        this.parent = parent;
    }

    public Foo deepCopy() {
        return new Foo(
          this.id, this.name, this.parent != null ? this.parent.deepCopy() : null);
    }
}
```

这里我们可以看到字段`id`和`name` 分别是`int`和`String`。这些数据类型是按值复制的。因此，我们可以简单地将它们赋值。

`parent`字段是另一个`Foo`，是一个类。如果`Foo `发生了变异，任何共享该引用的代码都会受到这些变化的影响。**我们要深度复制`parent` 字段**。

现在我们可以回到我们的`ArrayList`转换。**我们只需要`map`操作符将深层副本**插入到流中:

```java
ArrayList<Foo> newList = srcCollection.stream()
  .map(foo -> foo.deepCopy())
  .collect(toCollection(ArrayList::new));
```

我们可以修改其中一个集合的内容，而不会影响另一个。

深度复制可能是一个漫长的过程，具体取决于元素的数量和数据的深度。如果需要的话，在这里使用并行流可以提高性能。

## 6。控制列表顺序

默认情况下，我们的流将按照在源集合中遇到的顺序将元素传递给我们的`ArrayList`。

如果我们想改变顺序**，我们可以将`sorted()`操作符应用于流**。要按名称对我们的`Foo`对象进行排序:

```java
ArrayList<Foo> newList = srcCollection.stream()
  .sorted(Comparator.comparing(Foo::getName))
  .collect(toCollection(ArrayList::new));
```

我们可以在之前的教程中找到更多关于流排序的细节。

## 7。结论

`ArrayList`构造函数是将一个`Collection`的内容放入一个新的`ArrayList`的有效方法。

但是，如果我们需要调整结果列表，Streams API 提供了一种修改流程的强大方法。

本文中使用的代码可以在 GitHub 上找到完整的[。](https://web.archive.org/web/20221128045127/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-conversions)