# 如何在 Java 中将列表转换成地图

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-list-to-map>

## 1。概述

将`List`转换为`Map`是一个常见的任务。在本教程中，我们将介绍几种方法来做到这一点。

我们假设`List`的每个元素都有一个标识符，该标识符将被用作结果`Map`中的一个键。

## 延伸阅读:

## [将列表转换为自定义供应商的地图](/web/20220816153817/https://www.baeldung.com/list-to-map-supplier)

Learn several ways to convert a List into a Map using Custom Suppliers.[Read more](/web/20220816153817/https://www.baeldung.com/list-to-map-supplier) →

## [在 Java 中将列表转换成字符串](/web/20220816153817/https://www.baeldung.com/java-list-to-string)

Learn how to convert a List to a String using different techniques.[Read more](/web/20220816153817/https://www.baeldung.com/java-list-to-string) →

## [在 Java 中的列表和集合之间转换](/web/20220816153817/https://www.baeldung.com/convert-list-to-set-and-set-to-list)

How to Convert between a List and a Set using plain Java, Guava or Apache Commons Collections.[Read more](/web/20220816153817/https://www.baeldung.com/convert-list-to-set-and-set-to-list) →

## 2。样本数据结构

首先，我们将对元素进行建模:

```java
public class Animal {
    private int id;
    private String name;

    //  constructor/getters/setters
}
```

`id`字段是唯一的，所以我们可以将它作为关键字。

让我们以传统的方式开始转换。

## 3。Java 8 之前

显然，我们可以使用核心 Java 方法将`List`转换成`Map `:

```java
public Map<Integer, Animal> convertListBeforeJava8(List<Animal> list) {
    Map<Integer, Animal> map = new HashMap<>();
    for (Animal animal : list) {
        map.put(animal.getId(), animal);
    }
    return map;
}
```

现在我们测试转换:

```java
@Test
public void whenConvertBeforeJava8_thenReturnMapWithTheSameElements() {
    Map<Integer, Animal> map = convertListService
      .convertListBeforeJava8(list);

    assertThat(
      map.values(), 
      containsInAnyOrder(list.toArray()));
}
```

## 4。用 Java 8

从 Java 8 开始，我们可以使用流和`Collectors`将`List`转换成`Map`:

```java
 public Map<Integer, Animal> convertListAfterJava8(List<Animal> list) {
    Map<Integer, Animal> map = list.stream()
      .collect(Collectors.toMap(Animal::getId, Function.identity()));
    return map;
}
```

同样，让我们确保转换正确完成:

```java
@Test
public void whenConvertAfterJava8_thenReturnMapWithTheSameElements() {
    Map<Integer, Animal> map = convertListService.convertListAfterJava8(list);

    assertThat(
      map.values(), 
      containsInAnyOrder(list.toArray()));
}
```

## 5。使用番石榴库

除了核心 Java，我们可以使用第三方库进行转换。

### 5.1。Maven 配置

首先，我们需要向我们的`pom.xml`添加以下依赖项:

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

这个库的最新版本总是可以在这里找到。

### 5.2。用`Maps.uniqueIndex()`转换

其次，让我们使用`Maps.uniqueIndex()`方法将一个`List`转换成一个`Map`:

```java
public Map<Integer, Animal> convertListWithGuava(List<Animal> list) {
    Map<Integer, Animal> map = Maps
      .uniqueIndex(list, Animal::getId);
    return map;
}
```

最后，我们测试转换:

```java
@Test
public void whenConvertWithGuava_thenReturnMapWithTheSameElements() {
    Map<Integer, Animal> map = convertListService
      .convertListWithGuava(list);

    assertThat(
      map.values(), 
      containsInAnyOrder(list.toArray()));
} 
```

## 6。使用 Apache 公共库

我们还可以用 Apache Commons 库方法进行转换。

### 6.1。Maven 配置

首先，让我们包括 Maven 依赖性:

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-collections4</artifactId>
    <version>4.4</version>
</dependency>
```

此依赖关系的最新版本可从[这里](https://web.archive.org/web/20220816153817/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.commons%22%20AND%20a%3A%22commons-collections4%22)获得。

### 6.2。`MapUtils`

其次，我们将使用`MapUtils.populateMap()`进行转换:

```java
public Map<Integer, Animal> convertListWithApacheCommons2(List<Animal> list) {
    Map<Integer, Animal> map = new HashMap<>();
    MapUtils.populateMap(map, list, Animal::getId);
    return map;
}
```

最后，我们可以确保它按预期工作:

```java
@Test
public void whenConvertWithApacheCommons2_thenReturnMapWithTheSameElements() {
    Map<Integer, Animal> map = convertListService
      .convertListWithApacheCommons2(list);

    assertThat(
      map.values(), 
      containsInAnyOrder(list.toArray()));
}
```

## 7。价值观冲突

让我们看看如果`id`字段不是唯一的会发生什么。

### 7.1。`List`的`Animals`与`Id`的重复

首先，我们用非唯一的`id`创建一个`Animal`的`List`:

```java
@Before
public void init() {

    this.duplicatedIdList = new ArrayList<>();

    Animal cat = new Animal(1, "Cat");
    duplicatedIdList.add(cat);
    Animal dog = new Animal(2, "Dog");
    duplicatedIdList.add(dog);
    Animal pig = new Animal(3, "Pig");
    duplicatedIdList.add(pig);
    Animal cow = new Animal(4, "Cow");
    duplicatedIdList.add(cow);
    Animal goat= new Animal(4, "Goat");
    duplicatedIdList.add(goat);
}
```

如上图所示，`cow`和`goat`有相同的`id`。

### 7.2。检查行为

**Java `Map`的`put()`方法被实现，使得最新添加的值用同一个键覆盖前一个值。**

因此，传统转换和 Apache Commons `MapUtils.populateMap()`的行为方式相同:

```java
@Test
public void whenConvertBeforeJava8_thenReturnMapWithRewrittenElement() {

    Map<Integer, Animal> map = convertListService
      .convertListBeforeJava8(duplicatedIdList);

    assertThat(map.values(), hasSize(4));
    assertThat(map.values(), hasItem(duplicatedIdList.get(4)));
}

@Test
public void whenConvertWithApacheCommons_thenReturnMapWithRewrittenElement() {

    Map<Integer, Animal> map = convertListService
      .convertListWithApacheCommons(duplicatedIdList);

    assertThat(map.values(), hasSize(4));
    assertThat(map.values(), hasItem(duplicatedIdList.get(4)));
}
```

我们可以看到，`goat`用相同的`id`覆盖了`cow`。

然而，**和`MapUtils.populateMap()`分别抛出`IllegalStateException`和`IllegalArgumentException`**:

```java
@Test(expected = IllegalStateException.class)
public void givenADupIdList_whenConvertAfterJava8_thenException() {

    convertListService.convertListAfterJava8(duplicatedIdList);
}

@Test(expected = IllegalArgumentException.class)
public void givenADupIdList_whenConvertWithGuava_thenException() {

    convertListService.convertListWithGuava(duplicatedIdList);
}
```

## 8。结论

在这篇简短的文章中，我们介绍了将`List`转换为`Map, `的各种方法，并给出了核心 Java 和一些流行的第三方库的例子。

和往常一样，完整的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220816153817/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-conversions)