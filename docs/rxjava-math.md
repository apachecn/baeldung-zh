# RxJava 中的数学和聚合运算符

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rxjava-math>

## 1.介绍

在 RxJava 文章的[介绍之后，我们将看看聚合和数学运算符。](/web/20221205161936/https://www.baeldung.com/rx-java)

**这些操作必须等待源`Observable`发出所有项目。**因此，在`Observables`上使用这些操作符是很危险的，因为它们可能代表很长或无限长的序列。

第二，所有的例子都使用了一个`TestSubscriber,` 的实例，一个可以用于单元测试的特定种类的`Subscriber`，来执行断言，检查接收到的事件或者包装一个被模仿的`Subscriber.`

现在，让我们开始看看数学运算符。

## 2.设置

为了使用额外的操作符，我们需要[将额外的依赖关系](https://web.archive.org/web/20221205161936/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22io.reactivex%22%20rxjava-math)添加到`pom.xml:`中

```java
<dependency>
    <groupId>io.reactivex</groupId>
    <artifactId>rxjava-math</artifactId>
    <version>1.0.0</version>
</dependency>
```

或者，对于 Gradle 项目:

```java
compile 'io.reactivex:rxjava-math:1.0.0'
```

## 3.数学运算符

**`MathObservable` 专用于执行数学运算**，它的操作者使用另一个`Observable` 来发出可以被评估为数字的项目。

### 3.1.`Average`

`average`操作符发出一个值——由源发出的所有值的平均值。

让我们看看实际情况:

```java
Observable<Integer> sourceObservable = Observable.range(1, 20);
TestSubscriber<Integer> subscriber = TestSubscriber.create();

MathObservable.averageInteger(sourceObservable).subscribe(subscriber);

subscriber.assertValue(10);
```

处理原始值有四个类似的运算符:`averageInteger`、`average`、`Long`、`average`、`average`、`Double`。

### 3.2.`Max`

`max` 操作符发出遇到的最大数字。

让我们看看实际情况:

```java
Observable<Integer> sourceObservable = Observable.range(1, 20);
TestSubscriber<Integer> subscriber = TestSubscriber.create();

MathObservable.max(sourceObservable).subscribe(subscriber);

subscriber.assertValue(9);
```

需要注意的是，`max`操作符有一个重载的方法，该方法采用一个比较函数。

考虑到数学运算符也可以处理可以作为数字管理的对象，`max`重载运算符允许比较定制类型或定制标准类型排序。

让我们定义一下`Item` 类:

```java
class Item {
    private Integer id;

    // standard constructors, getter, and setter
}
```

我们现在可以定义`itemObservable`，然后使用`max`操作符来发出具有最高`id`的`Item`:

```java
Item five = new Item(5);
List<Item> list = Arrays.asList(
  new Item(1), 
  new Item(2), 
  new Item(3), 
  new Item(4), 
  five);
Observable<Item> itemObservable = Observable.from(list);

TestSubscriber<Item> subscriber = TestSubscriber.create();

MathObservable.from(itemObservable)
  .max(Comparator.comparing(Item::getId))
  .subscribe(subscriber);

subscriber.assertValue(five);
```

### 3.3.`Min`

`min` 操作符发出包含源中最小元素的单个项目:

```java
Observable<Integer> sourceObservable = Observable.range(1, 20);
TestSubscriber<Integer> subscriber = TestSubscriber.create();

MathObservable.min(sourceObservable).subscribe(subscriber);

subscriber.assertValue(1);
```

`min`操作符有一个接受比较器实例的重载方法:

```java
Item one = new Item(1);
List<Item> list = Arrays.asList(
  one, 
  new Item(2), 
  new Item(3), 
  new Item(4), 
  new Item(5));
TestSubscriber<Item> subscriber = TestSubscriber.create();
Observable<Item> itemObservable = Observable.from(list);

MathObservable.from(itemObservable)
  .min(Comparator.comparing(Item::getId))
  .subscribe(subscriber);

subscriber.assertValue(one);
```

### 3.4.`Sum`

`sum`操作符发出一个值，代表由源`Observable:`发出的所有数字的总和

```java
Observable<Integer> sourceObservable = Observable.range(1, 20);
TestSubscriber<Integer> subscriber = TestSubscriber.create();

MathObservable.sumInteger(sourceObservable).subscribe(subscriber);

subscriber.assertValue(210);
```

还有原语专用的类似运算符:`sumInteger`、`sum`、`Long`、`sum`、`sum`、`Double`。

## 4.聚合运算符

### 4.1.`Concat`

`concat`运算符将源发出的项目连接在一起`.`

现在让我们定义两个`Observables`并将它们连接起来:

```java
List<Integer> listOne = Arrays.asList(1, 2, 3, 4);
Observable<Integer> observableOne = Observable.from(listOne);

List<Integer> listTwo = Arrays.asList(5, 6, 7, 8);
Observable<Integer> observableTwo = Observable.from(listTwo);

TestSubscriber<Integer> subscriber = TestSubscriber.create();

Observable<Integer> concatObservable = observableOne
  .concatWith(observableTwo);

concatObservable.subscribe(subscriber);

subscriber.assertValues(1, 2, 3, 4, 5, 6, 7, 8);
```

更详细地说，`concat`操作符等待订阅传递给它的每个额外的`Observable`,直到前一个完成。

由于这个原因，连接一个立即开始发射物品的“hot”`Observable,`，将导致在所有先前的物品完成之前，丢失“hot”`Observable`发射的任何物品。

### 4.2.`Count`

`count`操作符发出由源发出的所有项目的计数:

让我们来数一数`Observable`发出的物品数量:

```java
List<String> lettersList = Arrays.asList(
  "A", "B", "C", "D", "E", "F", "G");
TestSubscriber<Integer> subscriber = TestSubscriber.create();

Observable<Integer> sourceObservable = Observable
  .from(lettersList).count();
sourceObservable.subscribe(subscriber);

subscriber.assertValue(7);
```

如果源`Observable`因错误而终止，`count`将传递一个通知错误，而不发出一个项目。然而，如果它根本没有终止，`count`既不会发射物品也不会终止。

对于`count`操作，还有一个`countLong`操作符，它最终发出一个`Long`值，用于那些可能超过`Integer`容量的序列。

### 4.3.`Reduce`

`reduce`操作符通过应用累加器函数将所有发出的元素减少为一个元素。

这个过程一直持续到所有的项都被发出，然后来自`reduce,`的`Observable,` 发出从函数返回的最终值。

现在，让我们看看如何简化一个列表`String`，以相反的顺序连接它们:

```java
List<String> list = Arrays.asList("A", "B", "C", "D", "E", "F", "G");
TestSubscriber<String> subscriber = TestSubscriber.create();

Observable<String> reduceObservable = Observable.from(list)
  .reduce((letter1, letter2) -> letter2 + letter1);
reduceObservable.subscribe(subscriber);

subscriber.assertValue("GFEDCBA");
```

### 4.4.`Collect`

`collect`操作符类似于`reduce`操作符，但是它专用于将元素收集到一个可变的数据结构中。

它需要两个参数:

*   返回空可变数据结构的函数
*   一个函数，当给定数据结构和发出的项时，它适当地修改数据结构

让我们看看如何从一个`Observable`返回一个`set`项:

```java
List<String> list = Arrays.asList("A", "B", "C", "B", "B", "A", "D");
TestSubscriber<HashSet> subscriber = TestSubscriber.create();

Observable<HashSet<String>> reduceListObservable = Observable
  .from(list)
  .collect(HashSet::new, HashSet::add);
reduceListObservable.subscribe(subscriber);

subscriber.assertValues(new HashSet(list));
```

### 4.5.`ToList`

The `toList` operator works just like the `collect` operation, but collects all elements into a single list – think about `Collectors.toList()` from the Stream API:

```java
Observable<Integer> sourceObservable = Observable.range(1, 5);
TestSubscriber<List> subscriber = TestSubscriber.create();

Observable<List<Integer>> listObservable = sourceObservable
  .toList();
listObservable.subscribe(subscriber);

subscriber.assertValue(Arrays.asList(1, 2, 3, 4, 5));
```

### 4.6.`ToSortedList`

就像前面的例子一样，但是发出的列表是排序的:

```java
Observable<Integer> sourceObservable = Observable.range(10, 5);
TestSubscriber<List> subscriber = TestSubscriber.create();

Observable<List<Integer>> listObservable = sourceObservable
  .toSortedList();
listObservable.subscribe(subscriber);

subscriber.assertValue(Arrays.asList(10, 11, 12, 13, 14));
```

正如我们所见，`toSortedList`使用默认的比较，但是也可以提供定制的比较函数。我们现在可以看到如何使用自定义排序函数对整数进行逆序排序:

```java
Observable<Integer> sourceObservable = Observable.range(10, 5);
TestSubscriber<List> subscriber = TestSubscriber.create();

Observable<List<Integer>> listObservable 
  = sourceObservable.toSortedList((int1, int2) -> int2 - int1);
listObservable.subscribe(subscriber);

subscriber.assertValue(Arrays.asList(14, 13, 12, 11, 10));
```

### 4.7.`ToMap`

`toMap`操作符将一个`Observable`发出的项目序列转换成一个由指定键函数键控的映射。

特别是，`toMap`操作符有不同的重载方法，这些方法需要以下一个、两个或三个参数:

1.  从项目中生成密钥的`keySelector`
2.  从发出的项目中产生实际值的`valueSelector`,该值将存储在映射中
3.  创建将保存项目的集合的`mapFactory`

让我们开始定义一个简单的类`Book`:

```java
class Book {
    private String title;
    private Integer year;

    // standard constructors, getters, and setters
}
```

我们现在可以看到如何将一系列发出的`Book` 项转换为`Map`，将书名作为键，将年份作为值`:`

```java
Observable<Book> bookObservable = Observable.just(
  new Book("The North Water", 2016), 
  new Book("Origin", 2017), 
  new Book("Sleeping Beauties", 2017)
);
TestSubscriber<Map> subscriber = TestSubscriber.create();

Observable<Map<String, Integer>> mapObservable = bookObservable
  .toMap(Book::getTitle, Book::getYear, HashMap::new);
mapObservable.subscribe(subscriber);

subscriber.assertValue(new HashMap() {{
  put("The North Water", 2016);
  put("Origin", 2017);
  put("Sleeping Beauties", 2017);
}});
```

### 4.8.`ToMultiMap`

在映射时，很多值共享同一个键是很常见的。将一个键映射到多个值的数据结构称为多重映射。

这可以通过`toMultiMap`操作符来实现，该操作符将一个`Observable`发出的项目序列转换成一个`List`，它也是一个由指定的键函数键控的映射。

这个操作符向`toMap` 操作符添加了另一个参数，这个参数允许指定值应该存储在哪个集合类型中。让我们来看看如何做到这一点:

```java
Observable<Book> bookObservable = Observable.just(
  new Book("The North Water", 2016), 
  new Book("Origin", 2017), 
  new Book("Sleeping Beauties", 2017)
);
TestSubscriber<Map> subscriber = TestSubscriber.create();

Observable multiMapObservable = bookObservable.toMultimap(
  Book::getYear, 
  Book::getTitle, 
  () -> new HashMap<>(), 
  (key) -> new ArrayList<>()
);
multiMapObservable.subscribe(subscriber);

subscriber.assertValue(new HashMap() {{
    put(2016, Arrays.asList("The North Water"));
    put(2017, Arrays.asList("Origin", "Sleeping Beauties"));
}});
```

## 5.结论

在本文中，我们探索了 RxJava 中可用的数学和聚合运算符——当然，还有如何使用它们的简单示例。

和往常一样，本文中的所有代码示例都可以在 Github 上找到[。](https://web.archive.org/web/20221205161936/https://github.com/eugenp/tutorials/tree/master/rxjava-modules/rxjava-operators)