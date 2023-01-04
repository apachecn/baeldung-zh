# 番石榴功能食谱

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/guava-functions-predicates>

## 1。概述

这本食谱被组织成**小而集中的食谱和代码片段**，用于使用番石榴函数式元素——谓词和函数。

**食谱格式**集中且实用——不需要多余的细节和解释。

## 2。烹饪书

按条件过滤集合(自定义谓词)

```java
List<Integer> numbers = Lists.newArrayList(1, 2, 3, 6, 10, 34, 57, 89);
Predicate<Integer> acceptEven = new Predicate<Integer>() {
    @Override
    public boolean apply(Integer number) {
        return (number % 2) == 0;
    }
};
List<Integer> evenNumbers = Lists.newArrayList(Collections2.filter(numbers, acceptEven));
Integer found = Collections.binarySearch(evenNumbers, 57);
assertThat(found, lessThan(0));
```

**从集合中过滤出空值**

```java
List<String> withNulls = Lists.newArrayList("a", "bc", null, "def");
Iterable<String> withoutNuls = Iterables.filter(withNulls, Predicates.notNull());
assertTrue(Iterables.all(withoutNuls, Predicates.notNull()));
```

**检查集合中所有元素的条件**

```java
List<Integer> evenNumbers = Lists.newArrayList(2, 6, 8, 10, 34, 90);
Predicate<Integer> acceptEven = new Predicate<Integer>() {
    @Override
    public boolean apply(Integer number) {
        return (number % 2) == 0;
    }
};
assertTrue(Iterables.all(evenNumbers, acceptEven));
```

**否定一个谓语**

```java
List<Integer> evenNumbers = Lists.newArrayList(2, 6, 8, 10, 34, 90);
Predicate<Integer> acceptOdd = new Predicate<Integer>() {
    @Override
    public boolean apply(Integer number) {
        return (number % 2) != 0;
    }
};
assertTrue(Iterables.all(evenNumbers, Predicates.not(acceptOdd)));
```

**应用简单功能**

```java
List<Integer> numbers = Lists.newArrayList(1, 2, 3);
List<String> asStrings = Lists.transform(numbers, Functions.toStringFunction());
assertThat(asStrings, contains("1", "2", "3"));
```

**通过首先应用中间函数对集合进行排序**

```java
List<Integer> numbers = Arrays.asList(2, 1, 11, 100, 8, 14);
Ordering<Object> ordering = Ordering.natural().onResultOf(Functions.toStringFunction());
List<Integer> inAlphabeticalOrder = ordering.sortedCopy(numbers);
List<Integer> correctAlphabeticalOrder = Lists.newArrayList(1, 100, 11, 14, 2, 8);
assertThat(correctAlphabeticalOrder, equalTo(inAlphabeticalOrder));
```

**复杂示例——链接谓词和函数**

```java
List<Integer> numbers = Arrays.asList(2, 1, 11, 100, 8, 14);
Predicate<Integer> acceptEvenNumber = new Predicate<Integer>() {
    @Override
    public boolean apply(Integer number) {
        return (number % 2) == 0;
    }
};
Function<Integer, Integer> powerOfTwo = new Function<Integer, Integer>() {
    @Override
    public Integer apply(Integer input) {
        return (int) Math.pow(input, 2);
    }
};

FluentIterable<Integer> powerOfTwoOnlyForEvenNumbers = 
FluentIterable.from(numbers).filter(acceptEvenNumber).transform(powerOfTwo);
assertThat(powerOfTwoOnlyForEvenNumbers, contains(4, 10000, 64, 196));
```

**构成两个函数**

```java
List<Integer> numbers = Arrays.asList(2, 3);
Function<Integer, Integer> powerOfTwo = new Function<Integer, Integer>() {
    @Override
    public Integer apply(Integer input) {
        return (int) Math.pow(input, 2);
    }
};
List<Integer> result = Lists.transform(numbers, 
    Functions.compose(powerOfTwo, powerOfTwo));
assertThat(result, contains(16, 81));
```

**创建一个由集合和函数支持的地图**

```java
Function<Integer, Integer> powerOfTwo = new Function<Integer, Integer>() {
    @Override
    public Integer apply(Integer input) {
        return (int) Math.pow(input, 2);
    }
};
Set<Integer> lowNumbers = Sets.newHashSet(2, 3, 4);

Map<Integer, Integer> numberToPowerOfTwoMuttable = Maps.asMap(lowNumbers, powerOfTwo);
Map<Integer, Integer> numberToPowerOfTwoImuttable = Maps.toMap(lowNumbers, powerOfTwo);
assertThat(numberToPowerOfTwoMuttable.get(2), equalTo(4));
assertThat(numberToPowerOfTwoImuttable.get(2), equalTo(4));
```

**用谓词创建一个函数**

```java
List<Integer> numbers = Lists.newArrayList(1, 2, 3, 6);
Predicate<Integer> acceptEvenNumber = new Predicate<Integer>() {
    @Override
    public boolean apply(Integer number) {
        return (number % 2) == 0;
    }
};
Function<Integer, Boolean> isEventNumberFunction = Functions.forPredicate(acceptEvenNumber);
List<Boolean> areNumbersEven = Lists.transform(numbers, isEventNumberFunction);

assertThat(areNumbersEven, contains(false, true, false, true));
```

## 3。更多番石榴食谱

番石榴是一个全面且非常有用的库——以下是烹饪书形式的更多 API:

*   #### [Guava ordering recipe](/web/20220926181116/https://www.baeldung.com/guava-order "The Ordering Cookbook")

*   #### [Guava series recipes](/web/20220926181116/https://www.baeldung.com/guava-collections "Guava Collections Cookbook")

享受吧。

## 4。结论

这种格式与我通常的教程有些不同——主要是因为这是我已经保存并使用了很长时间的内部开发食谱。我的目标是让这些信息在网上随时可用——并且每当我遇到一个新的有用的例子时，就把它添加进去。

所有这些例子和代码片段的实现**可以在 GitHub** 上找到[——这是一个基于 Maven 的项目，所以它应该很容易导入和运行。](https://web.archive.org/web/20220926181116/https://github.com/eugenp/tutorials/tree/master/guava-modules/guava-core)