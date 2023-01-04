# 放心使用 Groovy

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rest-assured-groovy>

## 1.概观

在本教程中，我们将看看如何在 Groovy 中使用放心库。

由于放心使用 Groovy，我们实际上有机会使用原始 Groovy 语法来创建更强大的测试用例。这是框架真正发挥作用的地方。

关于使用放心的必要设置，请查看我们的[前一篇文章](/web/20220627093504/https://www.baeldung.com/rest-assured-tutorial)。

## 2。Groovy 的集合 API

让我们从快速浏览一些基本的 Groovy 概念开始——用几个简单的例子来装备我们所需要的东西。

### 2.1。 *findAll* 方法

在这个例子中，我们将只关注`methods`、`closures`和`it`隐式变量。让我们首先创建一个很棒的单词集合:

```java
def words = ['ant', 'buffalo', 'cat', 'dinosaur']
```

现在，让我们用长度超过四个字母的单词创建另一个集合:

```java
def wordsWithSizeGreaterThanFour = words.findAll { it.length() > 4 }
```

这里，`findAll()`是应用于集合的方法，其中`closure`应用于该方法。`method`定义了应用于集合的逻辑，而`closure`为方法提供了一个谓词来定制逻辑。

我们告诉 Groovy 遍历集合，找到所有长度大于 4 的单词，并将结果返回到一个新的集合中。

### 2.2。*它的*变量

隐式变量`it`保存循环中的当前单词。新的集合`wordsWithSizeGreaterThanFour`将包含单词`buffalo`和`dinosaur`。

```java
['buffalo', 'dinosaur']
```

除了`findAll()`，还有其他的 Groovy 方法。

### 2.3。*收集*迭代器

最后是`collect`，它对集合中的每一项调用闭包，并返回一个包含每一项结果的新集合。让我们根据`words`系列中每件商品的尺寸创建一个新系列:

```java
def sizes = words.collect{it.length()} 
```

结果是:

```java
[3,7,3,8]
```

顾名思义，我们使用`sum`将集合中的所有元素加起来。我们可以这样总结`sizes`系列中的商品:

```java
def charCount = sizes.sum()
```

结果将是 21，即`words`集合中所有条目的字符数。

### 2.4。*最大/最小*操作者

`max/min`操作符被直观地命名为寻找集合中的最大或最小数:

```java
def maximum = sizes.max()
```

结果应该很明显，8。

### 2.5。*寻找*迭代器

我们使用`find`只搜索一个匹配闭包谓词的集合值。

```java
def greaterThanSeven=sizes.find{it>7}
```

结果 8 是满足谓词的集合项的第一个匹配项。

## 3。用 Groovy 验证 JSON】

如果我们在`http://localhost:8080/odds`有一个服务，它会返回我们最喜欢的足球比赛的赔率列表，如下所示:

```java
{
    "odds": [{
        "price": 1.30,
        "status": 0,
        "ck": 12.2,
        "name": "1"
    },
    {
        "price": 5.25,
        "status": 1,
        "ck": 13.1,
        "name": "X"
    },
    {
        "price": 2.70,
        "status": 0,
        "ck": 12.2,
        "name": "0"
    },
    {
        "price": 1.20,
        "status": 2,
        "ck": 13.1,
        "name": "2"
    }]
}
```

如果我们想验证状态大于 1 的赔率有价格`1.20`和 5 `.25`，那么我们这样做:

```java
@Test
public void givenUrl_whenVerifiesOddPricesAccuratelyByStatus_thenCorrect() {
    get("/odds").then().body("odds.findAll { it.status > 0 }.price",
      hasItems(5.25f, 1.20f));
}
```

这里发生的事情是这样的:我们使用 Groovy 语法加载键`odds`下的 JSON 数组。因为它有不止一个条目，所以我们获得了一个 Groovy 集合。然后，我们对这个集合调用`findAll`方法。

闭包谓词告诉 Groovy 用状态大于零的 JSON 对象创建另一个集合。

我们以`price`结束我们的路径，它告诉 groovy 在我们之前的 JSON 对象列表中创建另一个只包含赔率的价格的列表。然后我们将`hasItems` Hamcrest 匹配器应用于这个列表。

## 4。用 Groovy 验证 XML

假设我们在`http://localhost:8080/teachers`有一个服务，它返回一个教师列表，按照他们的`id`、`department`和`subjects`授课，如下所示:

```java
<teachers>
    <teacher department="science" id=309>
        <subject>math</subject>
        <subject>physics</subject>
    </teacher>
    <teacher department="arts" id=310>
        <subject>political education</subject>
        <subject>english</subject>
    </teacher>
</teachers>
```

现在我们可以验证在响应中返回的科学老师既教数学又教物理:

```java
@Test
public void givenUrl_whenVerifiesScienceTeacherFromXml_thenCorrect() {
    get("/teachers").then().body(
      "teachers.teacher.find { [[email protected]](/web/20220627093504/https://www.baeldung.com/cdn-cgi/l/email-protection) == 'science' }.subject",
        hasItems("math", "physics"));
}
```

我们使用 XML 路径`teachers.teacher`通过 XML 属性`department`获得教师列表。然后我们在这个列表中调用`find`方法。

我们对`find`的闭包谓词确保我们最终只得到来自`science`部门的教师。我们的 XML 路径终止于`subject` 标签。

因为有不止一个主题，我们将得到一个列表，并用`hasItems` Hamcrest 匹配器进行验证。

## 5.结论

在本文中，我们看到了如何在 Groovy 语言中使用放心库。

要获得本文的完整源代码，请查看我们的 [GitHub 项目](https://web.archive.org/web/20220627093504/https://github.com/eugenp/tutorials/tree/master/testing-modules/rest-assured)。