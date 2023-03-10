# 番石榴 19:有什么新鲜事？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/whats-new-in-guava-19>

## 1。概述

Google Guava 为库提供了简化 Java 开发的工具。在本教程中，我们将看看在[番石榴 19 版本](https://web.archive.org/web/20220521211352/https://github.com/google/guava/wiki/Release19)中引入的新功能。

## 2。`common.base`套餐变更

**2.1。添加了`CharMatcher`静态方法**

`CharMatcher`顾名思义，用于检查一个字符串是否匹配一组需求。

```java
String inputString = "someString789";
boolean result = CharMatcher.javaLetterOrDigit().matchesAllOf(inputString); 
```

在上面的例子中，`result`将是`true`。

`CharMatcher`也可以在需要变换字符串的时候使用。

```java
String number = "8 123 456 123";
String result = CharMatcher.whitespace().collapseFrom(number, '-'); 
```

在上例中，`result`将是“8-123-456-123”。

在`CharMatcher`的帮助下，您可以计算一个字符在给定字符串中出现的次数:

```java
String number = "8 123 456 123";
int result = CharMatcher.digit().countIn(number);
```

在上面的例子中，`result`将是 10。

以前版本的番石榴有匹配器常量，如`CharMatcher.WHITESPACE` 和`CharMatcher.JAVA_LETTER_OR_DIGIT`。

在番石榴 19 中，这些已经被等效的方法所取代(分别为`CharMatcher.whitespace()` 和`CharMatcher.javaLetterOrDigit()`)。这是为了减少使用`CharMatcher`时创建的类的数量。

使用静态工厂方法允许只在需要时创建类。在未来的版本中，matcher 常量将被弃用和删除。

**中的`2.2\. lazyStackTrace`法`Throwables`中的**

该方法返回所提供的`Throwable`的 stacktrace 元素(行)的`List`。如果只需要一部分，它会比遍历整个 stacktrace ( `Throwable.getStackTrace()`)更快，但是如果遍历整个 stacktrace，它会更慢。

```java
IllegalArgumentException e = new IllegalArgumentException("Some argument is incorrect");
List<StackTraceElement> stackTraceElements = Throwables.lazyStackTrace(e); 
```

## 3。`common.collect`套餐变更

**3.1。添加了`FluentIterable.toMultiset()`**

在之前的 Baeldung 文章中，[番石榴 18](/web/20220521211352/https://www.baeldung.com/whats-new-in-guava-18) 有什么新变化，我们看了一下`FluentIterable`。当您需要将一个`FluentIterable`转换成一个`ImmutableMultiSet`时，可以使用`toMultiset()`方法。

```java
User[] usersArray = {new User(1L, "John", 45), new User(2L, "Max", 15)};
ImmutableMultiset<User> users = FluentIterable.of(usersArray).toMultiset();
```

一个`Multiset`是集合，像`Set`一样，支持独立于顺序的等式。一个`Set`和一个`Multiset`的主要区别在于`Multiset`可能包含重复的元素。`Multiset`将相同的元素存储为同一个元素的出现次数，因此您可以调用`Multiset.count(java.lang.Object)`来获得给定对象的总出现次数。

让我们看几个例子:

```java
List<String> userNames = Arrays.asList("David", "Eugen", "Alex", "Alex", "David", "David", "David");

Multiset<String> userNamesMultiset = HashMultiset.create(userNames);

assertEquals(7, userNamesMultiset.size());
assertEquals(4, userNamesMultiset.count("David"));
assertEquals(2, userNamesMultiset.count("Alex"));
assertEquals(1, userNamesMultiset.count("Eugen"));
assertThat(userNamesMultiset.elementSet(), anyOf(containsInAnyOrder("Alex", "David", "Eugen")));
```

您可以很容易地确定重复元素的数量，这比使用标准 Java 集合要干净得多。

**3.2。增加了`RangeSet.asDescendingSetOfRanges()` 和`asDescendingMapOfRanges()`** 

`RangeSet`用于操作非空范围(区间)。我们可以将一个`RangeSet`描述为一组不相连的、非空的范围。当您向`RangeSet`添加新的非空范围时，任何连接的范围将被合并，空范围将被忽略:

让我们来看看一些可以用来构建新范围的方法:`Range.closed()`、`Range.openClosed()`、`Range.closedOpen()`、`Range.open()`。

它们之间的区别在于开放范围不包括它们的端点。它们在数学上有不同的名称。开放区间用“(”或“)”表示，而封闭区间用“[”或“]”表示。

例如(0，5)表示“大于 0 小于 5 的任何值”，而(0，5)表示“大于 0 小于等于 5 的任何值”:

```java
RangeSet<Integer> rangeSet = TreeRangeSet.create();
rangeSet.add(Range.closed(1, 10));
```

在这里，我们将 range [1，10]添加到我们的`RangeSet`中。现在，我们想通过增加新的产品系列来扩展它:

```java
rangeSet.add(Range.closed(5, 15));
```

你可以看到这两个范围在 5 处相连，所以`RangeSet`会将它们合并成一个新的单一范围，[1，15]:

```java
rangeSet.add(Range.closedOpen(10, 17));
```

这些范围在 10 处连接，因此它们将被合并，产生一个闭-开范围，[1，17]。您可以使用`contains`方法检查某个值是否包含在范围内:

```java
rangeSet.contains(15);
```

这将返回`true`，因为范围[1，17]包含 15。让我们试试另一个值:

```java
rangeSet.contains(17); 
```

这将返回`false`，因为 range [1，17]不包含它的上端点 17。您还可以使用`encloses`方法检查范围是否包含任何其他范围:

```java
rangeSet.encloses(Range.closed(2, 3));
```

这将返回`true`，因为范围[2，3]完全落在我们的范围[1，17]内。

还有几种方法可以帮助你进行区间操作，比如`Range.greaterThan()`、`Range.lessThan()`、`Range.atLeast()`、`Range.atMost()`。前两个将添加开放区间，后两个将添加封闭区间。例如:

```java
rangeSet.add(Range.greaterThan(22)); 
```

这会给你的`RangeSet`增加一个新的区间(22，+∞)，因为它和其他区间没有联系。

在新方法的帮助下，如`asDescendingSetOfRanges` (代表`RangeSet`)和`asDescendingMapOfRanges` (代表 ``RangeSet`` )你可以将`RangeSet`转换成`Set`或`Map`。

**3.3。增加了`Lists.cartesianProduct(List…)`和`Lists.cartesianProduct(List<List>>)`和**

笛卡尔积返回两个或更多集合的所有可能组合:

```java
List<String> first = Lists.newArrayList("value1", "value2");
List<String> second = Lists.newArrayList("value3", "value4");

List<List<String>> cartesianProduct = Lists.cartesianProduct(first, second);

List<String> pair1 = Lists.newArrayList("value2", "value3");
List<String> pair2 = Lists.newArrayList("value2", "value4");
List<String> pair3 = Lists.newArrayList("value1", "value3");
List<String> pair4 = Lists.newArrayList("value1", "value4");

assertThat(cartesianProduct, anyOf(containsInAnyOrder(pair1, pair2, pair3, pair4))); 
```

从这个例子中可以看出，结果列表将包含所提供列表的所有可能组合。

**3.4。添加了`Maps.newLinkedHashMapWithExpectedSize(int)`**

一个标准`LinkedHashMap`的初始大小是 16(你可以在`LinkedHashMap`的源码中验证这一点)。当它达到`HashMap`(默认为 0.75)的负载系数时，`HashMap`会重新散列并使其大小加倍。但是如果您知道您的`HashMap`将处理许多键-值对，您可以指定一个大于 16 的初始大小，这样可以避免重复的重散列:

```java
LinkedHashMap<Object, Object> someLinkedMap = Maps.newLinkedHashMapWithExpectedSize(512);
```

**3.5。重新添加了`Multisets.removeOccurrences(Multiset, Multiset)`**

该方法用于删除`Multiset`中的指定事件:

```java
Multiset<String> multisetToModify = HashMultiset.create();
Multiset<String> occurrencesToRemove = HashMultiset.create();

multisetToModify.add("John");
multisetToModify.add("Max");
multisetToModify.add("Alex");

occurrencesToRemove.add("Alex");
occurrencesToRemove.add("John");

Multisets.removeOccurrences(multisetToModify, occurrencesToRemove);
```

该操作后，`multisetToModify`中将只剩下“Max”。

注意，如果`multisetToModify`包含给定元素的多个实例，而`occurrencesToRemove`只包含该元素的一个实例，`removeOccurrences`将只删除一个实例。

## 4。common.hash 包更改

**4.1。添加了 Hashing.sha384()**

`Hashing.sha384()` 方法返回实现 SHA-384 算法的散列函数:

```java
int inputData = 15;

HashFunction hashFunction = Hashing.sha384();
HashCode hashCode = hashFunction.hashInt(inputData);
```

SHA-384 有 15 个是“0904 b 6277381 DCF bddd…2240 a 621 b 2 b5 E3 CDA 8”。

**4.2。增加了`Hashing.concatenating(HashFunction, HashFunction, HashFunction…)`和`Hashing.concatenating(Iterable<HashFunction>)`和**

在`Hashing.concatenating`方法的帮助下，您可以连接一系列散列函数的结果:

```java
int inputData = 15;

HashFunction crc32Function = Hashing.crc32();
HashCode crc32HashCode = crc32Function.hashInt(inputData);

HashFunction hashFunction = Hashing.concatenating(Hashing.crc32(), Hashing.crc32());
HashCode concatenatedHashCode = hashFunction.hashInt(inputData); 
```

得到的`concatenatedHashCode`将是“4acf27794acf2779”，与自身串接的`crc32HashCode`(“4 ACF 2779”)相同。

在我们的例子中，为了清楚起见，使用了单一的散列算法。然而，这并不是特别有用。当你需要使你的散列更强时，组合两个散列函数是有用的，因为只有当你的两个散列被破坏时，它才能被破坏。大多数情况下，使用两种不同的散列函数。

## 5。`common.reflect`套餐变更

**5.1。添加了`TypeToken.isSubtypeOf`**

`TypeToken` 用于在运行时操作和查询泛型类型，避免了由于类型擦除导致的[问题。](https://web.archive.org/web/20220521211352/https://github.com/google/guava/wiki/ReflectionExplained)

Java 在运行时不保留对象的泛型信息，所以不可能知道一个给定的对象是否有泛型。但是在反射的帮助下，您可以检测方法或类的一般类型。`TypeToken`使用这种变通方法，您无需额外的代码就可以处理和查询泛型类型。

在我们的例子中，你可以看到，如果没有`TypeToken`方法`isAssignableFrom`，将返回`true`，即使`ArrayList<String>`不能从`ArrayList<Integer>`赋值:

```java
ArrayList<String> stringList = new ArrayList<>();
ArrayList<Integer> intList = new ArrayList<>();
boolean isAssignableFrom = stringList.getClass().isAssignableFrom(intList.getClass());
```

要解决这个问题，我们可以借助`TypeToken`来检查这个。

```java
TypeToken<ArrayList<String>> listString = new TypeToken<ArrayList<String>>() { };
TypeToken<ArrayList<Integer>> integerString = new TypeToken<ArrayList<Integer>>() { };

boolean isSupertypeOf = listString.isSupertypeOf(integerString);
```

在本例中，`isSupertypeOf`将返回 false。

在以前的番石榴版本中，有一个用于此目的的方法`isAssignableFrom` ，但是从番石榴 19 开始，它被弃用，取而代之的是`isSupertypeOf`。此外，方法`isSubtypeOf(TypeToken)` 可以用来确定一个类是否是另一个类的子类型:

```java
TypeToken<ArrayList<String>> stringList = new TypeToken<ArrayList<String>>() { };
TypeToken<List> list = new TypeToken<List>() { };

boolean isSubtypeOf = stringList.isSubtypeOf(list);
```

`ArrayList`是`List`的一个子类型，所以结果将是`true`，正如所料。

## 6。`common.io`套餐变更

**6.1。添加了`ByteSource.sizeIfKnown()`** 

此方法返回源的字节大小(如果可以确定的话),而不打开数据流:

```java
ByteSource charSource = Files.asByteSource(file);
Optional<Long> size = charSource.sizeIfKnown();
```

**6.2。添加了`CharSource.length()`**

在以前的番石榴版本中，没有方法来确定一个`CharSource`的长度。现在你可以使用`CharSource.length()`来达到这个目的。

**6.3。添加了`CharSource.lengthIfKnown()`**

与`ByteSource,` 相同，但使用`CharSource.lengthIfKnown()`您可以确定文件的字符长度:

```java
CharSource charSource = Files.asCharSource(file, Charsets.UTF_8);
Optional<Long> length = charSource.lengthIfKnown(); 
```

## 7 .**。结论**

Guava 19 为其不断增长的库引入了许多有用的补充和改进。它非常值得考虑用于你的下一个项目。

本文中的代码示例可以在 [GitHub 资源库](https://web.archive.org/web/20220521211352/https://github.com/eugenp/tutorials/tree/master/guava-modules/guava-19)中找到。