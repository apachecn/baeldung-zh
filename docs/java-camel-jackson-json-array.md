# 使用 camel-jackson 解组 JSON 数组

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-camel-jackson-json-array>

## 1。概述

Apache Camel 是一个强大的开源集成框架，实现了许多已知的 T2 企业集成模式。

通常，当使用 Camel 处理消息路由时，我们会希望使用许多受支持的可插拔数据格式中的一种。鉴于 JSON 在大多数现代 API 和数据服务中很流行，它成为一个显而易见的选择。

在本教程中，**我们将看看使用`camel-jackson`组件将 [JSON 数组](/web/20220929002006/https://www.baeldung.com/jackson-collection-array)解组到 Java 对象列表中的几种方法。**

## 2。依赖性

首先，让我们将 [`camel-jackson`依赖项](https://web.archive.org/web/20220929002006/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3Aorg.apache.camel%20a%3Acamel-jackson)添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>org.apache.camel</groupId>
    <artifactId>camel-jackson</artifactId>
    <version>3.6.0</version>
</dependency>
```

然后，我们还将专门为我们的单元测试添加`camel-test`依赖项，这也可以从 [Maven Central](https://web.archive.org/web/20220929002006/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3Aorg.apache.camel%20a%3Acamel-test) 获得:

```java
<dependency>
    <groupId>org.apache.camel</groupId>
    <artifactId>camel-test</artifactId>
    <version>3.6.0</version>
</dependency>
```

## 3。果域类

在整个教程中，我们将使用几个 light [POJO](/web/20220929002006/https://www.baeldung.com/java-pojo-class) 对象来建模我们的水果域。

让我们继续定义一个带有 id 和名称的类来表示一种水果:

```java
public class Fruit {

    private String name;
    private int id;

    // standard getter and setters
}
```

接下来，我们将定义一个容器来保存一列`Fruit`对象:

```java
public class FruitList {

    private List<Fruit> fruits;

    public List<Fruit> getFruits() {
        return fruits;
    }

    public void setFruits(List<Fruit> fruits) {
        this.fruits = fruits;
    }
}
```

在接下来的几节中，我们将看到如何将表示水果列表的 JSON 字符串解组到这些域类中。**最终我们要寻找的是一个类型为`List<Fruit>`的变量，我们可以用**来处理它。

## 4。解组一个 JSON `FruitList`

在第一个例子中，我们将使用 JSON 格式表示一个简单的水果列表:

```java
{
    "fruits": [
        {
            "id": 100,
            "name": "Banana"
        },
        {
            "id": 101,
            "name": "Apple"
        }
    ]
}
```

最重要的是，**我们应该强调这个 JSON 表示一个对象，这个对象包含一个名为`fruits,`的属性，这个属性包含我们的数组**。

现在让我们设置 Apache Camel [路由](/web/20220929002006/https://www.baeldung.com/apache-camel-intro#domain-specific-language)来执行反序列化:

```java
@Override
protected RouteBuilder createRouteBuilder() throws Exception {
    return new RouteBuilder() {
        @Override
        public void configure() throws Exception {
            from("direct:jsonInput")
              .unmarshal(new JacksonDataFormat(FruitList.class))
              .to("mock:marshalledObject");
        }
    };
}
```

在本例中，我们使用名为`jsonInput`的`direct`端点。接下来，我们调用`unmarshal`方法，该方法使用指定的数据格式在 Camel exchange 上解组消息体。

**我们正在使用带有自定义解组类型** `**FruitList**.` 的`JacksonDataFormat`类，这实质上是一个简单的包装器，它包装了 [Jackon `ObjectMapper`](/web/20220929002006/https://www.baeldung.com/jackson-object-mapper-tutorial) ，并允许我们与 JSON 之间进行封送。

最后，我们将`unmarshal`方法的结果发送到一个名为`marshalledObject`的`mock`端点。正如我们将要看到的，这就是我们将如何测试我们的路线，看看它是否正常工作。

记住这一点，让我们继续编写我们的第一个单元测试:

```java
public class FruitListJacksonUnmarshalUnitTest extends CamelTestSupport {

    @Test
    public void givenJsonFruitList_whenUnmarshalled_thenSuccess() throws Exception {
        MockEndpoint mock = getMockEndpoint("mock:marshalledObject");
        mock.expectedMessageCount(1);
        mock.message(0).body().isInstanceOf(FruitList.class);

        String json = readJsonFromFile("/json/fruit-list.json");
        template.sendBody("direct:jsonInput", json);
        assertMockEndpointsSatisfied();

        FruitList fruitList = mock.getReceivedExchanges().get(0).getIn().getBody(FruitList.class);
        assertNotNull("Fruit lists should not be null", fruitList);

        List<Fruit> fruits = fruitList.getFruits();
        assertEquals("There should be two fruits", 2, fruits.size());

        Fruit fruit = fruits.get(0);
        assertEquals("Fruit name", "Banana", fruit.getName());
        assertEquals("Fruit id", 100, fruit.getId());

        fruit = fruits.get(1);
        assertEquals("Fruit name", "Apple", fruit.getName());
        assertEquals("Fruit id", 101, fruit.getId());
    }
}
```

让我们浏览一下测试的关键部分，以了解发生了什么:

*   首先，我们从扩展`CamelTestSupport`类开始——一个有用的测试工具基类
*   然后我们建立我们的测试期望。我们的`mock`变量应该有一个消息，消息类型应该是一个`FruitList`
*   **现在我们准备将 JSON 输入文件作为`String` 发送到我们之前定义的`direct`端点**
*   在我们检查我们的模拟期望已经被满足之后，我们可以自由地检索`FruitList`并检查内容是否如预期的那样

这个测试确认了我们的路由工作正常，我们的 JSON 按预期解组。厉害！

## 5。解组一个 JSON `Fruit`数组

另一方面，我们希望避免使用容器对象来保存我们的`Fruit`对象。我们可以修改我们的 JSON 来直接保存一个水果数组:

```java
[
    {
        "id": 100,
        "name": "Banana"
    },
    {
        "id": 101,
        "name": "Apple"
    }
]
```

这一次，我们的路线几乎是相同的，但是我们将它设置为专门使用 JSON 数组:

```java
@Override
protected RouteBuilder createRouteBuilder() throws Exception {
    return new RouteBuilder() {
        @Override
        public void configure() throws Exception {
            from("direct:jsonInput")
              .unmarshal(new ListJacksonDataFormat(Fruit.class))
              .to("mock:marshalledObject");
        }
    };
}
```

正如我们所看到的，与上一个例子的唯一区别是我们使用了带有自定义解组类型`Fruit`的`ListJacksonDataFormat`类。**这是一种 Jackson 数据格式类型，准备直接用于列表**。

同样，我们的单元测试也非常相似:

```java
@Test
public void givenJsonFruitArray_whenUnmarshalled_thenSuccess() throws Exception {
    MockEndpoint mock = getMockEndpoint("mock:marshalledObject");
    mock.expectedMessageCount(1);
    mock.message(0).body().isInstanceOf(List.class);

    String json = readJsonFromFile("/json/fruit-array.json");
    template.sendBody("direct:jsonInput", json);
    assertMockEndpointsSatisfied();

    @SuppressWarnings("unchecked")
    List<Fruit> fruitList = mock.getReceivedExchanges().get(0).getIn().getBody(List.class);
    assertNotNull("Fruit lists should not be null", fruitList);

    // more standard assertions
}
```

但是，相对于我们在上一节中看到的测试，有两个细微的区别:

*   我们首先设置我们的模拟期望直接包含一个带有`List.class`的主体
*   当我们以`List.class`的形式检索消息体时，我们将得到一个关于类型安全的标准警告——因此使用了`@SuppressWarnings(“unchecked”)`

## 6。结论

在这篇短文中，我们看到了两种使用 camel 消息路由和`camel-jackson`组件解组 JSON 数组的简单方法。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20220929002006/https://github.com/eugenp/tutorials/tree/master/spring-apache-camel)