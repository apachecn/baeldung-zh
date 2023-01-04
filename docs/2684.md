# 在数组列表中搜索字符串

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-search-string-arraylist>

## 1.概观

在本教程中，我们将研究在[`ArrayList`](/web/20221206003224/https://www.baeldung.com/java-arraylist)中搜索`String`的不同方法。我们的目的是检查特定的非空字符序列是否出现在`ArrayList` 的任何元素中，并返回一个包含所有匹配元素的列表。

## 2.基本循环

首先，让我们使用 Java 的`String`类的`contains`方法，使用一个基本循环来搜索给定搜索字符串中的字符序列:

```
public List<String> findUsingLoop(String search, List<String> list) {
    List<String> matches = new ArrayList<String>();

    for(String str: list) {
        if (str.contains(search)) {
            matches.add(str);
        }
    }

    return matches;
} 
```

## 3.流

**[Java 8 Streams API](/web/20221206003224/https://www.baeldung.com/java-8-streams)通过使用函数运算为我们提供了一个更加紧凑的解决方案。**

首先，我们将使用`filter()`方法在输入列表中搜索搜索字符串，然后，我们将使用`collect`方法创建并填充包含匹配元素的列表:

```
public List<String> findUsingStream(String search, List<String> list) {
    List<String> matchingElements = list.stream()
      .filter(str -> str.trim().contains(search))
      .collect(Collectors.toList());

    return matchingElements;
}
```

## 4.第三方库

如果我们不能使用 Java 8 Stream API，我们可以看看第三方库，如 Commons Collections 和 Google Guava。

要使用它们，我们只需要在 pom.xml 文件中添加[番石榴](https://web.archive.org/web/20221206003224/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.google.guava%22%20a%3A%22guava%22)、[公共集合](https://web.archive.org/web/20221206003224/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22commons-collections4%22%20g%3A%22org.apache.commons%22)，或者两者的依赖关系:

```
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>

<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-collections4</artifactId>
    <version>4.1</version>
</dependency>
```

### 4.1.公共收藏

Commons Collections 为我们提供了一个方法`IterableUtils.filteredIterable()`，它根据一个`Predicate`匹配给定的`Iterable`。

让我们调用`IterableUtils.filteredIterable()`，定义谓词来只选择那些包含搜索字符串的元素。然后，我们将使用`IteratorUtils.toList()`将`Iterable`转换为`List`:

```
public List<String> findUsingCommonsCollection(String search, List<String> list) {
    Iterable<String> result = IterableUtils.filteredIterable(list, new Predicate<String>() {
        public boolean evaluate(String listElement) {
            return listElement.contains(search);
        }
    });

    return IteratorUtils.toList(result.iterator());
} 
```

### 4.2.谷歌番石榴

Google Guava 用`Iterables.filter()` 方法提供了一个类似于 Apache 的`filteredIterable()`的解决方案。让我们用它来过滤列表，只返回与我们的搜索字符串匹配的元素:

```
public List<String> findUsingGuava(String search, List<String> list) {         
    Iterable<String> result = Iterables.filter(list, Predicates.containsPattern(search));

    return Lists.newArrayList(result.iterator());
}
```

## 5.结论

在本教程中，我们学习了在`ArrayList.` 中搜索`String`的不同方法，我们首先从一个简单的`for`循环开始，然后使用 Stream API 继续。最后，我们看到了一些使用两个第三方库的例子——Google Guava 和 Commons Collections `.`

GitHub 上的[提供了完整的示例。](https://web.archive.org/web/20221206003224/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-list-2)