# 在 Groovy 中查找集合中的元素

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/groovy-collections-find-elements>

## 1。简介

Groovy 提供了大量增强 Java 核心能力的方法。

在本教程中，我们将展示当**检查一个元素并在[几种类型的集合](/web/20220707143817/https://www.baeldung.com/groovy-lists)中找到它时，Groovy 是如何做到这一点的。**

## 2。测试元素是否存在

首先，我们将只关注测试给定的集合是否包含元素。

### 2.1。列表

Java 本身提供了几种用`java.util.List` 对列表中的项目进行**检查的方法:**

*   `contains`法
*   `indexOf `法

由于 Groovy 是一种 Java 兼容的语言，我们可以安全地使用它们。

让我们来看一个例子:

```
@Test
void whenListContainsElement_thenCheckReturnsTrue() {
    def list = ['a', 'b', 'c']

    assertTrue(list.indexOf('a') > -1)
    assertTrue(list.contains('a'))
}
```

除此之外， **Groovy 引入了成员操作符:**

```
element in list
```

这是 Groovy 提供的许多语法糖操作符之一。在它的帮助下，我们可以简化我们的代码:

```
@Test
void whenListContainsElement_thenCheckWithMembershipOperatorReturnsTrue() {
    def list = ['a', 'b', 'c']

    assertTrue('a' in list)
}
```

### 2.2。设置

和前面的例子一样，我们可以使用`java.util.Set#contains` 方法和`in`操作符:

```
@Test
void whenSetContainsElement_thenCheckReturnsTrue() {
    def set = ['a', 'b', 'c'] as Set

    assertTrue(set.contains('a'))
    assertTrue('a' in set)
}
```

### 2.3。地图

在`Map`的情况下，我们可以直接检查键或值:

```
@Test
void whenMapContainsKeyElement_thenCheckReturnsTrue() {
    def map = [a: 'd', b: 'e', c: 'f']

    assertTrue(map.containsKey('a'))
    assertFalse(map.containsKey('e'))
    assertTrue(map.containsValue('e'))
}
```

或者使用成员运算符来查找匹配的键:

```
@Test
void whenMapContainsKeyElement_thenCheckByMembershipReturnsTrue() {
    def map = [a: 'd', b: 'e', c: 'f']

    assertTrue('a' in map)
    assertFalse('f' in map)
}
```

当和地图一起使用时，我们应该小心使用成员运算符，因为这个运算符和布尔值一起使用有点混乱。底层机制从映射中检索相应的值，并由**将其转换为布尔值:**，而不是测试键是否存在

```
@Test
void whenMapContainsFalseBooleanValues_thenCheckReturnsFalse() {
    def map = [a: true, b: false, c: null]

    assertTrue(map.containsKey('b'))
    assertTrue('a' in map)
    assertFalse('b' in map)
    assertFalse('c' in map)
}
```

正如我们在上面的例子中可能看到的，出于同样的原因，使用`null`值也**有点危险。Groovy 将`false`和`null`都转换为布尔值`false`。**

## 3。所有匹配和任何匹配

在大多数情况下，我们处理由更复杂的对象组成的集合。在这一节中，我们将展示如何检查给定集合是否包含至少一个匹配元素，或者是否所有元素都匹配给定谓词。

让我们从定义一个简单的类开始，我们将在整个例子中使用它:

```
class Person {
    private String firstname
    private String lastname
    private Integer age

    // constructor, getters and setters
}
```

### 3.1。列表/设置

这次，我们将使用一个简单的`Person`对象列表:

```
private final personList = [
  new Person("Regina", "Fitzpatrick", 25),
  new Person("Abagail", "Ballard", 26),
  new Person("Lucian", "Walter", 30),
]
```

正如我们之前提到的， **Groovy 是一种兼容 Java 的语言**，所以我们先用 Java 8 引入的 [`Stream` API](/web/20220707143817/https://www.baeldung.com/java-8-streams-introduction) 创建一个例子:

```
@Test
void givenListOfPerson_whenUsingStreamMatching_thenShouldEvaluateList() {
    assertTrue(personList.stream().anyMatch {it.age > 20})
    assertFalse(personList.stream().allMatch {it.age < 30})
}
```

我们还可以使用 Groovy 方法`DefaultGroovyMethods#any `和`DefaultGroovyMethods#every `直接在集合上执行检查:

```
@Test
void givenListOfPerson_whenUsingCollectionMatching_thenShouldEvaluateList() {
    assertTrue(personList.any {it.age > 20})
    assertFalse(personList.every {it.age < 30})
}
```

### 3.2。地图

让我们从定义一个由`Person#firstname`映射的`Person `对象的`Map`开始:

```
private final personMap = [
  Regina : new Person("Regina", "Fitzpatrick", 25),
  Abagail: new Person("Abagail", "Ballard", 26),
  Lucian : new Person("Lucian", "Walter", 30)
]
```

我们可以通过它的键、值或整个条目来评估它。同样，让我们首先使用`Stream` API:

```
@Test
void givenMapOfPerson_whenUsingStreamMatching_thenShouldEvaluateMap() {
    assertTrue(personMap.keySet().stream().anyMatch {it == "Regina"})
    assertFalse(personMap.keySet().stream().allMatch {it == "Albert"})
    assertFalse(personMap.values().stream().allMatch {it.age < 30})
    assertTrue(personMap.entrySet().stream().anyMatch
      {it.key == "Abagail" && it.value.lastname == "Ballard"})
}
```

**然后，Groovy 集合 API:**

```
@Test
void givenMapOfPerson_whenUsingCollectionMatching_thenShouldEvaluateMap() {
    assertTrue(personMap.keySet().any {it == "Regina"})
    assertFalse(personMap.keySet().every {it == "Albert"})
    assertFalse(personMap.values().every {it.age < 30})
    assertTrue(personMap.any {firstname, person -> firstname == "Abagail" && person.lastname == "Ballard"})
}
```

正如我们所见，Groovy 不仅在操作地图时充分取代了`Stream` API，还允许我们直接在`Map`对象上执行检查，而不是使用`java.util.Map#entrySet`方法。

## 4。在集合中查找一个或多个元素

### 4.1。列表/设置

我们还可以使用谓词提取元素。让我们从熟悉的`Stream` API 方法开始:

```
@Test
void givenListOfPerson_whenUsingStreamFind_thenShouldReturnMatchingElements() {
    assertTrue(personList.stream().filter {it.age > 20}.findAny().isPresent())
    assertFalse(personList.stream().filter {it.age > 30}.findAny().isPresent())
    assertTrue(personList.stream().filter {it.age > 20}.findAll().size() == 3)
    assertTrue(personList.stream().filter {it.age > 30}.findAll().isEmpty())
}
```

正如我们所看到的，上面的例子使用了`java.util.Optional`来查找单个元素，因为`Stream` API 强制使用了这种方法。

另一方面，Groovy 提供了更紧凑的语法:

```
@Test
void givenListOfPerson_whenUsingCollectionFind_thenShouldReturnMatchingElements() {
    assertNotNull(personList.find {it.age > 20})
    assertNull(personList.find {it.age > 30})
    assertTrue(personList.findAll {it.age > 20}.size() == 3)
    assertTrue(personList.findAll {it.age > 30}.isEmpty())
}
```

通过使用 Groovy 的 API，**我们可以跳过创建一个`Stream`并过滤它**。

### 4.2。 **地图**

在`Map, `的情况下，有几个选项可供选择。**我们可以在关键字、值或完整条目中找到元素。**由于前两个基本上是一个`List`或一个`Set`，在本节中我们将只展示一个查找条目的例子。

让我们重新使用之前的`personMap`:

```
@Test
void givenMapOfPerson_whenUsingStreamFind_thenShouldReturnElements() {
    assertTrue(
      personMap.entrySet().stream()
        .filter {it.key == "Abagail" && it.value.lastname == "Ballard"}
        .findAny().isPresent())
    assertTrue(
      personMap.entrySet().stream()
        .filter {it.value.age > 20}
        .findAll().size() == 3)
}
```

同样，简化的 Groovy 解决方案:

```
@Test
void givenMapOfPerson_whenUsingCollectionFind_thenShouldReturnElements() {
    assertNotNull(personMap.find {it.key == "Abagail" && it.value.lastname == "Ballard"})
    assertTrue(personMap.findAll {it.value.age > 20}.size() == 3)
}
```

在这种情况下，好处更加显著。我们跳过`java.util.Map#entrySet `方法，使用一个带有`Map`提供的函数的闭包。

## 5。结论

在本文中，我们展示了 **Groovy 如何简化元素检查以及在几种类型的集合中查找元素。**

和往常一样，本教程中使用的完整代码示例可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220707143817/https://github.com/eugenp/tutorials/tree/master/core-groovy-modules/core-groovy-collections)