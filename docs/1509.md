# 如何将一个流拆分成多个流

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-split-stream>

## 1.概观

Java 的 Streams API 是处理数据的强大而通用的工具。根据定义，流操作是对一组数据的单次迭代。

然而，有时我们希望以不同的方式处理流的各个部分，并获得多组结果。

在本教程中，我们将学习如何将一个流分成多个组并独立处理它们。

## 2.使用收集器

**一个[流](/web/20220810180543/https://www.baeldung.com/java-streams)应该操作一次，并且有一个终端操作。**它可以有多个中间操作，但是在关闭之前只能收集一次数据。

这意味着 Streams API 规范明确禁止分流和每个分流有不同的中间操作。这将导致多个终端操作。但是，我们可以在终端操作内部拆分流。这将产生一个分成两个或更多组的结果。

### 2.1.用`partitioningBy`进行二进制分割

如果我们想把一个流分成两部分，我们可以使用来自`Collectors`类的`partitioningBy`。它接受一个`Predicate`并返回一个`Map`，将满足谓词的元素分组到`Boolean` `true`键下，其余的元素分组到`false`键下。

假设我们有一个文章列表，其中包含了关于它们应该被发布到的目标站点以及它们是否应该被展示的信息。

```
List<Article> articles = Lists.newArrayList(
  new Article("Baeldung", true),
  new Article("Baeldung", false),
  new Article("Programming Daily", false),
  new Article("The Code", false));
```

我们将它分成两组，一组只包含 Baeldung 文章，另一组包含其余的文章:

```
Map<Boolean, List<Article>> groupedArticles = articles.stream()
  .collect(Collectors.partitioningBy(a -> a.target.equals("Baeldung"))); 
```

让我们看看哪些文章被归档在地图中的`true`和`false`键下:

```
assertThat(groupedArticles.get(true)).containsExactly(
  new Article("Baeldung", true),
  new Article("Baeldung", false));
assertThat(groupedArticles.get(false)).containsExactly(
  new Article("Programming Daily", false),
  new Article("The Code", false));
```

### 2.2.用`groupingBy`分割

如果我们想要有更多的类别，那么我们需要使用`groupingBy`方法。它需要一个函数将每个元素分类到一个组中。然后它返回一个`Map`，将每个组分类器链接到它的元素集合。

假设我们想按目标站点对文章进行分组。返回的`Map`将具有包含站点名称的键和包含与给定站点相关的文章集合的值:

```
Map<String, List<Article>> groupedArticles = articles.stream()
  .collect(Collectors.groupingBy(a -> a.target));
assertThat(groupedArticles.get("Baeldung")).containsExactly(
  new Article("Baeldung", true),
  new Article("Baeldung", false));
assertThat(groupedArticles.get("Programming Daily")).containsExactly(new Article("Programming Daily", false));
assertThat(groupedArticles.get("The Code")).containsExactly(new Article("The Code", false));
```

## 3.使用`teeing`

从 Java 12 开始，我们有了另一个二进制分割选项。我们可以使用`teeing`收集器。 **`teeing`将两个收藏家合二为一。**每个元素都由它们处理，然后使用提供的合并函数合并成一个返回值。

### 3.1.`teeing`用一个`Predicate`

**`teeing`收集器与来自`Collectors`类的另一个收集器`filtering`很好地配对。**它接受一个谓词，用它来过滤已处理的元素，然后将它们传递给另一个收集器。

让我们把文章分成 Baeldung 和非 Baeldung 两类来统计一下。我们还将使用`List`构造函数作为合并函数:

```
List<Long> countedArticles = articles.stream().collect(Collectors.teeing(
  Collectors.filtering(article -> article.target.equals("Baeldung"), Collectors.counting()),
  Collectors.filtering(article -> !article.target.equals("Baeldung"), Collectors.counting()),
  List::of));
assertThat(countedArticles.get(0)).isEqualTo(2);
assertThat(countedArticles.get(1)).isEqualTo(2);
```

### 3.2.`teeing`有重叠结果

这个解决方案和以前的解决方案有一个重要的区别。我们之前创建的组没有重叠，来自源流的每个元素最多属于一个组。有了`teeing,`,我们不再受这个限制的约束，因为每个收集器都可能处理整个流。让我们看看如何利用它。

我们可能希望将文章分成两组，一组只包含特色文章，另一组只包含 Baeldung 文章。由于一篇文章可以同时在 Baeldung 上被特征化和定向，因此得到的文章集可能重叠。

这次我们不再计数，而是将它们收集到列表中:

```
List<List<Article>> groupedArticles = articles.stream().collect(Collectors.teeing(
  Collectors.filtering(article -> article.target.equals("Baeldung"), Collectors.toList()),
  Collectors.filtering(article -> article.featured, Collectors.toList()),
  List::of));

assertThat(groupedArticles.get(0)).hasSize(2);
assertThat(groupedArticles.get(1)).hasSize(1);

assertThat(groupedArticles.get(0)).containsExactly(
  new Article("Baeldung", true),
  new Article("Baeldung", false));
assertThat(groupedArticles.get(1)).containsExactly(new Article("Baeldung", true)); 
```

## 4.使用 RxJava

虽然 Java 的 Streams API 是一个有用的工具，但有时它还不够。其他的解决方案，比如由 [RxJava](/web/20220810180543/https://www.baeldung.com/rx-java) 提供的反应流，也许能够帮助我们。让我们看一个简短的例子，看看我们如何使用一个`Observable`和多个`Subscribers`来获得与我们的`Stream`例子相同的结果。

### 4.1.创建一个`Observable`

**首先，我们需要从文章列表中创建一个`Observable`实例。**我们可以使用`Observable`类的`from`工厂方法:

```
Observable<Article> observableArticles = Observable.from(articles);
```

### 4.2.过滤`Observables`

接下来，我们需要创建过滤文章的`Observables`。为此，我们将使用来自`Observable `类的`filter`方法:

```
Observable<Article> baeldungObservable = observableArticles.filter(
  article -> article.target.equals("Baeldung"));
Observable<Article> featuredObservable = observableArticles.filter(
  article -> article.featured);
```

### 4.3.创建多个`Subscribers`

**最后，我们需要订阅`Observables`并提供一个`Action`来描述我们想要对文章做什么。**现实世界中的一个例子是将它们保存在数据库中或发送给客户端，但我们将满足于将它们添加到列表中:

```
List<Article> baeldungArticles = new ArrayList<>();
List<Article> featuredArticles = new ArrayList<>();
baeldungObservable.subscribe(baeldungArticles::add);
featuredObservable.subscribe(featuredArticles::add);
```

## 5.结论

在本教程中，我们学习了如何将流分成组并分别处理它们。首先，我们查看了旧的 Streams API 方法:`groupingBy`和`partitionBy`。接下来，我们使用了一种更新的方法，利用了 Java 12 中引入的`teeing`方法。最后，我们研究了如何使用 RxJava 以更大的灵活性实现类似的结果。

和往常一样，源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220810180543/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-streams-4)