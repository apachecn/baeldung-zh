# Groovy 中迭代地图的快速指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/groovy-map-iterating>

## 1.介绍

在这个简短的教程中，我们将看看在 Groovy 中使用标准语言特性(如`each`、`eachWithIndex, `和`for-in `循环)迭代地图的方法。

## 2.`each`法

假设我们有以下地图:

```java
def map = [
    'FF0000' : 'Red',
    '00FF00' : 'Lime',
    '0000FF' : 'Blue',
    'FFFF00' : 'Yellow'
]
```

我们可以通过给`each`方法提供一个简单的闭包来迭代地图:

```java
map.each { println "Hex Code: $it.key = Color Name: $it.value" }
```

我们还可以通过给入口变量命名来稍微提高可读性:

```java
map.each { entry -> println "Hex Code: $entry.key = Color Name: $entry.value" }
```

或者，如果我们想分别处理键和值，我们可以在闭包中分别列出它们:

```java
map.each { key, val ->
    println "Hex Code: $key = Color Name $val"
}
```

在 Groovy 中，用文字符号创建的地图是有序的。我们可以预期我们的输出与我们在原始地图中定义的顺序相同。

## 3.`eachWithIndex`法

有时候我们想在迭代的时候知道`index`。

例如，假设我们想在地图中每隔一行进行缩进。为了在 Groovy 中做到这一点，我们将使用带有`entry`和`index`变量的`eachWithIndex`方法:

```java
map.eachWithIndex { entry, index ->
    def indent = ((index == 0 || index % 2 == 0) ? "   " : "")
    println "$index Hex Code: $entry.key = Color Name: $entry.value"
}
```

与`each`方法一样，我们可以选择在闭包中使用`key`和`value`变量，而不是`entry`:

```java
map.eachWithIndex { key, val, index ->
    def indent = ((index == 0 || index % 2 == 0) ? "   " : "")
    println "$index Hex Code: $key = Color Name: $val"
}
```

## 4.使用`For-in`循环

另一方面，如果我们的用例更适合命令式编程，我们也可以使用`for-in`语句迭代我们的映射:

```java
for (entry in map) {
    println "Hex Code: $entry.key = Color Name: $entry.value"
}
```

## 5.结论

在这个简短的教程中，我们学习了如何使用 Groovy 的`each`和`eachWithIndex`方法以及一个`for-in`循环来迭代一个地图。

示例代码可以在 [GitHub](https://web.archive.org/web/20220625233313/https://github.com/eugenp/tutorials/tree/master/core-groovy-modules/core-groovy-collections) 上找到。