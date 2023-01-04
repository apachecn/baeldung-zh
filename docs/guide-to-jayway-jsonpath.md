# JsonPath 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/guide-to-jayway-jsonpath>

## 1。概述

XML 的优势之一是处理的可用性——包括 XPath——它被定义为 W3C 标准。对于 JSON 来说，出现了一个类似的工具，叫做 JSONPath。

本教程将对 Jayway JsonPath 做一个**介绍，这是 [JSONPath 规范](https://web.archive.org/web/20220901190031/http://goessner.net/articles/JsonPath/)的一个 Java 实现。它描述了设置、语法、公共 API 和用例演示。**

## 延伸阅读:

## [春季集成测试](/web/20220901190031/https://www.baeldung.com/integration-testing-in-spring)

A quick guide to writing integration tests for a Spring Web application.[Read more](/web/20220901190031/https://www.baeldung.com/integration-testing-in-spring) →

## [Spring MVC 中的 http mediatypenotacceptableexception](/web/20220901190031/https://www.baeldung.com/spring-httpmediatypenotacceptable)

Learn how to deal with the HttpMediaTypeNotAcceptableException in Spring.[Read more](/web/20220901190031/https://www.baeldung.com/spring-httpmediatypenotacceptable) →

## [Spring REST API 的二进制数据格式](/web/20220901190031/https://www.baeldung.com/spring-rest-api-with-binary-data-formats)

In this article we explore how to configure Spring REST mechanism to utilize binary data formats which we illustrate with Kryo. Moreover we show how to support multiple data formats with Google Protocol buffers.[Read more](/web/20220901190031/https://www.baeldung.com/spring-rest-api-with-binary-data-formats) →

## 2。设置

要使用 JsonPath，我们只需要在 Maven pom 中包含一个依赖项:

```java
<dependency>
    <groupId>com.jayway.jsonpath</groupId>
    <artifactId>json-path</artifactId>
    <version>2.4.0</version>
</dependency>
```

## 3。语法

我们将使用下面的 JSON 结构来演示 JsonPath 的语法和 API:

```java
{
    "tool": 
    {
        "jsonpath": 
        {
            "creator": 
            {
                "name": "Jayway Inc.",
                "location": 
                [
                    "Malmo",
                    "San Francisco",
                    "Helsingborg"
                ]
            }
        }
    },

    "book": 
    [
        {
            "title": "Beginning JSON",
            "price": 49.99
        },

        {
            "title": "JSON at Work",
            "price": 29.99
        }
    ]
}
```

### 3.1。符号

JsonPath 使用特殊的符号来表示节点以及它们与 JsonPath 中相邻节点的连接。符号有两种样式:点和括号。

以下两个路径都引用了上述 JSON 文档中的同一个节点，即*创建者*节点的*位置*字段中的第三个元素，也就是根节点下属于`tool`的 *jsonpath* 对象的子节点。

首先，我们将看到带有点符号的路径:

```java
$.tool.jsonpath.creator.location[2]
```

现在让我们看看括号符号:

```java
$['tool']['jsonpath']['creator']['location'][2]
```

美元符号($)代表根成员对象。

### 3.2。操作员

JsonPath 中有几个有用的操作符:

*   **根节点($)** 表示 JSON 结构的根成员，无论它是对象还是数组。我们在前一小节中包括了使用示例。
*   **当前节点(@)** 表示正在处理的节点。我们通常将它作为谓词输入表达式的一部分。假设我们正在处理上面 JSON 文档中的`book`数组；表达式`book[?(@.price == 49.99)]`指的是数组中的第一个`book`。
*   **通配符(*)** 表示指定范围内的所有元素。例如，`book[*]`表示一个`book`数组中的所有节点。

### 3.3。功能和过滤器

JsonPath 还有一些函数，我们可以在路径的末尾使用它们来合成路径的输出表达式:`min()`、`max()`、`avg()`、`stddev()`和`length()`。

最后，我们有过滤器。这些布尔表达式将返回的节点列表限制为调用方法需要的那些。

几个例子是等式(`==`)、正则表达式匹配(`=~`)、包含(`in`)和检查空值(`empty`)。我们主要对谓词使用过滤器。

有关不同操作符、函数和过滤器的完整列表和详细解释，请参考 [JsonPath GitHub](https://web.archive.org/web/20220901190031/https://github.com/jayway/JsonPath) 项目。

## 4。操作

在我们开始操作之前，快速补充一下:本节使用了我们之前定义的 JSON 示例结构。

### 4.1。获取文件

JsonPath 有一种方便的方法来访问 JSON 文档。我们通过静态`read`API 来实现这一点:

```java
<T> T JsonPath.read(String jsonString, String jsonPath, Predicate... filters);
```

`read`API 可以与静态流畅 API 一起工作，以提供更大的灵活性:

```java
<T> T JsonPath.parse(String jsonString).read(String jsonPath, Predicate... filters);
```

对于不同类型的 JSON 源，我们可以使用`read`的其他重载变体，包括`Object`、*、InputStream* 、`URL`和`File`。

为了简单起见，这一部分的测试不包括参数列表中的谓词(空`varargs`)。但是我们将在后面的小节中讨论`predicates`。

让我们首先定义两个要处理的示例路径:

```java
String jsonpathCreatorNamePath = "$['tool']['jsonpath']['creator']['name']";
String jsonpathCreatorLocationPath = "$['tool']['jsonpath']['creator']['location'][*]";
```

接下来，我们将通过解析给定的 JSON 源`jsonDataSourceString`来创建一个`DocumentContext`对象。然后，新创建的对象将用于通过上面定义的路径读取内容:

```java
DocumentContext jsonContext = JsonPath.parse(jsonDataSourceString);
String jsonpathCreatorName = jsonContext.read(jsonpathCreatorNamePath);
List<String> jsonpathCreatorLocation = jsonContext.read(jsonpathCreatorLocationPath);
```

第一个`read` API 返回一个包含 JsonPath 创建者名字的`String`，而第二个返回它的地址列表。

我们将使用 JUnit `Assert` API 来确认这些方法是否按预期工作:

```java
assertEquals("Jayway Inc.", jsonpathCreatorName);
assertThat(jsonpathCreatorLocation.toString(), containsString("Malmo"));
assertThat(jsonpathCreatorLocation.toString(), containsString("San Francisco"));
assertThat(jsonpathCreatorLocation.toString(), containsString("Helsingborg"));
```

### 4.2。谓词

现在我们有了基础知识，让我们定义一个新的 JSON 示例，并说明如何创建和使用谓词:

```java
{
    "book": 
    [
        {
            "title": "Beginning JSON",
            "author": "Ben Smith",
            "price": 49.99
        },

        {
            "title": "JSON at Work",
            "author": "Tom Marrs",
            "price": 29.99
        },

        {
            "title": "Learn JSON in a DAY",
            "author": "Acodemy",
            "price": 8.99
        },

        {
            "title": "JSON: Questions and Answers",
            "author": "George Duckett",
            "price": 6.00
        }
    ],

    "price range": 
    {
        "cheap": 10.00,
        "medium": 20.00
    }
}
```

谓词为过滤器确定真或假的输入值，以便将返回的列表缩小到仅匹配的对象或数组。我们可以很容易地将一个`Predicate`集成到一个`Filter`中，方法是将它作为静态工厂方法的参数。然后可以使用那个*过滤器*从 JSON 字符串中读出请求的内容:

```java
Filter expensiveFilter = Filter.filter(Criteria.where("price").gt(20.00));
List<Map<String, Object>> expensive = JsonPath.parse(jsonDataSourceString)
  .read("$['book'][?]", expensiveFilter);
predicateUsageAssertionHelper(expensive);
```

我们还可以定义我们定制的`Predicate`，并将其用作`read` API 的参数:

```java
Predicate expensivePredicate = new Predicate() {
    public boolean apply(PredicateContext context) {
        String value = context.item(Map.class).get("price").toString();
        return Float.valueOf(value) > 20.00;
    }
};
List<Map<String, Object>> expensive = JsonPath.parse(jsonDataSourceString)
  .read("$['book'][?]", expensivePredicate);
predicateUsageAssertionHelper(expensive);
```

最后，谓词可以直接应用于`read` API，而无需创建任何对象，这被称为内联谓词:

```java
List<Map<String, Object>> expensive = JsonPath.parse(jsonDataSourceString)
  .read("$['book'][?(@['price'] > $['price range']['medium'])]");
predicateUsageAssertionHelper(expensive);
```

上面的所有三个`Predicate` 例子都在下面的断言助手方法的帮助下得到验证:

```java
private void predicateUsageAssertionHelper(List<?> predicate) {
    assertThat(predicate.toString(), containsString("Beginning JSON"));
    assertThat(predicate.toString(), containsString("JSON at Work"));
    assertThat(predicate.toString(), not(containsString("Learn JSON in a DAY")));
    assertThat(predicate.toString(), not(containsString("JSON: Questions and Answers")));
}
```

## 5。配置

### 5.1。选项

Jayway JsonPath 提供了几个选项来调整默认配置:

*   `Option.AS_PATH_LIST`返回评估命中的路径，而不是它们的值。
*   `Option.DEFAULT_PATH_LEAF_TO_NULL`对于缺失的叶子返回 null。
*   即使路径是确定的，也返回一个列表。
*   确保没有异常从路径评估中传播。
*   `Option.REQUIRE_PROPERTIES`当评估不确定的路径时，需要在路径中定义的属性。

以下是如何从头开始应用`Option`:

```java
Configuration configuration = Configuration.builder().options(Option.<OPTION>).build();
```

以及如何将其添加到现有配置中:

```java
Configuration newConfiguration = configuration.addOptions(Option.<OPTION>);
```

### 5.2 版。人口普查〔t1〕

在`Option`的帮助下，JsonPath 的默认配置对于大多数任务来说应该足够了。然而，具有更复杂用例的用户可以根据他们的特定需求修改 JsonPath 的行为——使用三种不同的 SPI:

*   SPI 让我们改变 JsonPath 解析和处理 JSON 文档的方式。
*   SPI 允许定制节点值和返回对象类型之间的绑定。
*   `CacheProvider` SPI 调整路径缓存的方式，这有助于提高性能。

## 6。用例示例

我们现在对 JsonPath 的功能有了很好的理解。我们来看一个例子。

本节演示了如何处理从 web 服务返回的 JSON 数据。

假设我们有一个电影信息服务，它返回以下结构:

```java
[
    {
        "id": 1,
        "title": "Casino Royale",
        "director": "Martin Campbell",
        "starring": 
        [
            "Daniel Craig",
            "Eva Green"
        ],
        "desc": "Twenty-first James Bond movie",
        "release date": 1163466000000,
        "box office": 594275385
    },

    {
        "id": 2,
        "title": "Quantum of Solace",
        "director": "Marc Forster",
        "starring": 
        [
            "Daniel Craig",
            "Olga Kurylenko"
        ],
        "desc": "Twenty-second James Bond movie",
        "release date": 1225242000000,
        "box office": 591692078
    },

    {
        "id": 3,
        "title": "Skyfall",
        "director": "Sam Mendes",
        "starring": 
        [
            "Daniel Craig",
            "Naomie Harris"
        ],
        "desc": "Twenty-third James Bond movie",
        "release date": 1350954000000,
        "box office": 1110526981
    },

    {
        "id": 4,
        "title": "Spectre",
        "director": "Sam Mendes",
        "starring": 
        [
            "Daniel Craig",
            "Lea Seydoux"
        ],
        "desc": "Twenty-fourth James Bond movie",
        "release date": 1445821200000,
        "box office": 879376275
    }
]
```

其中,`release date`字段的值是自纪元以来的毫秒数，而`box office`是以美元为单位的电影院中电影的收入。

我们将处理与 GET 请求相关的五种不同的工作场景，假设上述 JSON 层次结构已经被提取并存储在名为`jsonString`的`String`变量中。

### 6.1。获取给定 id的对象数据

在这个用例中，客户机通过向服务器提供电影的确切`id`来请求特定电影的详细信息。这个例子演示了服务器如何在返回客户端之前查找请求的数据。

假设我们需要找到一条`id`等于 2 的记录。

第一步是选择正确的数据对象:

```java
Object dataObject = JsonPath.parse(jsonString).read("$[?(@.id == 2)]");
String dataString = dataObject.toString();
```

JUnit `Assert` API 确认了几个字段的存在:

```java
assertThat(dataString, containsString("2"));
assertThat(dataString, containsString("Quantum of Solace"));
assertThat(dataString, containsString("Twenty-second James Bond movie"));
```

### 6.2。获得由主演的电影片名

假设我们想寻找一部由名为`Eva Green`的女演员主演的电影。服务器需要返回在`starring`数组中包含`Eva Green`的电影的`title`。

随后的测试将说明如何做到这一点，并验证返回的结果:

```java
@Test
public void givenStarring_whenRequestingMovieTitle_thenSucceed() {
    List<Map<String, Object>> dataList = JsonPath.parse(jsonString)
      .read("$[?('Eva Green' in @['starring'])]");
    String title = (String) dataList.get(0).get("title");

    assertEquals("Casino Royale", title);
}
```

### 6.3。总收入的计算

这个场景使用一个名为`length()`的 JsonPath 函数来计算电影记录的数量，以便计算所有电影的总收入。

让我们看看实现和测试:

```java
@Test
public void givenCompleteStructure_whenCalculatingTotalRevenue_thenSucceed() {
    DocumentContext context = JsonPath.parse(jsonString);
    int length = context.read("$.length()");
    long revenue = 0;
    for (int i = 0; i < length; i++) {
        revenue += context.read("$[" + i + "]['box office']", Long.class);
    }

    assertEquals(594275385L + 591692078L + 1110526981L + 879376275L, revenue);
}
```

### 6.4。收入最高的电影

这个用例举例说明了使用非默认的 JsonPath 配置选项，即`Option.AS_PATH_LIST`，来找出收入最高的电影。

首先，我们需要提取所有电影票房收入的列表。然后我们把它转换成一个数组进行排序:

```java
DocumentContext context = JsonPath.parse(jsonString);
List<Object> revenueList = context.read("$[*]['box office']");
Integer[] revenueArray = revenueList.toArray(new Integer[0]);
Arrays.sort(revenueArray);
```

我们可以很容易地从`revenueArray`排序数组中选取`highestRevenue`变量，然后用它来计算出收入最高的电影记录的路径:

```java
int highestRevenue = revenueArray[revenueArray.length - 1];
Configuration pathConfiguration = 
  Configuration.builder().options(Option.AS_PATH_LIST).build();
List<String> pathList = JsonPath.using(pathConfiguration).parse(jsonString)
  .read("$[?(@['box office'] == " + highestRevenue + ")]");
```

基于计算出的路径，我们将确定并返回相应电影的`title`:

```java
Map<String, String> dataRecord = context.read(pathList.get(0));
String title = dataRecord.get("title");
```

整个过程由`Assert` API 验证:

```java
assertEquals("Skyfall", title);
```

### 6.5。导演最新电影

这个例子将说明如何找出由名为`Sam Mendes`的导演执导的最后一部电影。

首先，我们创建一个由`Sam Mendes`导演的所有电影的列表:

```java
DocumentContext context = JsonPath.parse(jsonString);
List<Map<String, Object>> dataList = context.read("$[?(@.director == 'Sam Mendes')]");
```

然后，我们使用该列表提取发布日期。这些日期将存储在一个数组中，然后进行排序:

```java
List<Object> dateList = new ArrayList<>();
for (Map<String, Object> item : dataList) {
    Object date = item.get("release date");
    dateList.add(date);
}
Long[] dateArray = dateList.toArray(new Long[0]);
Arrays.sort(dateArray);
```

我们使用`lastestTime`变量(排序数组的最后一个元素)结合`director`字段的值来确定所请求电影的`title`:

```java
long latestTime = dateArray[dateArray.length - 1];
List<Map<String, Object>> finalDataList = context.read("$[?(@['director'] 
  == 'Sam Mendes' && @['release date'] == " + latestTime + ")]");
String title = (String) finalDataList.get(0).get("title");
```

下面的断言证明了一切都按预期工作:

```java
assertEquals("Spectre", title);
```

## 7。结论

本文介绍了 Jayway JsonPath 的基本特性——遍历和解析 JSON 文档的强大工具。

尽管 JsonPath 有一些缺点，比如缺少到达父节点或兄弟节点的操作符，但它在很多场景中非常有用。

所有这些例子和代码片段的实现都可以在 GitHub 上找到[。](https://web.archive.org/web/20220901190031/https://github.com/eugenp/tutorials/tree/master/json-modules/json-path)