# 百里香叶中的条件 CSS 类

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-mvc-thymeleaf-conditional-css-classes>

## 1.概观

在这个快速教程中，我们将学习几种不同的方法来有条件地添加 CSS 类。

如果你对百里香不熟悉，我们建议你查看我们对它的介绍。

## 2.使用`th:if`

假设我们的目标是生成一个`<span>` ，它的类由服务器决定:

```java
<span class="base condition-true">
   I have two classes: "base" and either "condition-true" or "condition-false" depending on a server-side condition.
</span>
```

在提供这个 HTML 之前，我们需要一个好的方法让服务器评估一个条件，包括`condition-true`类或者`condition-false`类，以及一个`base`类。

当模板化 HTML 时，需要为动态行为添加一些条件逻辑是很常见的。

首先，让我们用`th:if`来演示条件逻辑的最简单形式:

```java
<span th:if="${condition}" class="base condition-true">
   This HTML is duplicated. We probably want a better solution.
</span>
<span th:if="${!condition}" class="base condition-false">
   This HTML is duplicated. We probably want a better solution.
</span>
```

我们在这里可以看到，这将逻辑上导致正确的 CSS 类被附加到我们的 HTML 元素，**但是这个解决方案违反了 [DRY 原则](/web/20220926192146/https://www.baeldung.com/java-clean-code)** ，因为它需要复制整个代码块。

使用`th:if`在某些情况下肯定是有用的，但是我们应该寻找另一种方法来动态地附加一个 CSS 类。

## 3.使用`th:attr`

百里香叶为我们提供了一个属性，可以让我们定义其他属性，称为`th:attr`。

让我们用它来解决我们的问题:

```java
<span th:attr="class=${condition ? 'base condition-true' : 'base condition-false'}">
   This HTML is consolidated, which is good, but the Thymeleaf attribute still has some redundancy in it.
</span>
```

您可能已经注意到,`base`类仍然是重复的。另外，**在定义类时，我们可以使用一个更具体的百里香属性。**

## 4.使用`th:class`

`th:class`属性是`th:attr=”class=…”`的快捷方式，所以让我们使用它，同时将`base`类从条件结果中分离出来:

```java
<span th:class="'base '+${condition ? 'condition-true' : 'condition-false'}">
   The base CSS class still has to be appended with String concatenation. We can do a little bit better.
</span>
```

这个解决方案相当不错，因为它满足我们的需求，而且是干的。然而，百里香还有另一个我们可以受益的属性。

## 5.使用`th:classappend`

将我们的基类与条件逻辑完全解耦不是很好吗？**我们可以静态定义我们的`base`类，并将条件逻辑减少到只有相关的部分:**

```java
<span class="base" th:classappend="${condition ? 'condition-true' : 'condition-false'}">
   This HTML is consolidated, and the conditional class is appended separately from the static base class.
</span>
```

## 6.结论

随着百里香代码的每一次迭代，我们了解了一种有用的条件技术，以后可能会派上用场。最终，**我们发现使用`th:classappend`为我们提供了干代码和关注点分离的最佳组合**，同时也满足了我们的目标。

所有这些例子都可以在 GitHub 上的一个有效的百里香项目[中看到。](https://web.archive.org/web/20220926192146/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-thymeleaf-3)