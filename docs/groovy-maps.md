# Groovy 地图

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/groovy-maps>

## 1.概观

[Groovy](/web/20220926153008/https://www.baeldung.com/groovy-language) 将 [Java](/web/20220926153008/https://www.baeldung.com/category/java/) 中的 `[Map](/web/20220926153008/https://www.baeldung.com/java-collections)` API 扩展到**，为过滤、搜索、排序**等操作提供方法。它还提供了多种创建和操作地图的快捷方式。

在本教程中，我们将看看使用地图的绝妙方法。

## 2.创建 Groovy `Map` s

**我们可以使用 map literal 语法`[k:v]` 来创建地图。**基本上，它允许我们在一行中实例化一个地图并定义条目。

可以使用以下方法创建空地图:

```
def emptyMap = [:]
```

类似地，可以使用以下方法实例化带有值的映射:

```
def map = [name: "Jerry", age: 42, city: "New York"]
```

注意，**键没有用引号括起来，默认情况下，Groovy 会创建一个 [`java.util.LinkedHashMap`](/web/20220926153008/https://www.baeldung.com/java-linked-hashmap) 的实例。**我们可以通过使用`as`操作符来覆盖这个默认行为。

## 3.添加项目

让我们从定义地图开始:

```
def map = [name:"Jerry"]
```

我们可以在地图上添加一个键:

```
map["age"] = 42
```

然而，另一种更像 Javascript 的方法是使用属性符号(点运算符):

```
map.city = "New York"
```

换句话说，Groovy 支持以类似 bean 的方式访问键值对。

在向映射添加新项目时，我们也可以使用变量而不是文字作为键:

```
def hobbyLiteral = "hobby"
def hobbyMap = [(hobbyLiteral): "Singing"]
map.putAll(hobbyMap)
assertTrue(hobbyMap.hobby == "Singing")
assertTrue(hobbyMap[hobbyLiteral] == "Singing")
```

首先，我们必须创建一个存储键`hobby.`的新变量，然后我们使用这个包含在括号中的变量和 map literal 语法来创建另一个 map。

## 4.正在检索项目

文字语法或属性符号可用于从地图中获取项目。

对于定义为以下内容的地图:

```
def map = [name:"Jerry", age: 42, city: "New York", hobby:"Singing"]
```

我们可以得到键`name`对应的值:

```
assertTrue(map["name"] == "Jerry")
```

或者

```
assertTrue(map.name == "Jerry")
```

## 5.移除项目

我们可以使用`remove()`方法从基于键的地图中删除任何条目，但是**有时我们可能需要从地图中删除多个条目。我们可以通过使用`minus()`方法来做到这一点。**

`minus()`方法接受一个`Map`并在从底层地图中移除给定地图的所有条目后返回一个新的`Map`:

```
def map = [1:20, a:30, 2:42, 4:34, ba:67, 6:39, 7:49]

def minusMap = map.minus([2:42, 4:34]);
assertTrue(minusMap == [1:20, a:30, ba:67, 6:39, 7:49])
```

接下来，我们还可以根据条件删除条目。我们可以使用`removeAll()`方法来实现这一点:

```
minusMap.removeAll{it -> it.key instanceof String}
assertTrue(minusMap == [1:20, 6:39, 7:49])
```

相反，要保留满足条件的所有条目，我们可以使用`retainAll()`方法:

```
minusMap.retainAll{it -> it.value % 2 == 0}
assertTrue(minusMap == [1:20])
```

## 6.遍历条目

**我们可以使用`each()`** **和`eachWithIndex()`方法[遍历条目](/web/20220926153008/https://www.baeldung.com/groovy-map-iterating)。**

**`each()`方法提供隐式参数，如`entry`、`key`、`value,`对应当前`Entry`。**

除了`Entry`之外，`eachWithIndex()`方法还提供了一个索引。这两种方法都接受一个 [`Closure`](/web/20220926153008/https://www.baeldung.com/groovy-closures) 作为自变量。

在下一个例子中，我们将遍历每个`Entry.` 。传递给`each()`方法的`Closure`从隐式参数条目中获取键值对并打印出来:

```
map.each{entry -> println "$entry.key: $entry.value"}
```

接下来，我们将使用`eachWithIndex()`方法打印当前索引和其他值:

```
map.eachWithIndex{entry, i -> println "$i $entry.key: $entry.value"}
```

也可以要求分别提供`key`、`value,` 和索引:

```
map.eachWithIndex{key, value, i -> println "$i $key: $value"}
```

## 7.过滤

**我们可以使用`find(), findAll(),`和`grep()`方法根据键和值过滤和搜索地图条目。**

让我们首先定义一个映射来执行这些方法:

```
def map = [name:"Jerry", age: 42, city: "New York", hobby:"Singing"]
```

首先，我们来看一下`find()`方法，它接受一个`Closure`并返回匹配`Closure`条件的第一个`Entry`:

```
assertTrue(map.find{it.value == "New York"}.key == "city")
```

类似地，`findAll`也接受一个`Closure,`，但是返回一个`Map`，其中包含满足`Closure`中条件的所有键-值对:

```
assertTrue(map.findAll{it.value == "New York"} == [city : "New York"])
```

如果我们喜欢用`List` ，我们可以用`grep`代替`findAll`:

```
map.grep{it.value == "New York"}.each{it -> assertTrue(it.key == "city" && it.value == "New York")}
```

我们首先使用 grep 来查找值为 New York 的条目。然后，为了证明返回类型是`List,`，我们将为隐式参数中可用的列表中的每个`Entry`遍历`grep().`的结果，我们将检查它是否是预期的结果。

接下来，要找出地图中的所有项目是否满足某个条件，我们可以使用返回一个`boolean`的`every,`。

让我们检查一下地图中的所有值是否都是类型`String`:

```
assertTrue(map.every{it -> it.value instanceof String} == false)
```

类似地，我们可以使用`any`来确定地图中是否有任何项目匹配某个条件:

```
assertTrue(map.any{it -> it.value instanceof String} == true)
```

## 8.转化和收集

有时，我们可能希望将映射中的条目转换成新的值。**使用`collect()`和`collectEntries()`方法，可以分别将条目转换和收集到`Collection`或`Map,`中。**

让我们看一些例子。给定雇员 id 和雇员的映射:

```
def map = [
  1: [name:"Jerry", age: 42, city: "New York"],
  2: [name:"Long", age: 25, city: "New York"],
  3: [name:"Dustin", age: 29, city: "New York"],
  4: [name:"Dustin", age: 34, city: "New York"]]
```

我们可以使用`collect()`将所有员工的名字收集到一个列表中:

```
def names = map.collect{entry -> entry.value.name}
assertTrue(names == ["Jerry", "Long", "Dustin", "Dustin"])
```

然后，如果我们对一组独特的名称感兴趣，我们可以通过传递一个`Collection`对象来指定集合:

```
def uniqueNames = map.collect([] as HashSet){entry -> entry.value.name}
assertTrue(uniqueNames == ["Jerry", "Long", "Dustin"] as Set)
```

如果我们想将地图中的雇员姓名从小写改为大写，我们可以使用`collectEntries`。此方法返回转换值的映射:

```
def idNames = map.collectEntries{key, value -> [key, value.name]}
assertTrue(idNames == [1:"Jerry", 2:"Long", 3:"Dustin", 4:"Dustin"])
```

最后，**还可以将 `collect`方法与`find`和`findAll`方法**结合使用来转换过滤后的结果:

```
def below30Names = map.findAll{it.value.age < 30}.collect{key, value -> value.name}
assertTrue(below30Names == ["Long", "Dustin"])
```

在这里，我们将找到年龄在 20-30 岁之间的所有员工，并将他们收集到一个地图中。

## 9.分组

有时，我们可能希望根据条件将地图的某些项目分组到子地图中。

**`groupBy()`方法返回一个映射的映射，每个映射包含键值对，对于给定的条件，这些键值对的结果相同:**

```
def map = [1:20, 2: 40, 3: 11, 4: 93]

def subMap = map.groupBy{it.value % 2}
assertTrue(subMap == [0:[1:20, 2:40], 1:[3:11, 4:93]])
```

**创建子地图的另一种方法是使用`subMap()`** 。它与`groupBy()`的不同之处在于它只允许基于键的分组:

```
def keySubMap = map.subMap([1,2])
assertTrue(keySubMap == [1:20, 2:40])
```

在这种情况下，键 1 和键 2 的条目在新映射中返回，而所有其他条目都被丢弃。

## 10.整理

通常在排序时，我们可能希望根据键或值或两者来对映射中的条目进行排序。Groovy 提供了一个`sort()`方法可以用于这个目的。

给定一张地图:

```
def map = [ab:20, a: 40, cb: 11, ba: 93]
```

如果需要对键进行排序，我们将使用无参数`sort()`方法，它基于自然排序:

```
def naturallyOrderedMap = map.sort()
assertTrue([a:40, ab:20, ba:93, cb:11] == naturallyOrderedMap)
```

或者我们可以使用`sort(Comparator)`方法来提供比较逻辑:

```
def compSortedMap = map.sort({k1, k2 -> k1 <=> k2} as Comparator)
assertTrue([a:40, ab:20, ba:93, cb:11] == compSortedMap)
```

接下来，**为了对键、值或两者进行排序，我们可以为`sort()`** 提供一个`Closure`条件:

```
def cloSortedMap = map.sort({it1, it2 -> it1.value <=> it1.value})
assertTrue([cb:11, ab:20, a:40, ba:93] == cloSortedMap)
```

## 11.结论

在本文中，我们学习了如何在 Groovy 中创建`Map` s。然后，我们研究了在地图上添加、检索和删除项目的不同方法。

最后，我们讨论了 Groovy 用于执行常见操作的现成方法，比如过滤、搜索、转换和排序。

和往常一样，本文中的例子可以在 GitHub 上找到。