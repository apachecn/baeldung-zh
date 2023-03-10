# Apache Commons 收藏订购地图

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-commons-ordered-map>

[This article is part of a series:](javascript:void(0);)[• Apache Commons Collections Bag](/web/20220926202241/https://www.baeldung.com/apache-commons-bag)
[• Apache Commons Collections SetUtils](/web/20220926202241/https://www.baeldung.com/apache-commons-setutils)
• Apache Commons Collections OrderedMap (current article)[• Apache Commons Collections BidiMap](/web/20220926202241/https://www.baeldung.com/commons-collections-bidi-map)
[• A Guide to Apache Commons Collections CollectionUtils](/web/20220926202241/https://www.baeldung.com/apache-commons-collection-utils)
[• Apache Commons Collections MapUtils](/web/20220926202241/https://www.baeldung.com/apache-commons-map-utils)
[• Guide to Apache Commons CircularFifoQueue](/web/20220926202241/https://www.baeldung.com/commons-circular-fifo-queue)

## 1。概述

Apache Commons Collections 库提供了有用的类来补充 Java 集合框架。

在本文中，我们将回顾接口 [`OrderedMap`](https://web.archive.org/web/20220926202241/https://commons.apache.org/proper/commons-collections/apidocs/org/apache/commons/collections4/OrderedMap.html) ，它扩展了`java.util.Map`。

## 2。Maven 依赖关系

我们需要做的第一件事是在我们的`pom.xml`中添加 Maven 依赖项:

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-collections4</artifactId>
    <version>4.1</version>
</dependency> 
```

你可以在 [Maven Central repository](https://web.archive.org/web/20220926202241/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.commons%22%20AND%20a%3A%22commons-collections4%22) 上找到这个库的最新版本。

## 3。`OrderedMap`属性

简单来说，实现了`OrderedMap`接口的地图:

*   保持其键集的顺序，尽管该键集没有排序
*   可以用方法在两个方向上迭代:`firstKey()` 和`nextKey()`，或者`lastKey()`和`previousKey()`
*   可以用一个`MapIterator`遍历(也由库提供)
*   提供查找、更改、移除或替换元素的方法

## 4。使用`OrderedMap`

让我们在一个测试班中设置一个跑步者及其年龄的`OrderedMap`。我们将使用一个`LinkedMap`——库中提供的`OrderedMap`实现之一。

首先，让我们设置跑步者和年龄的数组，我们将使用这些数组来加载地图并验证值的顺序:

```java
public class OrderMapUnitTest {
    private String[] names = {"Emily", "Mathew", "Rose", "John", "Anna"};
    private Integer[] ages = {37, 28, 40, 36, 21};
    private LinkedMap<String, Integer> runnersLinkedMap;

    //...
}
```

现在，让我们初始化我们的地图:

```java
@Before
public void createRunners() {
    this.runnersLinkedMap = new LinkedMap<>();

    for (int i = 0; i < RUNNERS_COUNT; i++) {
        runners.put(this.names[i], this.ages[i]);
    }
} 
```

### 4.1。正向迭代

让我们看看正向迭代器是如何使用的:

```java
@Test
public void givenALinkedMap_whenIteratedForwards_thenPreservesOrder() {
    String name = this.runnersLinkedMap.firstKey();
    int i = 0;
    while (name != null) {
        assertEquals(name, names[i]);
        name = this.runnersLinkedMap.nextKey(name);
        i++;
    }
} 
```

**注意，当我们到达最后一个键时，方法`nextKey()`将返回一个`null`值。**

### 4.2。向后迭代

现在让我们从头开始，从最后一个键开始:

```java
@Test
public void givenALinkedMap_whenIteratedBackwards_thenPreservesOrder() {
    String name = this.runnersLinkedMap.lastKey();
    int i = RUNNERS_COUNT - 1;
    while (name != null) {
        assertEquals(name, this.names[i]);
        name = this.runnersLinkedMap.previousKey(name);
        i--;
    }
} 
```

**一旦我们到达第一个键，`previousKey()`方法将返回 null。**

### 4.3。`MapIterator`例子

现在让**使用`mapIterator()`方法获得一个`MapIterator`** ，因为我们展示了它如何保持数组`names`和`ages`中定义的跑步者的顺序:

```java
@Test
public void givenALinkedMap_whenIteratedWithMapIterator_thenPreservesOrder() {
    OrderedMapIterator<String, Integer> runnersIterator 
      = this.runnersLinkedMap.mapIterator();

    int i = 0;
    while (runnersIterator.hasNext()) {
        runnersIterator.next();

        assertEquals(runnersIterator.getKey(), this.names[i]);
        assertEquals(runnersIterator.getValue(), this.ages[i]);
        i++;
    }
} 
```

### 4.4。移除元件

最后，让我们看看如何通过索引或对象移除**元素:**

```java
@Test
public void givenALinkedMap_whenElementRemoved_thenSizeDecrease() {
    LinkedMap<String, Integer> lmap 
      = (LinkedMap<String, Integer>) this.runnersLinkedMap;

    Integer johnAge = lmap.remove("John");

    assertEquals(johnAge, new Integer(36));
    assertEquals(lmap.size(), RUNNERS_COUNT - 1);

    Integer emilyAge = lmap.remove(0);

    assertEquals(emilyAge, new Integer(37));
    assertEquals(lmap.size(), RUNNERS_COUNT - 2);
} 
```

## 5。提供的实现

目前，在该库的 4.1 版本中，`OrderedMap`接口有两个实现——`ListOrderedMap`和`LinkedMap`。

[`ListOrderedMap`](https://web.archive.org/web/20220926202241/https://commons.apache.org/proper/commons-collections/apidocs/org/apache/commons/collections4/map/ListOrderedMap.html) 使用`java.util.List`跟踪按键设置的顺序。它是`OrderedMap`的装饰器，可以通过使用静态方法`ListOrderedMap.decorate(Map map)`从任何`Map`创建。

[`LinkedMap`](https://web.archive.org/web/20220926202241/https://commons.apache.org/proper/commons-collections/apidocs/org/apache/commons/collections4/map/LinkedMap.html) 基于`HashMap`并通过允许双向迭代和`OrderedMap`接口的其他方法对其进行改进。

**两个实现都提供了三个在`OrderedMap`接口**之外的方法:

*   **`asList()`**–获取一个类型为`List<K>`(其中`K`是键的类型)的列表，保留地图的顺序
*   **`get(int index)`**–获取位置`index`的元素，与接口中提供的方法`get(Object o)`相反
*   **`indexOf(Object o)`**–获取有序地图中对象`o`的索引

我们可以将`OrderedMap`转换成`LinkedMap`来使用`asList()`方法:

```java
@Test
public void givenALinkedMap_whenConvertedToList_thenMatchesKeySet() {
    LinkedMap<String, Integer> lmap 
      = (LinkedMap<String, Integer>) this.runnersLinkedMap;

    List<String> listKeys = new ArrayList<>();
    listKeys.addAll(this.runnersLinkedMap.keySet());
    List<String> linkedMap = lmap.asList();

    assertEquals(listKeys, linkedMap);
} 
```

然后我们可以检查`LinkedMap`实现中方法`indexOf(Object o)`和`get(int index)`的功能:

```java
@Test
public void givenALinkedMap_whenSearchByIndexIsUsed_thenMatchesConstantArray() {
    LinkedMap<String, Integer> lmap 
      = (LinkedMap<String, Integer>) this.runnersLinkedMap;

    for (int i = 0; i < RUNNERS_COUNT; i++) {
        String name = lmap.get(i);

        assertEquals(name, this.names[i]);
        assertEquals(lmap.indexOf(this.names[i]), i);
    }
} 
```

## 6。结论

在这个快速教程中，我们回顾了`OrderedMap`接口及其主要方法和实现。

有关更多信息，请参见 Apache Commons Collections 库的 JavaDoc。

和往常一样，本文的完整测试类包含使用`LinkedMap`和`ListOrderedMap`的类似测试用例，可以从 [GitHub 项目](https://web.archive.org/web/20220926202241/https://github.com/eugenp/tutorials/tree/master/libraries-apache-commons-collections)下载。

Next **»**[Apache Commons Collections BidiMap](/web/20220926202241/https://www.baeldung.com/commons-collections-bidi-map)**«** Previous[Apache Commons Collections SetUtils](/web/20220926202241/https://www.baeldung.com/apache-commons-setutils)