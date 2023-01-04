# 番石榴订购指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/guava-ordering>

## 1.概观

在本文中，我们将看看来自番石榴库的`Ordering` 类。

`Ordering` 类实现了`Comparator`接口，并为我们提供了一个用于创建和链接比较器的有用的 fluent API。

顺便提一下，新的 [`Comparator.comparing()`](https://web.archive.org/web/20220526053753/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Comparator.html#comparing(java.util.function.Function)) API 也值得一看，它提供了类似的功能；[这里有一个使用该 API 的实际例子](/web/20220526053753/https://www.baeldung.com/java-8-sort-lambda)。

## 2.正在创建`Ordering`

`Ordering` 有一个有用的构建器方法，它返回一个合适的实例，可以用在集合上的`sort()` 方法中，或者任何需要`Comparator`实例的地方。

我们可以通过执行方法`natural():`来创建自然订单实例

```
List<Integer> integers = Arrays.asList(3, 2, 1);

integers.sort(Ordering.natural());

assertEquals(Arrays.asList(1,2,3), integers);
```

假设我们有一组`Person` 对象:

```
class Person {
    private String name;
    private Integer age;

    // standard constructors, getters
}
```

我们希望通过`age`字段对这些对象的列表进行排序。我们可以创建自己的自定义`Ordering` ，通过扩展它来实现这一点:

```
List<Person> persons = Arrays.asList(new Person("Michael", 10), new Person("Alice", 3));
Ordering<Person> orderingByAge = new Ordering<Person>() {
    @Override
    public int compare(Person p1, Person p2) {
        return Ints.compare(p1.age, p2.age);
    }
};

persons.sort(orderingByAge);

assertEquals(Arrays.asList(new Person("Alice", 3), new Person("Michael", 10)), persons);
```

然后我们可以使用我们的`orderingByAge` 并将其传递给`sort()` 方法。

## 3.链接`Orderings`

这个类的一个有用的特性是，我们可以链接不同的`Ordering.` 方式，比如说我们有一个人的集合，我们想按`age` 字段对其进行排序，并在列表的开头有`null` 年龄字段值:

```
List<Person> persons = Arrays.asList(
  new Person("Michael", 10),
  new Person("Alice", 3), 
  new Person("Thomas", null));

Ordering<Person> ordering = Ordering
  .natural()
  .nullsFirst()
  .onResultOf(new Function<Person, Comparable>() {
      @Override
      public Comparable apply(Person person) {
          return person.age;
      }
});

persons.sort(ordering);

assertEquals(Arrays.asList(
  new Person("Thomas", null), 
  new Person("Alice", 3), 
  new Person("Michael", 10)), persons);
```

这里需要注意的重要一点是特定`Ordering`的执行顺序——顺序是从右到左。所以首先执行`onResultOf()` ,该方法提取我们想要比较的字段。

然后，`nullFirst()`比较器被执行。因此，最终排序后的集合将有一个`Person` 对象，该对象在列表的开头有一个`null` 字段作为`age`字段。

在排序过程结束时，使用方法`natural()`指定的自然排序来比较`age`字段。

## 4.结论

在本文中，我们查看了来自 Guava 库的`Ordering` 类，它允许我们创建更流畅和优雅的 `Comparator`。我们创建了自己的自定义`Ordering,` ,我们使用了 API 中的预定义类，并将它们链接起来以实现更多的自定义顺序。

所有这些例子和代码片段的实现都可以在 GitHub 项目中找到——这是一个基于 Maven 的项目，所以应该很容易导入和运行。