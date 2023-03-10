# Groovy 中的类别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/groovy-categories>

## 1.概观

我们有时会想，如果我们没有能力修改源代码，是否可以在编译后的 Java 或 Groovy 类中添加一些额外的方便的方法。事实证明，一个 Groovy 类别让我们做到了这一点。

Groovy 是一种动态的、强大的 JVM 语言，它有许多[元编程](/web/20221127232117/https://www.baeldung.com/groovy-metaprogramming)特性。

在本教程中，我们将探索 Groovy 中类别的概念。

## 2.什么是类别？

类别是一种元编程特性，受 Objective-C 的启发，它允许我们向新的或现有的 Java 或 Groovy 类添加额外的功能。

与[扩展](/web/20221127232117/https://www.baeldung.com/groovy-metaprogramming#extensions)不同，类别提供的附加功能在默认情况下是不启用的。因此，启用类别的关键是`use`代码块。

**由类别实现的额外特性只能在`use`代码块中访问。**

## 3.Groovy 中的类别

让我们讨论 Groovy 开发工具包中已经存在的几个突出的类别。

### 3.1.`TimeCategory`

在`groovy.time`包中有 [`TimeCategory`](https://web.archive.org/web/20221127232117/http://docs.groovy-lang.org/latest/html/api/groovy/time/TimeCategory.html) 类，它增加了一些处理`Date`和`Time`对象的便捷方法。

这个类别增加了将`Integer`转换成秒、分、日、月等时间符号的能力。

另外，`TimeCategory`类提供了类似于`plus`和`minus`的方法，分别用于将`Duration`添加到`Date`对象中的**和从`Date` 对象**中减去`Duration`。

让我们检查一下由`TimeCategory`类提供的一些方便的特性。对于这些例子，我们将首先创建一个`Date`对象，然后使用`TimeCategory`执行一些操作:

```java
def jan_1_2019 = new Date("01/01/2019")
use (TimeCategory) {
    assert jan_1_2019 + 10.seconds == new Date("01/01/2019 00:00:10")
    assert jan_1_2019 + 20.minutes == new Date("01/01/2019 00:20:00")
    assert jan_1_2019 - 1.day == new Date("12/31/2018")
    assert jan_1_2019 - 2.months == new Date("11/01/2018")
}
```

下面详细讨论一下代码。

这里，`10.seconds`创建了值为 10 秒的 [`TimeDuration`](https://web.archive.org/web/20221127232117/http://docs.groovy-lang.org/latest/html/api/groovy/time/TimeDuration.html) 对象。并且，加号(+)操作符将`TimeDuration`对象添加到`Date`对象中。

同样，`1.day`创建值为 1 天的 [`Duration`](https://web.archive.org/web/20221127232117/http://docs.groovy-lang.org/latest/html/api/groovy/time/Duration.html) 对象。并且，减号(-)运算符从`Date`对象中减去`Duration`对象。

另外，`now`、`ago,`和`from` 等一些方法可以通过`TimeCategory`类获得，这允许**创建相对日期**。

例如，`5.days.from.now`将创建一个值比当前日期早 5 天的`Date`对象。同样， `2.hours.ago`设置当前时间前 2 小时的值。

让我们看看他们的行动。此外，我们将使用`SimpleDateFormat`来忽略时间的界限，同时比较两个相似的`Date`对象:

```java
SimpleDateFormat sdf = new SimpleDateFormat("dd/MM/yyyy")
use (TimeCategory) {
    assert sdf.format(5.days.from.now) == sdf.format(new Date() + 5.days)

    sdf = new SimpleDateFormat("dd/MM/yyyy hh:mm:ss")
    assert sdf.format(10.minutes.from.now) == sdf.format(new Date() + 10.minutes)
    assert sdf.format(2.hours.ago) == sdf.format(new Date() - 2.hours)
}
```

因此，使用`TimeCategory`类，我们可以使用我们已经知道的类编写简单且可读性更好的代码。

### 3.2.`DOMCategory`

`groovy.xml.dom`包中有 [`DOMCategory`](https://web.archive.org/web/20221127232117/http://docs.groovy-lang.org/latest/html/api/groovy/xml/dom/DOMCategory.html) 类。它提供了一些使用 Java 的 DOM 对象的简便方法。

更具体地说， **`DOMCategory`允许对 DOM 元素进行 GPath 操作，从而允许更容易地遍历和处理 XML**。

首先，让我们编写一个简单的 XML 文本，并使用`DOMBuilder`类解析它:

```java
def baeldungArticlesText = """
<articles>
    <article core-java="true">
        <title>An Intro to the Java Debug Interface (JDI)</title>
        <desc>A quick and practical overview of Java Debug Interface.</desc>
    </article>
    <article core-java="false">
        <title>A Quick Guide to Working with Web Services in Groovy</title>
        <desc>Learn how to work with Web Services in Groovy.</desc>
    </article>
</articles>
"""

def baeldungArticlesDom = DOMBuilder.newInstance().parseText(baeldungArticlesText)
def root = baeldungArticlesDom.documentElement
```

这里，`root`对象包含 DOM 的所有子节点。让我们使用`DOMCategory` 类遍历这些节点:

```java
use (DOMCategory) {
    assert root.article.size() == 2

    def articles = root.article
    assert articles[0].title.text() == "An Intro to the Java Debug Interface (JDI)"
    assert articles[1].desc.text() == "Learn how to work with Web Services in Groovy."
}
```

这里，`DOMCategory`类允许**使用`[GPath](https://web.archive.org/web/20221127232117/https://groovy-lang.org/processing-xml.html#_gpath)`提供的点操作**轻松访问节点和元素。此外，它还提供了类似于 **`size`和`text`的方法来访问任意节点或元素**的信息。

现在，让我们使用`DOMCategory`向`root` DOM 对象添加一个新节点:

```java
use (DOMCategory) {
    def articleNode3 = root.appendNode(new QName("article"), ["core-java": "false"])
    articleNode3.appendNode("title", "Metaprogramming in Groovy")
    articleNode3.appendNode("desc", "Explore the concept of metaprogramming in Groovy")

    assert root.article.size() == 3
    assert root.article[2].title.text() == "Metaprogramming in Groovy"
}
```

类似地，`DOMCategory` 类也包含一些类似于 **`appendNode` 和 `setValue`的方法来修改 DOM** 。

## 4.创建类别

既然我们已经看到了几个 Groovy 类别，那么让我们来探索如何创建一个自定义类别。

### 4.1.使用自身对象

category 类必须遵循特定的实践来实现额外的特性。

**首先，添加附加特征的方法应该是`static`。其次，该方法的第一个参数应该是这个新特性适用的对象。**

让我们将`capitalize`特性添加到`String` 类中。这将简单地将`String`的第一个字母改为大写。

首先，我们将用一个`static`方法`capitalize`和`String`类型作为第一个参数来编写`BaeldungCategory`类:

```java
class BaeldungCategory {
    public static String capitalize(String self) {
        String capitalizedStr = self;
        if (self.size() > 0) {
            capitalizedStr = self.substring(0, 1).toUpperCase() + self.substring(1);
        }
        return capitalizedStr
    }
}
```

接下来，让我们编写一个快速测试来启用`BaeldungCategory`并验证`String`对象上的`capitalize`特性:

```java
use (BaeldungCategory) {
    assert "norman".capitalize() == "Norman"
}
```

同样，让我们编写一个特性来计算一个数的另一个数的幂:

```java
public static double toThePower(Number self, Number exponent) {
    return Math.pow(self, exponent);
}
```

最后，让我们测试我们的自定义类别:

```java
use (BaeldungCategory) {
    assert 50.toThePower(2) == 2500
    assert 2.4.toThePower(4) == 33.1776
}
```

### 4.2.`@Category`注释

我们还可以使用`@groovy.lang.Category`注释来**声明一个类别为实例样式的类**。当使用注释时，我们必须提供我们的类别适用的类名。

在方法中使用关键字`this`可以访问对象的实例。因此，`self`对象不需要成为第一个参数。

让我们编写一个`NumberCategory`类，并用`@Category`注释将其声明为一个类别。此外，我们将在新类别中添加一些额外的功能，如`cube`和`divideWithRoundUp`:

```java
@Category(Number)
class NumberCategory {
    public Number cube() {
        return this*this*this
    }

    public int divideWithRoundUp(BigDecimal divisor, boolean isRoundUp) {
        def mathRound = isRoundUp ? BigDecimal.ROUND_UP : BigDecimal.ROUND_DOWN
        return (int)new BigDecimal(this).divide(divisor, 0, mathRound)
    }
}
```

这里，`divideWithRoundUp` 特性将一个数除以除数，并基于`isRoundUp`参数将结果向上/向下舍入到下一个或上一个整数。

让我们测试我们的新类别:

```java
use (NumberCategory) {
    assert 3.cube() == 27
    assert 25.divideWithRoundUp(6, true) == 5
    assert 120.23.divideWithRoundUp(6.1, true) == 20
    assert 150.9.divideWithRoundUp(12.1, false) == 12
}
```

## 5.结论

在本文中，我们探讨了 Groovy 中类别的概念——这是一个元编程特性，可以在 Java 和 Groovy 类上启用附加特性。

我们已经研究了 Groovy `.` 中已经存在的几个类别，如`TimeCategory`和`DOMCategory,`，同时，我们还探索了一些使用这些类别与`Date`和 Java 的 DOM 协同工作的简便方法。

最后，我们探索了几种创建我们自己的自定义类别的方法。

像往常一样，所有的代码实现都可以在 GitHub 上获得[。](https://web.archive.org/web/20221127232117/https://github.com/eugenp/tutorials/tree/master/core-groovy-modules/core-groovy-2)