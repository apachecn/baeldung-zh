# 哈姆克雷斯特收藏食谱

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hamcrest-collections-arrays>

## 1。简介

这本食谱说明了如何利用 Hamcrest 匹配器来处理和测试集合。

食谱的**格式是以示例为中心的**并且实用——不需要额外的细节和解释。

首先，让我们做一个快速的静态导入来涵盖我们接下来要使用的大多数实用 API:

```java
import static org.hamcrest.Matchers.*;
```

## 延伸阅读:

## [Hamcrest 普通核心匹配器](/web/20221208143859/https://www.baeldung.com/hamcrest-core-matchers)

Explore the different methods of CoreMatchers class in the Hamcrest library.[Read more](/web/20221208143859/https://www.baeldung.com/hamcrest-core-matchers) →

## [火腿豆火柴](/web/20221208143859/https://www.baeldung.com/hamcrest-bean-matchers)

Learn about Hamcrest bean matchers - a tool that provides an effective way of making assertions, a frequently used feature when writing unit tests.[Read more](/web/20221208143859/https://www.baeldung.com/hamcrest-bean-matchers) →

## [用 Hamcrest 测试](/web/20221208143859/https://www.baeldung.com/java-junit-hamcrest-guide)

In this very practical tutorial, we focus on using the Hamcrest API and on writing neater and more intuitive unit tests for our software.[Read more](/web/20221208143859/https://www.baeldung.com/java-junit-hamcrest-guide) →

## 2。烹饪书

**检查单个元素是否在集合中**

```java
List<String> collection = Lists.newArrayList("ab", "cd", "ef");
assertThat(collection, hasItem("cd"));
assertThat(collection, not(hasItem("zz")));
```

**检查集合中是否有多个元素**

```java
List<String> collection = Lists.newArrayList("ab", "cd", "ef");
assertThat(collection, hasItems("cd", "ef"));
```

**检查集合中的所有元素** 

**–有严格的顺序**

```java
List<String> collection = Lists.newArrayList("ab", "cd", "ef");
assertThat(collection, contains("ab", "cd", "ef"));
```

**–任意顺序**

```java
List<String> collection = Lists.newArrayList("ab", "cd", "ef");
assertThat(collection, containsInAnyOrder("cd", "ab", "ef"));
```

**检查集合是否为空**

```java
List<String> collection = Lists.newArrayList();
assertThat(collection, empty());
```

**检查数组是否为空**

```java
String[] array = new String[] { "ab" };
assertThat(array, not(emptyArray()));
```

**检查地图是否为空**

```java
Map<String, String> collection = Maps.newHashMap();
assertThat(collection, equalTo(Collections.EMPTY_MAP));
```

**检查 Iterable 是否为空**

```java
Iterable<String> collection = Lists.newArrayList();
assertThat(collection, emptyIterable());
```

**检查集合大小**

```java
List<String> collection = Lists.newArrayList("ab", "cd", "ef");
assertThat(collection, hasSize(3));
```

**检查可迭代的大小**

```java
Iterable<String> collection = Lists.newArrayList("ab", "cd", "ef");
assertThat(collection, Matchers.<String> iterableWithSize(3));
```

**检查每一项的情况**

```java
List<Integer> collection = Lists.newArrayList(15, 20, 25, 30);
assertThat(collection, everyItem(greaterThan(10)));
```

## 3。结论

这种形式是一种实验——我正在出版一些关于特定主题的内部开发食谱——[Google Guava](/web/20221208143859/https://www.baeldung.com/guava-collections "Guava Collections Cookbook")和现在的 Hamcrest。我的目标是让这些信息在网上随时可用——并且每当我遇到一个新的有用的例子时，就把它添加进去。

所有这些例子和代码片段的实现**可以在 GitHub** 上找到[——这是一个基于 Maven 的项目，所以它应该很容易导入和运行。](https://web.archive.org/web/20221208143859/https://github.com/eugenp/tutorials/tree/master/guava-modules/guava-collections)