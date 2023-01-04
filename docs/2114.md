# 在 Groovy 中使用 XML

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/groovy-xml>

## 1。简介

Groovy 提供了大量专门用于遍历和操作 XML 内容的方法。

在本教程中，我们将演示如何使用各种方法在 Groovy 中从 XML 中添加、编辑或删除元素。我们还将展示如何**从头开始创建一个 XML 结构**。

## 2。定义模型

让我们在资源目录中定义一个 XML 结构，我们将在整个示例中使用它:

```
<articles>
    <article>
        <title>First steps in Java</title>
        <author id="1">
            <firstname>Siena</firstname>
            <lastname>Kerr</lastname>
        </author>
        <release-date>2018-12-01</release-date>
    </article>
    <article>
        <title>Dockerize your SpringBoot application</title>
        <author id="2">
            <firstname>Jonas</firstname>
            <lastname>Lugo</lastname>
        </author>
        <release-date>2018-12-01</release-date>
    </article>
    <article>
        <title>SpringBoot tutorial</title>
        <author id="3">
            <firstname>Daniele</firstname>
            <lastname>Ferguson</lastname>
        </author>
        <release-date>2018-06-12</release-date>
    </article>
    <article>
        <title>Java 12 insights</title>
        <author id="1">
            <firstname>Siena</firstname>
            <lastname>Kerr</lastname>
        </author>
        <release-date>2018-07-22</release-date>
    </article>
</articles>
```

并将其读入一个`InputStream`变量:

```
def xmlFile = getClass().getResourceAsStream("articles.xml")
```

## 3。`XmlParser`

让我们用`XmlParser`类开始探索这个流。

### 3.1。阅读

读取和解析 XML 文件可能是开发人员必须做的最常见的 XML 操作。`XmlParser`提供了一个非常简单明了的界面:

```
def articles = new XmlParser().parse(xmlFile)
```

此时，我们可以使用 GPath 表达式访问 XML 结构的属性和值。

现在让我们实现一个简单的测试，使用 [Spock](/web/20220628125858/https://www.baeldung.com/groovy-spock) 来检查我们的 *articles* 对象是否正确:

```
def "Should read XML file properly"() {
    given: "XML file"

    when: "Using XmlParser to read file"
    def articles = new XmlParser().parse(xmlFile)

    then: "Xml is loaded properly"
    articles.'*'.size() == 4
    articles.article[0].author.firstname.text() == "Siena"
    articles.article[2].'release-date'.text() == "2018-06-12"
    articles.article[3].title.text() == "Java 12 insights"
    articles.article.find { it.author.'@id'.text() == "3" }.author.firstname.text() == "Daniele"
}
```

为了理解如何访问 XML 值以及如何使用 GPath 表达式，让我们先关注一下`XmlParser#parse `操作结果的内部结构。

`articles`对象是`groovy.util.Node. `的一个实例，每个`Node`由名称、属性映射、值和父对象(可以是`null`或另一个`Node)`)组成。

在我们的例子中，`articles`的值是一个`groovy.util.NodeList `实例，它是一个`Node`集合的包装类。`NodeList`扩展了`java.util.ArrayList`类，它提供了通过索引提取元素的功能。为了获得一个字符串值`Node, `，我们使用`groovy.util.Node#text().`

在上面的例子中，我们引入了一些 GPath 表达式:

*   `articles.article[0].author.firstname` —获取第一篇文章作者的名字—`articles.article[n]`将直接访问第`n`篇文章
*   `‘*'`——获得`article`的孩子的列表——这相当于`groovy.util.Node#children()`
*   `author.'@id'` —获取`author` 元素的`id` 属性—`author.'@attributeName'` 通过其名称访问属性值(等同于:`author[‘@id']`和`[[email protected]](/web/20220628125858/https://www.baeldung.com/cdn-cgi/l/email-protection)`)

### 3.2。添加一个节点

与前面的例子类似，让我们先将 XML 内容读入一个变量。这将允许我们定义一个新节点，并使用`groovy.util.Node#append.`将其添加到我们的文章列表中

现在让我们实施一个测试来证明我们的观点:

```
def "Should add node to existing xml using NodeBuilder"() {
    given: "XML object"
    def articles = new XmlParser().parse(xmlFile)

    when: "Adding node to xml"
    def articleNode = new NodeBuilder().article(id: '5') {
        title('Traversing XML in the nutshell')
        author {
            firstname('Martin')
            lastname('Schmidt')
        }
        'release-date'('2019-05-18')
    }
    articles.append(articleNode)

    then: "Node is added to xml properly"
    articles.'*'.size() == 5
    articles.article[4].title.text() == "Traversing XML in the nutshell"
}
```

正如我们在上面的例子中看到的，这个过程非常简单。

我们还注意到，我们使用了`groovy.util.NodeBuilder,` ,这是对使用`Node`构造函数定义`Node`的一个很好的替代。

### 3.3。修改节点

我们还可以使用`XmlParser`来修改节点的值。为此，让我们再次解析 XML 文件的内容。接下来，我们可以通过更改`Node`对象的`value`字段来编辑内容节点。

让我们记住，当`XmlParser`使用 GPath 表达式时，我们总是检索`NodeList,` 的实例，所以为了修改第一个(也是唯一的)元素，我们必须使用它的索引来访问它。

让我们通过编写一个快速测试来检查我们的假设:

```
def "Should modify node"() {
    given: "XML object"
    def articles = new XmlParser().parse(xmlFile)

    when: "Changing value of one of the nodes"
    articles.article.each { it.'release-date'[0].value = "2019-05-18" }

    then: "XML is updated"
    articles.article.findAll { it.'release-date'.text() != "2019-05-18" }.isEmpty()
}
```

在上面的例子中，我们还使用了 [Groovy 集合 API](/web/20220628125858/https://www.baeldung.com/groovy-collections-find-elements) 来遍历`NodeList`。

### 3.4。更换节点

接下来，让我们看看如何替换整个节点，而不仅仅是修改它的一个值。

类似于添加新元素，我们将使用`NodeBuilder`作为`Node`定义，然后使用`groovy.util.Node#replaceNode`替换其中的一个现有节点:

```
def "Should replace node"() {
    given: "XML object"
    def articles = new XmlParser().parse(xmlFile)

    when: "Adding node to xml"
    def articleNode = new NodeBuilder().article(id: '5') {
        title('Traversing XML in the nutshell')
        author {
            firstname('Martin')
            lastname('Schmidt')
        }
        'release-date'('2019-05-18')
    }
    articles.article[0].replaceNode(articleNode)

    then: "Node is added to xml properly"
    articles.'*'.size() == 4
    articles.article[0].title.text() == "Traversing XML in the nutshell"
}
```

### 3.5。删除节点

使用`XmlParser`删除一个节点相当复杂。虽然`Node`类提供了`remove(Node child)`方法，但在大多数情况下，我们不会单独使用它。

相反，我们将展示如何删除其值满足给定条件的节点。

默认情况下，使用`Node.NodeList`引用链**访问嵌套元素会返回相应子节点的副本**。因此，我们不能在我们的`article`集合上直接使用`java.util.NodeList#removeAll`方法。

要通过谓词删除一个节点，我们必须首先找到匹配我们条件的所有节点，然后遍历它们，并在每次时调用父节点上的`java.util.Node#remove`方法。

让我们实现一个测试，删除作者 id 不是`3`的所有文章:

```
def "Should remove article from xml"() {
    given: "XML object"
    def articles = new XmlParser().parse(xmlFile)

    when: "Removing all articles but the ones with id==3"
    articles.article
      .findAll { it.author.'@id'.text() != "3" }
      .each { articles.remove(it) }

    then: "There is only one article left"
    articles.children().size() == 1
    articles.article[0].author.'@id'.text() == "3"
}
```

正如我们所看到的，作为删除操作的结果，我们收到了一个只有一篇文章的 XML 结构，它的 id 是`3`。

## 4。`XmlSlurper`

Groovy 还提供了另一个专门处理 XML 的类。在这一节中，我们将展示如何使用`XmlSlurper.`来读取和操作 XML 结构

### 4.1。阅读

和前面的例子一样，让我们从解析文件中的 XML 结构开始:

```
def "Should read XML file properly"() {
    given: "XML file"

    when: "Using XmlSlurper to read file"
    def articles = new XmlSlurper().parse(xmlFile)

    then: "Xml is loaded properly"
    articles.'*'.size() == 4
    articles.article[0].author.firstname == "Siena"
    articles.article[2].'release-date' == "2018-06-12"
    articles.article[3].title == "Java 12 insights"
    articles.article.find { it.author.'@id' == "3" }.author.firstname == "Daniele"
}
```

如我们所见，界面与`XmlParser`完全相同。**然而，输出结构使用了`groovy.util.slurpersupport.GPathResult`** ，它是`Node`的包装类。`GPathResult`提供了方法的简化定义，例如:`equals()`和`toString()`通过包装`Node#text().`结果，我们可以直接使用它们的名字来读取字段和参数。

### 4.2。添加一个节点

添加一个`Node`也非常类似于使用`XmlParser`。然而，在这种情况下，`groovy.util.slurpersupport.` `GPathResult#appendNode`提供了一个方法，该方法将`java.lang.Object `的一个实例作为参数。因此，我们可以按照`Node` `Builder`引入的相同约定来简化新的`Node`定义:

```
def "Should add node to existing xml"() {
    given: "XML object"
    def articles = new XmlSlurper().parse(xmlFile)

    when: "Adding node to xml"
    articles.appendNode {
        article(id: '5') {
            title('Traversing XML in the nutshell')
            author {
                firstname('Martin')
                lastname('Schmidt')
            }
            'release-date'('2019-05-18')
        }
    }

    articles = new XmlSlurper().parseText(XmlUtil.serialize(articles))

    then: "Node is added to xml properly"
    articles.'*'.size() == 5
    articles.article[4].title == "Traversing XML in the nutshell"
}
```

如果我们需要用`XmlSlurper,`修改 XML 的结构，我们必须重新初始化`articles`对象才能看到结果。**我们可以通过结合使用`groovy.util.XmlSlurper#parseText`和`groovy.xmlXmlUtil#serialize` 方法`.`和**来实现

### 4.3。修改节点

正如我们之前提到的，`GPathResult`引入了一种简化的数据操作方法。也就是说，与`XmlSlurper,`相反，我们可以使用节点名或参数名直接修改值:

```
def "Should modify node"() {
    given: "XML object"
    def articles = new XmlSlurper().parse(xmlFile)

    when: "Changing value of one of the nodes"
    articles.article.each { it.'release-date' = "2019-05-18" }

    then: "XML is updated"
    articles.article.findAll { it.'release-date' != "2019-05-18" }.isEmpty()
}
```

请注意，当我们只修改 XML 对象的值时，我们不必再次解析整个结构。

### 4.4。更换节点

现在让我们来替换整个节点。再次，`GPathResult `前来救援。我们可以很容易地使用`groovy.util.slurpersupport.NodeChild#replaceNode`替换节点，它扩展了`GPathResult `，并遵循使用`Object`值作为参数的相同约定:

```
def "Should replace node"() {
    given: "XML object"
    def articles = new XmlSlurper().parse(xmlFile)

    when: "Replacing node"
    articles.article[0].replaceNode {
        article(id: '5') {
            title('Traversing XML in the nutshell')
            author {
                firstname('Martin')
                lastname('Schmidt')
            }
            'release-date'('2019-05-18')
        }
    }

    articles = new XmlSlurper().parseText(XmlUtil.serialize(articles))

    then: "Node is replaced properly"
    articles.'*'.size() == 4
    articles.article[0].title == "Traversing XML in the nutshell"
}
```

正如添加节点时的情况一样，我们正在修改 XML 的结构，所以我们必须再次解析它。

### 4.5。删除节点

要使用`XmlSlurper,`删除一个节点，我们可以简单地通过提供一个空的`Node`定义来重用`groovy.util.slurpersupport.NodeChild#replaceNode` 方法:

```
def "Should remove article from xml"() {
    given: "XML object"
    def articles = new XmlSlurper().parse(xmlFile)

    when: "Removing all articles but the ones with id==3"
    articles.article
      .findAll { it.author.'@id' != "3" }
      .replaceNode {}

    articles = new XmlSlurper().parseText(XmlUtil.serialize(articles))

    then: "There is only one article left"
    articles.children().size() == 1
    articles.article[0].author.'@id' == "3"
}
```

同样，修改 XML 结构需要重新初始化我们的`articles`对象。

## 5。`XmlParser`vs`XmlSlurper`

正如我们在例子中展示的,`XmlParser `和`XmlSlurper`的用法非常相似。我们可以用这两种方法或多或少取得相同的结果。然而，它们之间的一些差异可能会使天平偏向一方或另一方。

首先， **`XmlParser`总是将整个文档解析成 DOM-ish 结构。正因为如此，我们可以同时读写它**。我们不能用`XmlSlurper`做同样的事情，因为它评估路径更慢。因此，`XmlParser`会消耗更多的内存。

另一方面，`XmlSlurper`使用了更直接的定义，使得工作起来更简单。我们还需要记住，**使用`XmlSlurper` 对 XML 进行的任何结构更改都需要重新初始化，如果一个接一个地进行许多更改，这会对性能产生不可接受的影响**。

应该谨慎决定使用哪种工具，这完全取决于用例。

## 6。`MarkupBuilder`

除了读取和操作 XML 树，Groovy 还提供了从头开始创建 XML 文档的工具。现在让我们使用`groovy.xml.MarkupBuilder`创建一个由第一个例子中的前两篇文章组成的文档:

```
def "Should create XML properly"() {
    given: "Node structures"

    when: "Using MarkupBuilderTest to create xml structure"
    def writer = new StringWriter()
    new MarkupBuilder(writer).articles {
        article {
            title('First steps in Java')
            author(id: '1') {
                firstname('Siena')
                lastname('Kerr')
            }
            'release-date'('2018-12-01')
        }
        article {
            title('Dockerize your SpringBoot application')
            author(id: '2') {
                firstname('Jonas')
                lastname('Lugo')
            }
            'release-date'('2018-12-01')
        }
    }

    then: "Xml is created properly"
    XmlUtil.serialize(writer.toString()) == XmlUtil.serialize(xmlFile.text)
}
```

在上面的例子中，我们可以看到`MarkupBuilder`对`Node`的定义使用了与我们之前对`NodeBuilder`和`GPathResult` 使用的完全相同的方法。

为了将来自`MarkupBuilder`的输出与预期的 XML 结构进行比较，我们使用了`groovy.xml.XmlUtil#serialize`方法。

## 7 .**。结论**

在本文中，我们探索了使用 Groovy 操作 XML 结构的多种方法。

我们看了使用 Groovy 提供的两个类解析、添加、编辑、替换和删除节点的例子:`XmlParser`和`XmlSlurper`。我们还讨论了它们之间的差异，并展示了如何使用`MarkupBuilder`从头开始构建 XML 树。

和往常一样，本文中使用的完整代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220628125858/https://github.com/eugenp/tutorials/tree/master/core-groovy-modules/core-groovy-2)