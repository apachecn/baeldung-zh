# 百里香列出了实用程序对象

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/thymeleaf-lists-utility>

## 1.概观

[百里叶](https://web.archive.org/web/20220728105348/http://www.thymeleaf.org/)是一个处理和创建 HTML 的 Java 模板引擎。

在这个快速教程中，我们将研究百里香的`lists`工具对象来执行常见的基于列表的操作。

## 2.计算大小

首先，**`size`方法返回一个列表的长度。**我们可以通过`th:text` 属性包含它:

```java
size: <span th:text="${#lists.size(myList)}"/>
```

`myList` 是我们自己的对象。我们已经通过控制器通过了[:](/web/20220728105348/https://www.baeldung.com/thymeleaf-in-spring-mvc)

```java
@GetMapping("/size")
public String usingSize(Model model) {
    model.addAttribute("myList", getColors());
    return "lists/size";
}
```

## 3.检查列表是否为空

**如果给定列表没有元素，`isEmpty`方法**返回 true:

```java
<span th:text="${#lists.isEmpty(myList)}"/> 
```

通常，该实用方法与条件句–`th:if` 和`th:unless`一起使用:

```java
<span th:unless="${#lists.isEmpty(myList)}">List is not empty</span>
```

## 4.检查成员资格

**`contains`方法**检查一个元素是否是给定列表的成员:

```java
myList contains red: <span th:text="${#lists.contains(myList, 'red')}"/>
```

类似地，**我们可以使用`containsAll`方法检查多个元素**的成员资格:

```java
myList contains red and green: <span th:text='${#lists.containsAll(myList, {"red", "green"})}'/>
```

## 5.整理

**`sort`方法**使我们能够对列表进行排序:

```java
sort: <span th:text="${#lists.sort(myList)}"/>

sort with Comparator: <span th:text="${#lists.sort(myList, reverse)}"/>
```

这里我们有两个重载的`sort` 方法`.` ，首先，我们按照自然顺序对列表进行排序—`${#lists.sort(myList)}.` ,其次，我们传递一个类型为`Comparator`的附加参数。在我们的例子中，我们从模型中获取这个比较器。

## 6.转换为`List`

最后，**我们可以使用`toList`方法将`Iterable`和数组转换为`List`。**

```java
<span th:with="convertedList=${#lists.toList(myArray)}">
    converted list size: <span th:text="${#lists.size(convertedList)}"/>
</span>
```

这里我们创建一个新的`List`、`convertedList`，然后用# `lists.size.`打印它的大小

## 7.摘要

在本教程中，我们研究了百里香内置的`lists` 工具对象以及如何有效地使用它。

与往常一样，所有示例的源代码都可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220728105348/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-thymeleaf-2)