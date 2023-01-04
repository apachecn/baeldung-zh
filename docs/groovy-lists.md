# Groovy 中的列表

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/groovy-lists>

## 1.概观

在 [Groovy](/web/20220926183714/https://www.baeldung.com/groovy-language) 中，我们可以像在 [Java](/web/20220926183714/https://www.baeldung.com/java-collections) 中一样处理列表。但是，由于它对扩展方法的支持，它附带了更多的功能。

在本教程中，我们将看看 Groovy 对列表的变异、过滤和排序。

## 2.创建 Groovy 列表

Groovy 在处理集合时提供了一些有趣的快捷方式，这利用了它对动态类型和文字语法的支持。

让我们首先使用简写语法创建一个包含一些值的列表:

```java
def list = [1,2,3]
```

同样，我们可以创建一个空列表:

```java
def emptyList = []
```

默认情况下，Groovy 会创建一个`java.util.ArrayList`的实例。

然而，**我们也可以指定要创建的列表类型**:

```java
def linkedList = [1,2,3] as LinkedList
ArrayList arrList = [1,2,3]
```

接下来，通过使用构造函数参数，列表可用于创建其他列表:

```java
def copyList = new ArrayList(arrList)
```

我们也可以通过克隆来做到这一点:

```java
def cloneList = arrList.clone()
```

请注意，克隆创建了列表的浅层副本。

Groovy 使用“==”操作符来比较两个列表中的元素是否相等。

继续前面的例子，比较`cloneList`和`arrlist`，结果是`true`:

```java
assertTrue(cloneList == arrList)
```

现在让我们看看如何对列表执行一些常见的操作。

## 3.从列表中检索项目

我们可以使用文字语法从列表中获取一个项目:

```java
def list = ["Hello", "World"]
assertTrue(list[1] == "World")
```

或者我们可以使用`get()`和`getAt()`方法:

```java
assertTrue(list.get(1) == "World")
assertTrue(list.getAt(1) == "World")
```

我们还可以使用正索引和负索引从列表中获取项目。

使用负索引时，列表从右向左读取:

```java
assertTrue(list[-1] == "World")
assertTrue(list.getAt(-2) == "Hello")
```

注意，`get()`方法不支持负索引。

## 4.向列表中添加项目

向列表中添加项目有多种快捷方式。

让我们定义一个空列表，并向其中添加一些项目:

```java
def list = []

list << 1
list.add("Apple")
assertTrue(list == [1, "Apple"])
```

接下来，我们还可以指定放置该项的索引。

另外，**如果列表的长度小于指定的索引，Groovy 会添加与差值**一样多的`null`值:

```java
list[2] = "Box"
list[4] = true
assertTrue(list == [1, "Apple", "Box", null, true])
```

最后，我们可以使用“+=”操作符向列表中添加新的条目。

与其他方法相比，**这个操作符创建了一个新的列表对象，并将其赋给变量`list`** :

```java
def list2 = [1,2]
list += list2
list += 12        
assertTrue(list == [1, 6.0, "Apple", "Box", null, true, 1, 2, 12])
```

## 5.更新列表中的项目

我们可以使用文字语法或`set()`方法更新列表中的条目:

```java
def list =[1, "Apple", 80, "App"]
list[1] = "Box"
list.set(2,90)
assertTrue(list == [1, "Box", 90,  "App"])
```

在此示例中，索引 1 和 2 处的项目用新值更新。

## 6.从列表中删除项目

我们可以使用`remove()`方法删除特定索引处的项目:

```java
def list = [1,2,3,4,5,5,6,6,7]
list.remove(3)
assertTrue(list == [1,2,3,5,5,6,6,7])
```

我们也可以使用`removeElement()`方法删除一个元素。

这将从列表中移除元素的第一个匹配项:

```java
list.removeElement(5)
assertTrue(list == [1,2,3,5,6,6,7])
```

另外，**我们可以使用`minus`操作符从列表中删除一个元素的所有出现。**

但是，该运算符不会改变基础列表，而是返回一个新列表:

```java
assertTrue(list - 6 == [1,2,3,5,7])
```

## 7.在列表上迭代

Groovy 在现有的 Java `Collections` API 中添加了新的方法。

这些方法简化了过滤、搜索、排序、聚合等操作。通过封装样板代码。它们还支持广泛的输入，包括闭包和输出数据结构。

让我们从遍历列表的两种方法开始。

`each()`方法接受闭包，与 Java 中的`foreach()`方法非常相似。

Groovy 在每次迭代中传递一个对应于当前元素的隐式参数`it`:

```java
def list = [1,"App",3,4]
list.each {println it * 2}
```

另一个方法`eachWithIndex()`除了提供当前元素之外，还提供当前索引值:

```java
list.eachWithIndex{ it, i -> println "$i : $it" }
```

## 8.过滤

过滤是另一种经常在列表上执行的操作，Groovy 提供了许多不同的方法供选择。

让我们定义一个要操作的列表:

```java
def filterList = [2,1,3,4,5,6,76]
```

为了找到匹配条件的第一个对象，我们可以使用`find`:

```java
assertTrue(filterList.find {it > 3} == 4)
```

为了找到所有符合条件的对象，我们可以使用`findAll`:

```java
assertTrue(filterList.findAll {it > 3} == [4,5,6,76])
```

让我们看另一个例子。

这里我们需要一个包含所有数字元素的列表:

```java
assertTrue(filterList.findAll {it instanceof Number} == [2,1,3,4,5,6,76])
```

或者，我们可以使用`grep`方法来做同样的事情:

```java
assertTrue(filterList.grep( Number ) == [2,1,3,4,5,6,76])
```

**`grep`和`find`方法的区别在于`grep`可以接受一个`Object`或一个`Closure`作为参数。**

因此，它允许进一步将条件语句减少到最低限度:

```java
assertTrue(filterList.grep {it > 6} == [76])
```

此外，`grep`使用`Object#isCase(java.lang.Object)`来评估列表中每个元素的条件。

有时，我们可能只对列表中独特的项目感兴趣。有两个重载方法可以用于这个目的。

`unique()`方法可选地接受一个闭包，并在底层列表中只保留匹配闭包条件的元素，而丢弃其他元素。

默认情况下，它使用自然排序来确定唯一性:

```java
def uniqueList = [1,3,3,4]
uniqueList.unique()
assertTrue(uniqueList == [1,3,4])
```

或者，如果要求不改变底层列表，我们可以使用`toUnique()`方法:

```java
assertTrue(["A", "B", "Ba", "Bat", "Cat"].toUnique {it.size()} == ["A", "Ba", "Bat"])
```

**如果我们想检查一个列表中的部分或全部项目是否满足某个条件，我们可以使用`every()`和`any()`方法。**

`every()`方法根据列表中的每个元素评估闭包中的条件。

那么只有当列表中的所有元素都满足条件时，它才返回`true`:

```java
def conditionList = [2,1,3,4,5,6,76]
assertFalse(conditionList.every {it < 6})
```

另一方面，如果列表中的任何元素满足条件，则`any()`方法返回`true`:

```java
assertTrue(conditionList.any {it % 2 == 0})
```

## 9.整理

默认情况下，Groovy 根据自然顺序对列表中的项目进行排序:

```java
assertTrue([1,2,1,0].sort() == [0,1,1,2])
```

但是**我们也可以通过自定义排序逻辑**传递一个`Comparator`:

```java
Comparator mc = {a,b -> a == b? 0: a < b? 1 : -1}
def list = [1,2,1,0]
list.sort(mc)
assertTrue(list == [2,1,1,0])
```

此外，我们可以使用`min()`或`max()`方法找到最大值或最小值，而不需要显式调用`sort()`:

```java
def strList = ["na", "ppp", "as"]
assertTrue(strList.max() == "ppp")
```

```java
Comparator minc = {a,b -> a == b? 0: a < b? -1 : 1}
def numberList = [3, 2, 0, 7]
assertTrue(numberList.min(minc) == 0)
```

## 10.收集

有时，我们可能希望修改列表中的项目，并返回另一个包含更新值的列表。

我们可以使用`collect()`方法来做到这一点:

```java
def list = ["Kay","Henry","Justin","Tom"]
assertTrue(list.collect{"Hi " + it} == ["Hi Kay","Hi Henry","Hi Justin","Hi Tom"])
```

## 11.连接

有时，我们可能需要将列表中的项目连接起来。

为此，我们可以使用`join()`方法:

```java
assertTrue(["One","Two","Three"].join(",") == "One,Two,Three")
```

## 12.结论

在本文中，我们介绍了 Groovy 添加到 Java `Collections` API 的一些扩展。

我们首先看字面语法，然后看它在创建、更新、删除和检索列表中的项目时的用法。

最后，我们看了 Groovy 对迭代、过滤、搜索、收集、连接和排序列表的支持。

和往常一样，文章中讨论的所有例子都可以在 GitHub 上找到[。](https://web.archive.org/web/20220926183714/https://github.com/eugenp/tutorials/tree/master/core-groovy-modules/core-groovy-collections)