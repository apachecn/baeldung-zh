# 在 Javadoc 中链接到外部 URL

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/javadoc-linking-external-url>

## 1.介绍

在编写代码时，我们可能会参考互联网上的文章，如维基页面、指南或图书馆的官方文档。在 Javadoc 中添加这些参考文章的链接可能是个好主意。

在本教程中，我们将学习如何在 Javadoc 中引用外部 URL。

## 2.创建内嵌链接

Java 没有为外部链接提供任何特殊的工具，但是我们可以使用标准的 HTML。以下语法用于创建内嵌链接:

```java
/**
 * Some text <a href="URL#value">label</a> 
 */
```

这里，`URL#value`可以是一个相对或绝对的 URL。

让我们考虑一个例子:

```java
/** 
 * Refer to <a href="http://www.baeldung.com">Baeldung</a> 
 */
```

这将呈现为:

**参考[拜尔东](https://web.archive.org/web/20220930183519/https://www.baeldung.com/)**

## 3.创建带有标题的内嵌链接

**另一种方法是创建包含链接的标题。**`@see`标签用于实现这一点，如下所示:

```java
/**
 * @see <a href="URL#value">label</a>
 */
```

考虑下面的例子:

```java
/**
 * @see <a href="http://www.baeldung.com">Baeldung</a> 
 */
```

这将创建一个包含链接的“另见”标题:
**另见:**
[拜尔东](https://web.archive.org/web/20220930183519/https://www.baeldung.com/)

## 4.创建到另一个类的 Javadoc 的链接

标签专门用于链接到其他类和方法的 Javadoc。这是一个内嵌标签，它转换成指向给定类或方法引用的文档的 HTML 超链接:

`{@link <class or method reference>}`

假设我们有一个包含方法`demo`的类`DemoOne`:

```java
/** 
 * Javadoc
 */
class DemoOne {

  /**
   * Javadoc
  */
  void demo() {
    //some code
  }
}
```

现在，我们可以通过以下方式从另一个类链接到上述类和方法的 Javadoc:

```java
/** 
 * See also {@link org.demo.DemoOne}
 */
```

```java
/**
 * See also {@link org.demo.DemoOne#demo()}
 */
```

这个标签可以用在任何可以写评论的地方，而`@see`创建自己的部分。

总而言之，当我们在描述中使用类名或方法名时，`@link`是首选。另一方面，当在描述中没有提到相关的参考文献时，或者作为到相同参考文献的多个链接的替换，使用`@see`。

## 5.结论

在本文中，我们学习了在 Javadoc 中创建外部链接的方法。我们还看了一下`@see`和`@link`标签之间的区别。