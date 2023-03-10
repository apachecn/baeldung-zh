# 流 API 中的 mapMulti 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-mapmulti>

## 1.概观

在本教程中，我们将回顾 Java 16 中引入的方法`Stream::mapMulti`。我们将编写简单的例子来说明如何使用它。特别是**我们会看到这种方法类似于`Stream::` `flatMap`** 。我们将介绍在什么情况下我们更喜欢使用`mapMulti`而不是`flatMap` 。

请务必查看我们关于 [`Java Streams`](/web/20221208143909/https://www.baeldung.com/java-streams) 的文章，以便更深入地了解流 API。

## 2.方法签名

省略通配符，`mapMulti`方法可以更简洁地编写:

```java
<R> Stream<R> mapMulti​(BiConsumer<T, Consumer<R>> mapper)
```

是个`Stream`中级操作。它需要一个`BiConsumer` 功能接口的实现作为参数。**`BiConsumer`的实现取一个`Stream`元素`T`，如果需要，将其转换成`type` `R`，并调用`mapper'` s `Consumer::accept`。**

在 Java 的`mapMulti`方法实现中，`mapper`是实现`Consumer` 功能接口的缓冲区。

每次我们调用`Consumer::accept,`时，它都会在缓冲区中累积元素，并将它们传递给流管道。

## 3.简单的实现示例

让我们考虑对一个整数列表做如下操作:

```java
List<Integer> integers = Arrays.asList(1, 2, 3, 4, 5);
double percentage = .01;
List<Double> evenDoubles = integers.stream()
  .<Double>mapMulti((integer, consumer) -> {
    if (integer % 2 == 0) {
        consumer.accept((double) integer * ( 1 + percentage));
    }
  })
  .collect(toList());
```

在我们对`BiConsumer<T, Consumer<R>> mapper`的 lambda 实现中，我们首先只选择偶数整数，然后将它们加上`percentage`中指定的数量，将结果转换为`double,`，并完成对`consumer.accept`的调用。

正如我们之前看到的，`consumer`只是一个将返回元素传递给流管道的缓冲区。(作为旁注，注意我们必须为返回值使用类型见证`<Double>mapMulti` ,因为否则，编译器无法推断出方法签名中`R`的正确类型。)

根据元素是奇数还是偶数，这可以是一对一的转换。

请注意，前面的代码示例中的`if statement`扮演了一个`Stream::filter`的角色，并将整数转换为一个双精度数，这是一个`Stream::map`的角色。因此，我们可以使用`Stream's` `filter`和`map`来实现相同的结果:

```java
List<Integer> integers = Arrays.asList(1, 2, 3, 4, 5);
double percentage = .01;
List<Double> evenDoubles = integers.stream()
  .filter(integer -> integer % 2 == 0)
  .<Double>map(integer -> ((double) integer * ( 1 + percentage)))
  .collect(toList());
```

然而， **`mapMulti` 的实现更加直接，因为我们不需要调用这么多的流中间操作**。

另一个优点是 **`mapMulti`的实现是必须的，这给了我们更多的自由来进行元素转换**。

为了支持`int`、`long`和`double`原始类型，我们有`mapMulti`的`mapMultiToDouble`、`mapMultiToInt,`和`mapMultiToLong`变体。

例如，我们可以使用`mapMultiToDouble`来计算前`List`个 doubles 的和:

```java
List<Integer> integers = Arrays.asList(1, 2, 3, 4, 5);
double percentage = .01;
double sum = integers.stream()
  .mapMultiToDouble((integer, consumer) -> {
    if (integer % 2 == 0) {
        consumer.accept(integer * (1 + percentage));
    }
  })
  .sum();
```

## 4.更现实的例子

让我们考虑一组`Album` s:

```java
public class Album {

    private String albumName;
    private int albumCost;
    private List<Artist> artists;

    Album(String albumName, int albumCost, List<Artist> artists) {
        this.albumName = albumName;
        this.albumCost = albumCost;
        this.artists = artists;
    }
    // ...
}
```

每个`Album`都有一个`Artist`列表:

```java
public class Artist {

    private final String name;
    private boolean associatedMajorLabels;
    private List<String> majorLabels;

    Artist(String name, boolean associatedMajorLabels, List<String> majorLabels) {
        this.name = name;
        this.associatedMajorLabels = associatedMajorLabels;
        this.majorLabels = majorLabels;
    }
    // ...
}
```

如果我们想要收集艺术家-专辑名称对的列表，我们可以使用`mapMulti`来实现它:

```java
List<Pair<String, String>> artistAlbum = albums.stream()
  .<Pair<String, String>> mapMulti((album, consumer) -> {
      for (Artist artist : album.getArtists()) {
          consumer.accept(new ImmutablePair<String, String>(artist.getName(), album.getAlbumName()));
      }
  })
```

对于流中的每个专辑，我们迭代艺术家，创建艺术家专辑名称的 Apache Commons `ImmutablePair`，并调用 C `onsumer::accept`。`mapMulti`的实现积累了消费者接受的元素，并将它们传递给流管道。

这具有一对多转换的效果，其中结果在消费者中累积，但最终被平铺到新的流中。这基本上就是`Stream::flatMap`所做的，因此我们可以通过以下实现获得相同的结果:

```java
List<Pair<String, String>> artistAlbum = albums.stream()
  .flatMap(album -> album.getArtists()
      .stream()
      .map(artist -> new ImmutablePair<String, String>(artist.getName(), album.getAlbumName())))
  .collect(toList()); 
```

我们看到两种方法给出了相同的结果。接下来我们将讨论在哪些情况下使用`mapMulti`更有利。

## 5.什么时候用`mapMulti`代替`flatMap`

### 5.1.用少量元素替换流元素

如 Java 文档中所述:“当用少量(可能为零)元素替换每个流元素时。使用这种方法避免了为每组结果元素创建一个新的`Stream`实例的开销，正如`flatMap”.`所要求的

让我们写一个简单的例子来说明这个场景:

```java
int upperCost = 9;
List<Pair<String, String>> artistAlbum = albums.stream()
  .<Pair<String, String>> mapMulti((album, consumer) -> {
    if (album.getAlbumCost() < upperCost) {
        for (Artist artist : album.getArtists()) {
            consumer.accept(new ImmutablePair<String, String>(artist.getName(), album.getAlbumName()));
      }
    }
  })
```

对于每张专辑，我们迭代艺术家，根据专辑的价格与变量`upperCost`的比较，累积零个或几个艺术家-专辑对。

使用`flatMap`完成相同的结果:

```java
int upperCost = 9;
List<Pair<String, String>> artistAlbum = albums.stream()
  .flatMap(album -> album.getArtists()
    .stream()
    .filter(artist -> upperCost > album.getAlbumCost())
    .map(artist -> new ImmutablePair<String, String>(artist.getName(), album.getAlbumName())))
  .collect(toList());
```

我们看到**`mapMulti`的命令式实现更具性能——我们不必像使用`flatMap`** 的声明式方法那样为每个处理过的元素创建中间流。

### 5.2.当更容易生成结果元素时

让我们在`Album`类中编写一个方法，将所有艺术家-专辑对及其相关的主要标签传递给消费者:

```java
public class Album {

    //...
    public void artistAlbumPairsToMajorLabels(Consumer<Pair<String, String>> consumer) {

        for (Artist artist : artists) {
            if (artist.isAssociatedMajorLabels()) {
                String concatLabels = artist.getMajorLabels().stream().collect(Collectors.joining(","));
                consumer.accept(new ImmutablePair<>(artist.getName()+ ":" + albumName, concatLabels));
            }
        }
    }
    // ...
}
```

如果艺术家与主要标签有关联，则实现会将标签连接成一个逗号分隔的字符串。然后它创建一对带有标签的艺术家专辑名，并调用 C `onsumer::accept`。

如果我们想获得所有对的列表，那么使用带有方法引用`Album::artistAlbumPairsToMajorLabels`的`mapMulti`就很简单:

```java
List<Pair<String, String>> copyrightedArtistAlbum = albums.stream()
  .<Pair<String, String>> mapMulti(Album::artistAlbumPairsToMajorLabels)
  .collect(toList());
```

我们看到，在更复杂的情况下，我们可以有非常复杂的方法引用实现。例如， [Java 文档](https://web.archive.org/web/20221208143909/https://docs.oracle.com/en/java/javase/16/docs/api/java.base/java/util/stream/Stream.html#mapMulti(java.util.function.BiConsumer))给出了一个使用递归的例子。

一般来说，使用 f `latMap`复制相同的结果会非常困难。因此，在生成结果元素比按照 `**flatMap**.`中的要求以`Stream`的形式返回结果元素容易得多的情况下，我们应该**使用`mapMulti`**

## 6.结论

在本教程中，我们已经用不同的例子介绍了如何实现`mapMulti`。我们已经看到了它与`flatMap`的比较，以及何时使用它更有利。

特别是，当一些流元素需要替换时，或者当使用命令式方法生成流管道的元素更容易时，建议使用`mapMulti`。

源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221208143909/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-16)