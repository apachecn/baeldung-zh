# 在百里香叶中使用布尔运算

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/thymeleaf-boolean>

## 1.介绍

在这个快速教程中，我们将看看如何在百里香叶中使用布尔值。

在我们深入细节之前，百里香的基础知识可以在这篇文章中找到。

## 2.将表达式计算为布尔值

**在百里香叶中，任何值都可以计算为布尔值。**我们有几个值解释为`false` *:*

*   `null`
*   布尔值`false`
*   数量`0`
*   字符\ `0`
*   琴弦`“false”`、`“off”`和`“no”`

任何其他值被评估为`true`。

## 3.使用布尔作为渲染条件

为了有条件地呈现 HTML 元素，我们有两个选项:`th:if `和`th:unless `属性。

它们的效果正好相反——只有当属性值为`true `时，百里香叶才会呈现具有`th:if `属性的元素，而只有当属性值为`false:`时，百里香叶才会呈现具有`th`:除非属性的元素

```java
<span th:if="${true}">will be rendered</span>
<span th:unless="${true}">won't be rendered</span>
<span th:if="${false}">won't be rendered</span>
<span th:unless="${false}">will be rendered</span>
```

## 4.逻辑和条件运算符

此外，我们可以使用百里香叶中的三个经典逻辑运算符:

*   `and`
*   `or`
*   用关键字`not `或“！”进行否定标志

**我们可以在变量表达式中使用这些操作符，或者将多个变量表达式组合在一起:**

```java
<span th:if="${isRaining or isCold}">The weather is bad</span>
<span th:if="${isRaining} or ${isCold}">The weather is bad</span>
<span th:if="${isSunny and isWarm}">The weather is good</span>
<span th:if="${isSunny} and ${isWarm}">The weather is good</span>
<span th:if="${not isCold}">It's warm</span>
<span th:if="${!isCold}">It's warm</span>
<span th:if="not ${isCold}">It's warm</span>
<span th:if="!${isCold}">It's warm</span>
```

我们也可以使用条件操作符:`if-then`、`if-then-else`和默认操作符。

`if-then-else `运算符是通常的三进制，或`?`:运算符:

```java
It's <span th:text="${isCold} ? 'cold' : 'warm'"></span>
```

此外，`if-then `操作符是一个简化版本，我们没有 else 部分:

```java
<span th:text="${isRaining or isCold} ? 'The weather is bad'"></span>
```

默认操作符返回第一个操作数(如果不是`null `),否则返回第二个操作数:

```java
<span th:text="'foo' ?: 'bar'"></span> <!-- foo -->
<span th:text="null ?: 'bar'"></span> <!-- bar -->
<span th:text="0 ?: 'bar'"></span> <!-- 0 -->
<span th:text="1 ?: 'bar'"></span> <!-- 1 -->
```

默认操作符也称为 Elvis 操作符，因为它与 Elvis 的发型非常相似。

注意，Elvis 操作符只做一个`null `检查，它不会将第一个操作数作为布尔运算。

## 5.`#bools `效用对象

`#bools `是一个工具对象，默认情况下在表达式中可用，有一些方便的方法:

*   `#bools.isTrue(obj) `返回参数是否被评估为`true`
*   `#bools.isFalse(obj) `返回参数是否被评估为`false`
*   `#bools.xxxIsTrue(collection) `用`#bools.isTrue() `将参数的元素转换为布尔值，然后将它们收集到相同类型的集合中
*   `#bools.xxxIsFalse(collection) `用`#bools.isFalse() `将参数的元素转换为布尔值，然后将它们收集到相同类型的集合中
*   如果参数中的所有元素都被评估为`true`，则`#bools.xxxAnd(collection) `返回`true`
*   如果参数中的任何元素被评估为`true`，则`#bools.xxxOr(collection) `返回`true `

在上述方法中，`xxx `可以是`array`、`list `或`set`，这取决于方法的参数(以及`xxxIsTrue() `和`xxxIsFalse()`的返回值)。

## 6.结论

在本文中，我们看到了 Thymeleaf 如何将值解释为布尔值，以及我们如何有条件地呈现元素和使用布尔表达式。

像往常一样，Github 上的[提供了代码(有更多的例子)。](https://web.archive.org/web/20221208143909/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-thymeleaf-2)