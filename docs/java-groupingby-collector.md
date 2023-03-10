# Java 8 分组指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-groupingby-collector>

## 1。简介

在本教程中，**我们将通过各种例子来看看`groupingBy`收集器是如何工作的。**

为了理解本教程中的内容，我们需要 Java 8 特性的基础知识。我们可以看看 Java 8 流的介绍和 Java 8 收集器的 T2 指南。

## 延伸阅读:

## [将 Java 流收集到不可变集合](/web/20220729193215/https://www.baeldung.com/java-stream-immutable-collection)

了解如何将 Java 流收集到不可变集合。[阅读更多](/web/20220729193215/https://www.baeldung.com/java-stream-immutable-collection)→

## [Java 8 Collectors toMap](/web/20220729193215/https://www.baeldung.com/java-collectors-tomap)

学习如何使用 Collectors 类的 toMap()方法。[阅读更多](/web/20220729193215/https://www.baeldung.com/java-collectors-tomap)→2.10。聚合分组结果的多个属性

## 2。`groupingBy` 收藏家

Java 8 `Stream` API 让我们以声明的方式处理数据集合。

静态工厂方法`Collectors.groupingBy()`和`Collectors.groupingByConcurrent()`为我们提供了类似于 SQL 语言中的“`GROUP BY'`子句的功能。**我们使用它们按照属性对对象进行分组，并将结果存储在一个`Map`实例中。**

`groupingBy `的重载方法有:

*   首先，使用分类函数作为方法参数:

```java
static <T,K> Collector<T,?,Map<K,List<T>>> 
  groupingBy(Function<? super T,? extends K> classifier)
```

*   其次，以分类函数和第二收集器为方法参数:

```java
static <T,K,A,D> Collector<T,?,Map<K,D>>
  groupingBy(Function<? super T,? extends K> classifier, 
    Collector<? super T,A,D> downstream)
```

*   最后，使用分类函数、供应商方法(提供包含最终结果的`Map`实现)和第二个收集器作为方法参数:

```java
static <T,K,D,A,M extends Map<K,D>> Collector<T,?,M>
  groupingBy(Function<? super T,? extends K> classifier, 
    Supplier<M> mapFactory, Collector<? super T,A,D> downstream)
```

### 2.1。代码设置示例

为了演示`groupingBy()`的用法，让我们定义一个`BlogPost`类(我们将使用一串`BlogPost`对象):

```java
class BlogPost {
    String title;
    String author;
    BlogPostType type;
    int likes;
} 
```

接下来，`BlogPostType`:

```java
enum BlogPostType {
    NEWS,
    REVIEW,
    GUIDE
} 
```

然后是`BlogPost`对象的`List`:

```java
List<BlogPost> posts = Arrays.asList( ... );
```

让我们也定义一个`Tuple`类，它将用于通过组合帖子的`type`和`author`属性来对帖子进行分组:

```java
class Tuple {
    BlogPostType type;
    String author;
} 
```

### 2.2。通过单个列进行简单分组

先说最简单的`groupingBy`方法，它只取一个分类函数作为它的参数。分类函数被应用于流的每个元素。

我们使用函数返回的值作为从`groupingBy`收集器获得的映射的键。

要将博客文章列表中的博客文章按其`type`分组:

```java
Map<BlogPostType, List<BlogPost>> postsPerType = posts.stream()
  .collect(groupingBy(BlogPost::getType)); 
```

### 2.3。`groupingBy`用复杂的`Map`键键入

分类函数不限于只返回标量或字符串值。只要我们确保实现了必要的`equals`和`hashcode`方法，结果图的键可以是任何对象。

对于使用两个字段作为键的**组，我们可以使用`javafx.util`或`org.apache.commons.lang3.tuple `包**中提供的`Pair`类。

例如，按照 Apache Commons `Pair`实例中组合的类型和作者对列表中的博客文章进行分组:

```java
Map<Pair<BlogPostType, String>, List<BlogPost>> postsPerTypeAndAuthor = posts.stream()
  .collect(groupingBy(post -> new ImmutablePair<>(post.getType(), post.getAuthor())));
```

类似地，我们可以使用之前定义的 Tuple 类，这个类可以很容易地被泛化以包含更多需要的字段。前面使用元组实例的示例是:

```java
Map<Tuple, List<BlogPost>> postsPerTypeAndAuthor = posts.stream()
  .collect(groupingBy(post -> new Tuple(post.getType(), post.getAuthor()))); 
```

Java 16 引入了 [`record`](/web/20220729193215/https://www.baeldung.com/java-record-keyword) 的概念，作为生成不可变 Java 类的新形式。

**`record`特性为我们提供了一种比元组更简单、更清晰、更安全的方式来完成`groupingBy`** 。例如，我们在`BlogPost`中定义了一个`record`实例:

```java
public class BlogPost {
    private String title;
    private String author;
    private BlogPostType type;
    private int likes;
    record AuthPostTypesLikes(String author, BlogPostType type, int likes) {};

    // constructor, getters/setters
} 
```

现在，使用`record`实例按照类型、作者和喜好将列表中的`BlotPost`分组非常简单:

```java
Map<BlogPost.AuthPostTypesLikes, List<BlogPost>> postsPerTypeAndAuthor = posts.stream()
  .collect(groupingBy(post -> new BlogPost.AuthPostTypesLikes(post.getAuthor(), post.getType(), post.getLikes()))); 
```

### 2.4。修改返回的`Map`值类型

`groupingBy`的第二个重载需要一个额外的第二个收集器(下游收集器),应用于第一个收集器的结果。

当我们指定一个分类函数，而不是一个下游收集器时，`toList()`收集器在幕后使用。

让我们使用`toSet()`收集器作为下游收集器，并获得一个`Set`博客帖子(而不是一个`List`):

```java
Map<BlogPostType, Set<BlogPost>> postsPerType = posts.stream()
  .collect(groupingBy(BlogPost::getType, toSet())); 
```

### 2.5。按多个字段分组

下游收集器的一个不同应用是对第一组 by 的结果进行二次`groupingBy`。

将`BlogPost`的`List`先按`author`分组，再按`type`分组:

```java
Map<String, Map<BlogPostType, List>> map = posts.stream()
  .collect(groupingBy(BlogPost::getAuthor, groupingBy(BlogPost::getType)));
```

### 2.6。获取分组结果的平均值

通过使用下游收集器，我们可以在分类函数的结果中应用聚合函数。

例如，要找到每篇博客文章`type`的平均数量`likes`:

```java
Map<BlogPostType, Double> averageLikesPerType = posts.stream()
  .collect(groupingBy(BlogPost::getType, averagingInt(BlogPost::getLikes))); 
```

### 2.7。从分组结果中获取总和

为了计算每个`type`的`likes`的总和:

```java
Map<BlogPostType, Integer> likesPerType = posts.stream()
  .collect(groupingBy(BlogPost::getType, summingInt(BlogPost::getLikes))); 
```

### 2.8。从分组结果中获取最大值或最小值

我们可以执行的另一个聚合是获取具有最大点赞数的博客帖子:

```java
Map<BlogPostType, Optional<BlogPost>> maxLikesPerPostType = posts.stream()
  .collect(groupingBy(BlogPost::getType,
  maxBy(comparingInt(BlogPost::getLikes)))); 
```

同样，我们可以应用`minBy`下游收集器来获取最少数量为`likes`的博文。

请注意，`maxBy`和`minBy`收集器考虑了它们所应用到的收集可能为空的可能性。这就是为什么映射中的值类型是`Optional<BlogPost>`。

### 2.9。获取分组结果属性的摘要

`Collectors` API 提供了一个汇总收集器，当我们需要同时计算一个数字属性的计数、总和、最小值、最大值和平均值时，可以使用这个收集器。

让我们为每个不同类型的博客帖子的 likes 属性计算一个摘要:

```java
Map<BlogPostType, IntSummaryStatistics> likeStatisticsPerType = posts.stream()
  .collect(groupingBy(BlogPost::getType, 
  summarizingInt(BlogPost::getLikes))); 
```

每种类型的`IntSummaryStatistics`对象包含`likes`属性的计数、总和、平均值、最小值和最大值。对于双精度值和长整型值，还存在其他汇总对象。

### 2.10。聚合分组结果的多个属性

在前面的章节中，我们已经看到了如何一次聚合一个字段。**我们可以遵循一些技术来对多个字段进行聚合**。

**第一种方法是使用`Collectors::collectingAndThen`作为`groupingBy`** 的下游集电极。对于`collectingAndThen` 的第一个参数，我们使用`Collectors::toList`将流收集到一个列表中。第二个参数应用完成转换，我们可以将它与任何支持聚合的`Collectors'`类方法一起使用，以获得我们想要的结果。

例如，让我们按`author`分组，并对每个分组计算`titles`的数量，列出`titles`，并提供`likes`的汇总统计。为了实现这一点，我们首先向`BlogPost`添加一条新记录:

```java
public class BlogPost {
    // ...
    record PostCountTitlesLikesStats(long postCount, String titles, IntSummaryStatistics likesStats){};
     // ...
}
```

`groupingBy`和`collectingAndThen`的实现将是:

```java
Map<String, BlogPost.PostCountTitlesLikesStats> postsPerAuthor = posts.stream()
  .collect(groupingBy(BlogPost::getAuthor, collectingAndThen(toList(), list -> {
    long count = list.stream()
      .map(BlogPost::getTitle)
      .collect(counting());
    String titles = list.stream()
      .map(BlogPost::getTitle)
      .collect(joining(" : "));
    IntSummaryStatistics summary = list.stream()
      .collect(summarizingInt(BlogPost::getLikes));
    return new BlogPost.PostCountTitlesLikesStats(count, titles, summary);
  }))); 
```

在 collectingAndThen 的第一个参数中我们得到了一个`BlogPos` `t`的列表。我们在最后的转换中使用它作为 lambda 函数的输入来计算生成`PostCountTitlesLikesStats`的值。

获取给定`author`的信息非常简单:

```java
BlogPost.PostCountTitlesLikesStats result = postsPerAuthor.get("Author 1");
assertThat(result.postCount()).isEqualTo(3L);
assertThat(result.titles()).isEqualTo("News item 1 : Programming guide : Tech review 2");
assertThat(result.likesStats().getMax()).isEqualTo(20);
assertThat(result.likesStats().getMin()).isEqualTo(15);
assertThat(result.likesStats().getAverage()).isEqualTo(16.666d, offset(0.001d)); 
```

**如果我们使用`Collectors::toMap`来收集和聚合流**的元素，我们还可以做更复杂的聚合。

让我们考虑一个简单的例子，我们想通过`author`将`BlogPost`元素分组，并将`titles`与`like`分数的上限和连接起来。

首先，我们创建将封装我们的聚合结果的记录:

```java
public class BlogPost {
    // ...
    record TitlesBoundedSumOfLikes(String titles, int boundedSumOfLikes) {};
    // ...
} 
```

然后，我们按以下方式对数据流进行分组和累积:

```java
int maxValLikes = 17;
Map<String, BlogPost.TitlesBoundedSumOfLikes> postsPerAuthor = posts.stream()
  .collect(toMap(BlogPost::getAuthor, post -> {
    int likes = (post.getLikes() > maxValLikes) ? maxValLikes : post.getLikes();
    return new BlogPost.TitlesBoundedSumOfLikes(post.getTitle(), likes);
  }, (u1, u2) -> {
    int likes = (u2.boundedSumOfLikes() > maxValLikes) ? maxValLikes : u2.boundedSumOfLikes();
    return new BlogPost.TitlesBoundedSumOfLikes(u1.titles().toUpperCase() + " : " + u2.titles().toUpperCase(), u1.boundedSumOfLikes() + likes);
  })); 
```

`toMap`的第一个参数对应用`BlogPost::getAuthor`的按键进行分组。

第二个参数使用 lambda 函数转换映射的值，将每个`BlogPost`转换成一个`TitlesBoundedSumOfLikes`记录。

`toMap`的第三个参数处理给定键的重复元素，这里我们使用另一个 lambda 函数连接`titles`并将`likes`与`maxValLikes`中指定的最大允许值相加。

### 2.11。将分组结果映射到不同类型

我们可以通过对分类函数的结果应用一个`mapping`下游收集器来实现更复杂的聚合。

让我们把每篇博文`type`的`title`串联起来:

```java
Map<BlogPostType, String> postsPerType = posts.stream()
  .collect(groupingBy(BlogPost::getType, 
  mapping(BlogPost::getTitle, joining(", ", "Post titles: [", "]")))); 
```

我们在这里所做的是将每个`BlogPost`实例映射到它的`title`，然后将文章标题流简化为一个串联的`String`。在这个例子中，`Map`值的类型也不同于默认的`List`类型。

### 2.11。修改返回`Map`类型

当使用`groupingBy`收集器时，我们不能假设返回的`Map`的类型。如果我们想明确从 group by 中获得哪种类型的`Map`，那么我们可以使用`groupingBy`方法的第三种变体，它允许我们通过传递一个`Map`供应商函数来改变`Map` 的类型。

让我们通过向`groupingBy`方法传递一个`EnumMap`供应商函数来检索一个`EnumMap`:

```java
EnumMap<BlogPostType, List<BlogPost>> postsPerType = posts.stream()
  .collect(groupingBy(BlogPost::getType, 
  () -> new EnumMap<>(BlogPostType.class), toList())); 
```

## 3。并发`groupingBy`收集者

与`groupingBy` 相似的是`groupingByConcurrent`收集器，它利用多核架构。这个收集器有三个重载方法，它们与`groupingBy`收集器各自的重载方法采用完全相同的参数。然而，`groupingByConcurrent`收集器的返回类型必须是`ConcurrentHashMap`类或其子类的一个实例。

要并发执行分组操作，流需要是并行的:

```java
ConcurrentMap<BlogPostType, List<BlogPost>> postsPerType = posts.parallelStream()
  .collect(groupingByConcurrent(BlogPost::getType)); 
```

如果我们选择将一个`Map`供应商函数传递给`groupingByConcurrent`收集器，那么我们需要确保该函数返回一个`ConcurrentHashMap`或者它的一个子类。

## 4。Java 9 新增功能

Java 9 引入了两个新的收集器，可以很好地与`groupingBy`一起工作；更多关于他们的信息可以在[这里](/web/20220729193215/https://www.baeldung.com/java9-stream-collectors)找到。

## 5。结论

在本文中，我们探索了 Java 8 `Collectors` API 提供的`groupingBy` 收集器的用法。

我们学习了如何使用`groupingBy`根据元素的属性对元素流进行分类，以及如何进一步收集、变异和简化分类结果，从而得到最终的容器。

本文中示例的完整实现可以在 GitHub 项目中找到。