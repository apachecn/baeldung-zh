# Java 8 流 API 教程

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-8-streams>

## 1。概述

在这个全面的教程中，我们将详细介绍 Java 8 流从创建到并行执行的实际应用。

为了理解这些材料，读者需要具备 Java 8 (lambda 表达式，`Optional,` 方法引用)和 Stream API 的基础知识。为了更熟悉这些话题，请看看我们之前的文章:[Java 8 中的新特性](/web/20220718115601/https://www.baeldung.com/java-8-new-features)和[Java 8 流简介](/web/20220718115601/https://www.baeldung.com/java-8-streams-introduction)。

## 延伸阅读:

## [Lambda 表达式和函数接口:技巧和最佳实践](/web/20220718115601/https://www.baeldung.com/java-8-lambda-expressions-tips)

Tips and best practices on using Java 8 lambdas and functional interfaces.[Read more](/web/20220718115601/https://www.baeldung.com/java-8-lambda-expressions-tips) →

## [Java 8 的收集器指南](/web/20220718115601/https://www.baeldung.com/java-8-collectors)

The article discusses Java 8 Collectors, showing examples of built-in collectors, as well as showing how to build custom collector.[Read more](/web/20220718115601/https://www.baeldung.com/java-8-collectors) →

## 2。流创建

有许多方法可以创建不同源的流实例。一旦创建，实例**将不会修改它的源，**因此允许从单个源创建多个实例。

### 2.1。空流

在创建空流的情况下，我们应该使用 **`empty()`** 方法:

```
Stream<String> streamEmpty = Stream.empty();
```

我们经常在创建时使用`empty()` 方法，以避免为没有元素的流返回`null`:

```
public Stream<String> streamOf(List<String> list) {
    return list == null || list.isEmpty() ? Stream.empty() : list.stream();
}
```

### 2.2。`Collection` 【T2 之流】

我们也可以创建任何类型的`Collection` ( `Collection, List, Set`)流:

```
Collection<String> collection = Arrays.asList("a", "b", "c");
Stream<String> streamOfCollection = collection.stream();
```

### 2.3。数组流

数组也可以是流的源:

```
Stream<String> streamOfArray = Stream.of("a", "b", "c");
```

我们还可以从现有数组或数组的一部分创建一个流:

```
String[] arr = new String[]{"a", "b", "c"};
Stream<String> streamOfArrayFull = Arrays.stream(arr);
Stream<String> streamOfArrayPart = Arrays.stream(arr, 1, 3);
```

### 2.4。 `Stream.builder()`

**当使用生成器时，** **需要在语句的右边额外指定类型，**否则`build()`方法会创建一个`Stream<Object>:`的实例

```
Stream<String> streamBuilder =
  Stream.<String>builder().add("a").add("b").add("c").build();
```

### 2.5。 `Stream.generate()`

**`generate()`** 方法接受一个`Supplier<T>` 来生成元素。由于产生的流是无限的，开发人员应该指定所需的大小，否则`generate()`方法将一直工作到达到内存限制:

```
Stream<String> streamGenerated =
  Stream.generate(() -> "element").limit(10);
```

上面的代码用值`“element.”`创建了一个十个字符串的序列

### 2.6。 `Stream.iterate()`

另一种创建无限流的方法是使用 **`iterate()`** 方法:

```
Stream<Integer> streamIterated = Stream.iterate(40, n -> n + 2).limit(20);
```

结果流的第一个元素是`iterate()`方法的第一个参数。当创建每个后续元素时，指定的函数应用于前一个元素。在上面的例子中，第二个元素是 42。

### 2.7。原语流

Java 8 提供了用三种原语类型创建流的可能性:`int, long`和`double.`因为`Stream<T>`是一个泛型接口，没有办法使用原语作为泛型的类型参数，所以创建了三个新的特殊接口: **`IntStream, LongStream, DoubleStream.`**

使用新界面减少了不必要的自动装箱，从而提高了生产效率:

```
IntStream intStream = IntStream.range(1, 3);
LongStream longStream = LongStream.rangeClosed(1, 3);
```

**`range(int startInclusive, int endExclusive)`** 方法创建一个从第一个参数到第二个参数的有序流。它以等于 1 的步长递增后续元素的值。结果不包括最后一个参数，它只是序列的一个上限。

`**rangeClosed(int startInclusive, int endInclusive)** `方法做同样的事情，只有一点不同，包括了第二个元素。我们可以使用这两种方法来生成三种类型的原语流中的任何一种。

从 Java 8 开始， [`Random`](https://web.archive.org/web/20220718115601/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Random.html) 类为生成原语流提供了广泛的方法。例如，下面的代码创建了一个包含三个元素的`DoubleStream,` :

```
Random random = new Random();
DoubleStream doubleStream = random.doubles(3);
```

### 2.8。`String` 【T2 之流】

在`String`类的`chars()`方法的帮助下，我们也可以使用` String`作为创建流的源。由于在 JDK 没有`CharStream` 的接口，我们使用`IntStream`来表示字符流。

```
IntStream streamOfChars = "abc".chars();
```

以下示例根据指定的`RegEx`将`String` 分解成子字符串:

```
Stream<String> streamOfString =
  Pattern.compile(", ").splitAsStream("a, b, c");
```

### 2.9。文件流

此外，Java NIO 类`Files` 允许我们通过`lines()`方法生成文本文件的`Stream<String>`。文本的每一行都成为流的一个元素:

```
Path path = Paths.get("C:\\file.txt");
Stream<String> streamOfStrings = Files.lines(path);
Stream<String> streamWithCharset = 
  Files.lines(path, Charset.forName("UTF-8"));
```

可以将`Charset`指定为`lines()`方法的一个参数。

## 3。引用流

我们可以实例化一个流，并有一个对它的可访问的引用，只要只调用中间操作。执行终端操作会导致流不可访问`.`

为了证明这一点，我们将暂时忘记，最佳实践是将操作序列链接起来。除了不必要的冗长之外，从技术上讲，以下代码是有效的:

```
Stream<String> stream = 
  Stream.of("a", "b", "c").filter(element -> element.contains("b"));
Optional<String> anyElement = stream.findAny();
```

然而，在调用终端操作后试图重用同一引用将触发`IllegalStateException:`

```
Optional<String> firstElement = stream.findFirst();
```

由于`IllegalStateException`是一个 `RuntimeException`，编译器不会发出问题信号。所以记住 **Java 8** **流不能被重用是非常重要的。**

这种行为合乎逻辑。我们设计流是为了在函数式风格中将有限的操作序列应用于元素源，而不是存储元素。

因此，为了使前面的代码正常工作，应该进行一些更改:

```
List<String> elements =
  Stream.of("a", "b", "c").filter(element -> element.contains("b"))
    .collect(Collectors.toList());
Optional<String> anyElement = elements.stream().findAny();
Optional<String> firstElement = elements.stream().findFirst();
```

## 4。流管道

为了对数据源的元素执行一系列操作并聚合它们的结果，我们需要三个部分:源操作(T0)、中间操作(T2)和终端操作(T4)。

中间操作返回新的修改后的流。例如，要创建一个没有几个元素的现有流的新流，应该使用`skip()`方法:

```
Stream<String> onceModifiedStream =
  Stream.of("abcd", "bbcd", "cbcd").skip(1);
```

如果我们需要一个以上的修改，我们可以链接中间操作。假设我们还需要用前几个字符的子字符串替换当前`Stream<String>`的每个元素。我们可以通过链接 *skip()* 和 *map()* 方法来做到这一点:

```
Stream<String> twiceModifiedStream =
  stream.skip(1).map(element -> element.substring(0, 3));
```

正如我们所见，`map()`方法将 lambda 表达式作为参数。如果我们想了解更多关于 lambdas 的知识，我们可以看看我们的教程 [Lambda 表达式和函数接口:技巧和最佳实践](/web/20220718115601/https://www.baeldung.com/java-8-lambda-expressions-tips)。

一条小溪本身是没有价值的；用户对终端操作的结果感兴趣，它可以是某种类型的值或应用于流中每个元素的动作。每个流只能使用一个终端操作。

使用流的正确和最方便的方式是通过**流管道，它是由流源、中间操作和终端操作组成的链:**

```
List<String> list = Arrays.asList("abc1", "abc2", "abc3");
long size = list.stream().skip(1)
  .map(element -> element.substring(0, 3)).sorted().count();
```

## 5。惰性调用

**中级操作懒。**这意味着**它们只有在终端操作执行需要时才会被调用。**

例如，让我们调用方法 `wasCalled()` `,` ，它在每次被调用时递增一个内部计数器:

```
private long counter;

private void wasCalled() {
    counter++;
}
```

现在让我们从操作`filter()`中调用方法`wasCalled` `()`:

```
List<String> list = Arrays.asList(“abc1”, “abc2”, “abc3”);
counter = 0;
Stream<String> stream = list.stream().filter(element -> {
    wasCalled();
    return element.contains("2");
});
```

由于我们有一个三个元素的源，我们可以假设`filter()`方法将被调用三次，并且`counter` 变量的值将是 3。然而，运行这段代码根本不会改变`counter` ，它仍然是零，所以`filter()`方法甚至一次都没有被调用。缺少终端操作的原因。

让我们稍微重写一下代码，添加一个`map()`操作和一个终端操作，`findFirst().`我们还将添加在日志记录的帮助下跟踪方法调用顺序的能力:

```
Optional<String> stream = list.stream().filter(element -> {
    log.info("filter() was called");
    return element.contains("2");
}).map(element -> {
    log.info("map() was called");
    return element.toUpperCase();
}).findFirst();
```

生成的日志显示我们调用了两次`filter()`方法和一次`map()`方法。这是因为管道垂直执行。在我们的例子中，流的第一个元素不满足过滤器的谓词。然后我们调用第二个元素的`filter()`方法，它通过了过滤器。没有调用第三个元素的`filter()`，我们通过管道进入了`map()`方法。

`findFirst()`操作只满足一个元素。所以在这个特殊的例子中，惰性调用允许我们避免两次方法调用，一次是针对`filter()`的，一次是针对`map().`的

## 6。执行顺序

从性能的角度来看，**正确的顺序是流管道中链接操作最重要的方面之一:**

```
long size = list.stream().map(element -> {
    wasCalled();
    return element.substring(0, 3);
}).skip(2).count();
```

这段代码的执行将使计数器的值增加 3。这意味着我们调用了流的`map()`方法三次，但是`size`的值是 1。所以得到的流只有一个元素，我们无缘无故地执行了三次昂贵的`map()`操作中的两次。

如果我们改变`skip()` 和`map()`方法`,` 的顺序，`counter` 只会增加一个。所以我们将只调用一次`map()` 方法:

```
long size = list.stream().skip(2).map(element -> {
    wasCalled();
    return element.substring(0, 3);
}).count();
```

这给我们带来了下面的规则:**减少流大小的中间操作应该放在应用于每个元素的操作之前。**所以我们需要将 s `kip(), filter(),`和`distinct()`这样的方法放在流管道的顶部。

## 7。流减少

API 有许多终端操作，它们将流聚合到一个类型或一个原语:`count(), max(), min(),` 和 `sum().` 然而，这些操作根据预定义的实现工作。那么如果一个开发者需要定制一个流的缩减机制，那么**是什么呢？**有两种方法允许我们这样做，分别是`**reduce()**` 和 **`collect()`** 方法。

### 7.1。`reduce()`法

此方法有三种变体，它们的签名和返回类型各不相同。它们可以有以下参数:

**identity–**累加器的初始值，或者如果流为空并且没有要累加的内容，则为默认值

**累加器—**指定元素聚合逻辑的函数。由于累加器为每一步减少创建一个新值，新值的数量等于流的大小，只有最后一个值是有用的。这对表演不是很好。

**合并器—**一个聚集累加器结果的功能。我们只在并行模式下调用 combiner 来减少来自不同线程的累加器的结果。

现在让我们来看看这三种方法的实际应用:

```
OptionalInt reduced =
  IntStream.range(1, 4).reduce((a, b) -> a + b);
```

`reduced` = 6 (1 + 2 + 3)

```
int reducedTwoParams =
  IntStream.range(1, 4).reduce(10, (a, b) -> a + b);
```

`reducedTwoParams` = 16 (10 + 1 + 2 + 3)

```
int reducedParams = Stream.of(1, 2, 3)
  .reduce(10, (a, b) -> a + b, (a, b) -> {
     log.info("combiner was called");
     return a + b;
  });
```

结果将与上一个示例(16)相同，并且没有登录，这意味着没有调用 combiner。要使合并器工作，流应该是并行的:

```
int reducedParallel = Arrays.asList(1, 2, 3).parallelStream()
    .reduce(10, (a, b) -> a + b, (a, b) -> {
       log.info("combiner was called");
       return a + b;
    });
```

这里的结果是不同的(36)，合并器被调用了两次。这里，归约通过以下算法工作:累加器通过将流的每个元素加到`identity`中运行三次。这些行动是同时进行的。结果，他们有(10+1 = 11；10 + 2 = 12;10 + 3 = 13;).现在合并器可以合并这三个结果。为此需要两次迭代(12+13 = 25；25 + 11 = 36).

### 7.2。`collect()`法

流的缩减也可以由另一个终端操作执行，即`collect()`方法。它接受一个类型为`Collector,` 的参数，该参数指定了归约的机制。对于最常见的操作，已经创建了预定义的收集器。在`Collectors`类型的帮助下可以访问它们。

在本节中，我们将使用以下`List`作为所有流的源:

```
List<Product> productList = Arrays.asList(new Product(23, "potatoes"),
  new Product(14, "orange"), new Product(13, "lemon"),
  new Product(23, "bread"), new Product(13, "sugar"));
```

**将流转换到`Collection` ( *集合，列表*或*集合* ):**

```
List<String> collectorCollection = 
  productList.stream().map(Product::getName).collect(Collectors.toList());
```

**还原为*串* :**

```
String listToString = productList.stream().map(Product::getName)
  .collect(Collectors.joining(", ", "[", "]"));
```

`joiner()` 方法可以有一到三个参数(分隔符、前缀、后缀)。使用`joiner()`最方便的是，开发人员不需要检查流是否到达其末尾来应用后缀，也不需要应用分隔符。`Collector`会处理好的。

**处理流中所有数值元素的平均值:**

```
double averagePrice = productList.stream()
  .collect(Collectors.averagingInt(Product::getPrice));
```

**处理流中所有数值元素的和:**

```
int summingPrice = productList.stream()
  .collect(Collectors.summingInt(Product::getPrice));
```

方法`averagingXX(), summingXX()`和`summarizingXX()`可以使用原语(`int, long, double`)和它们的包装类(`Integer, Long, Double`)。这些方法的一个更强大的特性是提供映射。结果，开发者不需要在`collect()`方法之前使用额外的`map()`操作。

**收集流元素的统计信息:**

```
IntSummaryStatistics statistics = productList.stream()
  .collect(Collectors.summarizingInt(Product::getPrice));
```

通过使用类型`IntSummaryStatistics`的结果实例，开发人员可以通过应用`toString()`方法来创建统计报告。结果将是一个与这个`“IntSummaryStatistics{count=5, sum=86, min=13, average=17,200000, max=23}.”`相同的`String`

通过应用`getCount(), getSum(), getMin(), getAverage(),` 和 `getMax().` 方法，从该对象中提取`count, sum, min,` 和 `average`的单独值也很容易。所有这些值都可以从单个管道中提取。

**根据指定函数对流的元素进行分组:**

```
Map<Integer, List<Product>> collectorMapOfLists = productList.stream()
  .collect(Collectors.groupingBy(Product::getPrice));
```

在上面的例子中，流被简化为`Map`，它根据价格对所有产品进行分组。

**根据一些谓词将流的元素分组:**

```
Map<Boolean, List<Product>> mapPartioned = productList.stream()
  .collect(Collectors.partitioningBy(element -> element.getPrice() > 15));
```

**推动收集器执行附加转换:**

```
Set<Product> unmodifiableSet = productList.stream()
  .collect(Collectors.collectingAndThen(Collectors.toSet(),
  Collections::unmodifiableSet));
```

在这个特殊的例子中，收集器将一个流转换成了一个`Set`，然后用它创建了一个不可改变的`Set`。

**自定义收集器:**

如果出于某种原因需要创建自定义收集器，那么最简单、最不冗长的方法就是使用类型为`Collector.`的方法`of()`

```
Collector<Product, ?, LinkedList<Product>> toLinkedList =
  Collector.of(LinkedList::new, LinkedList::add, 
    (first, second) -> { 
       first.addAll(second); 
       return first; 
    });

LinkedList<Product> linkedListOfPersons =
  productList.stream().collect(toLinkedList);
```

在这个例子中，`Collector`的一个实例被简化为`LinkedList` <人员>。

## 8。并行流

在 Java 8 之前，并行化是复杂的。 [`ExecutorService`](/web/20220718115601/https://www.baeldung.com/java-executor-service-tutorial) 和`[ForkJoin](/web/20220718115601/https://www.baeldung.com/java-fork-join)` 的出现稍微简化了开发人员的生活，但是仍然值得记住如何创建一个特定的执行程序，如何运行它，等等。Java 8 引入了一种以函数方式实现并行的方法。

API 允许我们创建并行流，以并行模式执行操作。当流的来源是一个*集合*或一个*数组*时，可以借助 ***parallelStream()*** 方法来实现:

```
Stream<Product> streamOfCollection = productList.parallelStream();
boolean isParallel = streamOfCollection.isParallel();
boolean bigPrice = streamOfCollection
  .map(product -> product.getPrice() * 12)
  .anyMatch(price -> price > 200);
```

如果流的来源不是`Collection` 或`array`，则应使用 **`parallel()`** 方法:

```
IntStream intStreamParallel = IntStream.range(1, 150).parallel();
boolean isParallel = intStreamParallel.isParallel();
```

在幕后，Stream API 自动使用`ForkJoin` 框架并行执行操作。默认情况下，将使用公共线程池，并且没有办法(至少目前没有)为它分配一些自定义线程池。[这可以通过使用一组定制的并行收集器来解决。](https://web.archive.org/web/20220718115601/https://github.com/pivovarit/parallel-collectors)

在并行模式下使用流时，避免阻塞操作。当任务需要相似的执行时间时，最好使用并行模式。如果一个任务持续的时间比另一个长得多，它会减慢整个应用程序的工作流程。

使用`sequential()`方法可以将并行模式下的流转换回顺序模式:

```
IntStream intStreamSequential = intStreamParallel.sequential();
boolean isParallel = intStreamSequential.isParallel();
```

## 9。结论

Stream API 是一个强大但易于理解的工具集，用于处理元素序列。如果使用得当，它可以让我们减少大量的样板代码，创建更多可读的程序，并提高应用程序的生产率。

在本文展示的大多数代码示例中，我们没有使用流(我们没有应用`close()`方法或终端操作)。在真正的应用程序中，**不要让实例化的流未被使用，因为这将导致内存泄漏。**

本文附带的完整代码示例可以在 GitHub 上获得。