# Java——从列表中随机获取项目/元素

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-random-list-element>

## 1。简介

选择一个随机的`List`元素是一个非常基本的操作，但是实现起来并不容易。在本文中，我们将展示在不同的上下文中实现这一点的最有效的方法。

## 2。挑选一个或多个随机物品

为了从一个`List`实例中获取一个随机项目，您需要生成一个随机索引号，然后使用`List.get()`方法根据这个生成的索引号获取一个项目。

这里的关键点是要记住，你不能使用超过你的`List's`大小的索引。

### 2.1。单一随机项目

为了选择一个随机指标，可以使用`Random.nextInt(int bound)` 方法:

```java
public void givenList_shouldReturnARandomElement() {
    List<Integer> givenList = Arrays.asList(1, 2, 3);
    Random rand = new Random();
    int randomElement = givenList.get(rand.nextInt(givenList.size()));
}
```

而不是`Random` 类，你可以一直使用静态方法`Math.random()` 并乘以列表大小(`Math.random()`生成 0(含)和 1(不含)之间的`Double`随机值，所以乘法后记得强制转换为`int`)。

### 2.2。多线程环境下选择随机索引

当使用单个`Random`类实例编写多线程应用程序时，可能会导致为访问该实例的每个进程选择相同的值。我们总是可以通过使用专用的`ThreadLocalRandom` 类为每个线程创建一个新的实例:

```java
int randomElementIndex
  = ThreadLocalRandom.current().nextInt(listSize) % givenList.size();
```

### 2.3。选择重复的随机项目

有时你可能想从列表中挑选一些元素。这很简单:

```java
public void givenList_whenNumberElementsChosen_shouldReturnRandomElementsRepeat() {
    Random rand = new Random();
    List<String> givenList = Arrays.asList("one", "two", "three", "four");

    int numberOfElements = 2;

    for (int i = 0; i < numberOfElements; i++) {
        int randomIndex = rand.nextInt(givenList.size());
        String randomElement = givenList.get(randomIndex);
    }
}
```

### 2.4。选择无重复的随机项目

这里，您需要确保元素在选择后从列表中删除:

```java
public void givenList_whenNumberElementsChosen_shouldReturnRandomElementsNoRepeat() {
    Random rand = new Random();
    List<String> givenList = Lists.newArrayList("one", "two", "three", "four");

    int numberOfElements = 2;

    for (int i = 0; i < numberOfElements; i++) {
        int randomIndex = rand.nextInt(givenList.size());
        String randomElement = givenList.get(randomIndex);
        givenList.remove(randomIndex);
    }
}
```

### 2.5。选择随机系列

如果您想获得随机的元素序列，utils 类可能会很方便:

```java
public void givenList_whenSeriesLengthChosen_shouldReturnRandomSeries() {
    List<Integer> givenList = Lists.newArrayList(1, 2, 3, 4, 5, 6);
    Collections.shuffle(givenList);

    int randomSeriesLength = 3;

    List<Integer> randomSeries = givenList.subList(0, randomSeriesLength);
}
```

## 3。结论

在本文中，我们探索了从不同场景`.`的`List`实例`e` 中获取随机元素的最有效方法

代码示例可以在 [GitHub](https://web.archive.org/web/20220815153345/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-list) 上找到。