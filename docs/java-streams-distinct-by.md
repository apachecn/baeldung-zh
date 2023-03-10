# Java 流 API 中的 DistinctBy

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-streams-distinct-by>

## 1。概述

在列表中搜索不同的元素是我们作为程序员经常面临的任务之一。从 Java 8 开始，随着`Streams`的引入，我们有了一个新的 API 来使用函数方法处理数据。

在本文中，我们将展示使用列表中对象的特定属性过滤集合的不同替代方法。

## 2。使用流 API

Stream API 提供了基于`Object`类的`equals()`方法返回列表中不同元素的`distinct()`方法。

然而，如果我们想要通过特定的属性进行过滤，那么它就变得不那么灵活了。我们有一种选择，就是编写一个维护状态的过滤器。

### 2.1。使用状态过滤器

一个可能的解决方案是实现一个有状态的`Predicate:`

```java
public static <T> Predicate<T> distinctByKey(
    Function<? super T, ?> keyExtractor) {

    Map<Object, Boolean> seen = new ConcurrentHashMap<>(); 
    return t -> seen.putIfAbsent(keyExtractor.apply(t), Boolean.TRUE) == null; 
}
```

为了测试它，我们将使用下面的`Person`类，它具有属性`age`、`email`和`name:`

```java
public class Person { 
    private int age; 
    private String name; 
    private String email; 
    // standard getters and setters 
}
```

为了通过`name`获得新的过滤集合，我们可以使用:

```java
List<Person> personListFiltered = personList.stream() 
  .filter(distinctByKey(p -> p.getName())) 
  .collect(Collectors.toList());
```

## 3。使用 Eclipse 集合

[Eclipse Collections](https://web.archive.org/web/20221128094831/https://www.eclipse.org/collections/) 是一个库，它提供了在 Java 中处理`Streams`和集合的额外方法。

### 3.1。使用`ListIterate.distinct()`

`ListIterate.distinct()`方法允许我们使用各种`HashingStrategies.` 来过滤`Stream`，这些策略可以使用 lambda 表达式或方法引用来定义。

如果我们想按`Person's`名称过滤:

```java
List<Person> personListFiltered = ListIterate
  .distinct(personList, HashingStrategies.fromFunction(Person::getName));
```

或者，如果我们要使用的属性是 primitive (int，long，double)，我们可以使用这样一个专门的函数:

```java
List<Person> personListFiltered = ListIterate.distinct(
  personList, HashingStrategies.fromIntFunction(Person::getAge));
```

### 3.2。Maven 依赖关系

我们需要将以下依赖项添加到我们的`pom.xml`中，以便在我们的项目中使用 Eclipse 集合:

```java
<dependency> 
    <groupId>org.eclipse.collections</groupId> 
    <artifactId>eclipse-collections</artifactId> 
    <version>8.2.0</version> 
</dependency>
```

您可以在 [Maven Central](https://web.archive.org/web/20221128094831/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.eclipse.collections%22%20AND%20a%3A%22eclipse-collections%22) 存储库中找到最新版本的 Eclipse 集合库。

要了解更多关于这个库的信息，我们可以去[这篇文章](/web/20221128094831/https://www.baeldung.com/eclipse-collections)。

## 4。使用 Vavr(Javaslan**)**

这是 Java 8 的函数库，提供了不可变的数据和函数控制结构。

### 4.1。使用`List.distinctBy`

为了过滤列表，这个类提供了自己的 List 类，它有一个`distinctBy()`方法，允许我们根据它包含的对象的属性进行过滤:

```java
List<Person> personListFiltered = List.ofAll(personList)
  .distinctBy(Person::getName)
  .toJavaList();
```

### 4.2。Maven 依赖关系

我们将把下面的依赖项添加到我们的`pom.xml`中，以便在我们的项目中使用 Vavr。

```java
<dependency> 
    <groupId>io.vavr</groupId> 
    <artifactId>vavr</artifactId> 
    <version>0.9.0</version>  
</dependency>
```

你可以在 [Maven Central](https://web.archive.org/web/20221128094831/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22vavr%22) 资源库中找到最新版本的 Vavr 库。

要了解更多关于这个库的信息，我们可以去[这篇文章](/web/20221128094831/https://www.baeldung.com/vavr)。

## 5。使用 StreamEx

这个库为 Java 8 流处理提供了有用的类和方法。

### 5.1。使用`StreamEx.distinct`

在提供的类中有`StreamEx`，它有`distinct`方法，我们可以向它发送一个对我们想要区分的属性的引用:

```java
List<Person> personListFiltered = StreamEx.of(personList)
  .distinct(Person::getName)
  .toList();
```

### 5.2。Maven 依赖关系

我们将把下面的依赖项添加到我们的`pom.xml`中，以便在我们的项目中使用 StreamEx。

```java
<dependency> 
    <groupId>one.util</groupId> 
    <artifactId>streamex</artifactId> 
    <version>0.6.5</version> 
</dependency>
```

您可以在 [Maven Central](https://web.archive.org/web/20221128094831/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22one.util%22%20AND%20a%3A%22streamex%22) 存储库中找到 StreamEx 库的最新版本。

## 6。结论

在这个快速教程中，我们探索了如何使用标准 Java 8 API 和其他库的其他替代方法，基于一个属性获得流的不同元素的例子。

和往常一样，完整的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221128094831/https://github.com/eugenp/tutorials/tree/master/libraries-4)