# jest–elastic search Java 客户端

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/elasticsearch-jest>

## 1.介绍

任何使用过 [Elasticsearch](/web/20220524005021/https://www.baeldung.com/elasticsearch-java) 的人都知道，使用他们的 [RESTful 搜索 API](/web/20220524005021/https://www.baeldung.com/elasticsearch-full-text-search-rest-api) 构建查询可能是乏味且容易出错的。

在本教程中，我们将看看 [Jest](https://web.archive.org/web/20220524005021/https://github.com/searchbox-io/Jest) ，一个用于 Elasticsearch 的 HTTP Java 客户端。尽管 Elasticsearch 提供了自己的原生 Java 客户端， **Jest 提供了更流畅的 API 和更简单的界面来与**协同工作。

## 2.Maven 依赖性

我们需要做的第一件事是将 [Jest 库](https://web.archive.org/web/20220524005021/https://search.maven.org/search?q=g:io.searchbox%20a:jest)导入我们的 POM:

```java
<dependency>
    <groupId>io.searchbox</groupId>
    <artifactId>jest</artifactId>
    <version>6.3.1</version>
</dependency>
```

Jest 的版本遵循主要的 Elasticsearch 产品的版本。这有助于确保客户端和服务器之间的兼容性。

通过包含 Jest 依赖项，相应的 [Elasticsearch 库](https://web.archive.org/web/20220524005021/https://search.maven.org/search?q=g:org.elasticsearch%20a:elasticsearch)将作为传递依赖项被包含。

## 3.使用 Jest 客户端

在这一节中，我们将看看如何使用 Jest 客户端来执行 Elasticsearch 的常见任务。

要使用 Jest 客户端，我们只需使用 `JestClientFactory`创建一个`JestClient` 对象。**创建这些对象的成本很高，而且是线程安全的**，所以我们将创建一个可以在整个应用程序中共享的单例实例:

```java
public JestClient jestClient() {
    JestClientFactory factory = new JestClientFactory();
    factory.setHttpClientConfig(
      new HttpClientConfig.Builder("http://localhost:9200")
        .multiThreaded(true)
        .defaultMaxTotalConnectionPerRoute(2)
        .maxTotalConnection(10)
        .build());
    return factory.getObject();
}
```

这将创建一个 Jest 客户端，连接到本地运行的 Elasticsearch 客户端。虽然这个连接示例很简单， **Jest 也完全支持代理、SSL、认证，甚至节点发现**。

`JestClient`类是通用的，只有少数几个公共方法。**我们将使用的主要接口是`execute`** ，它采用了`Action`接口的一个实例。Jest 客户端提供了几个构建器类来帮助创建与 Elasticsearch 交互的不同操作。

**所有 Jest 调用的结果都是`JestResult`** 的一个实例。我们可以通过调用`isSucceeded`来检查是否成功。对于不成功的行动，我们可以致电`getErrorMessage`了解更多详情:

```java
JestResult jestResult = jestClient.execute(new Delete.Builder("1").index("employees").build());

if (jestResult.isSucceeded()) {
    System.out.println("Success!");
}
else {
    System.out.println("Error: " + jestResult.getErrorMessage());
}
```

### 3.1.管理指数

为了检查索引是否存在，我们使用了`IndicesExists`动作:

```java
JestResult result = jestClient.execute(new IndicesExists.Builder("employees").build()) 
```

为了创建一个索引，我们使用了`CreateIndex`动作:

```java
jestClient.execute(new CreateIndex.Builder("employees").build());
```

这将使用默认设置创建一个索引。我们可以在索引创建期间覆盖特定设置:

```java
Map<String, Object> settings = new HashMap<>();
settings.put("number_of_shards", 11);
settings.put("number_of_replicas", 2);
jestClient.execute(new CreateIndex.Builder("employees").settings(settings).build());
```

使用`ModifyAliases`动作创建或更改别名也很简单:

```java
jestClient.execute(new ModifyAliases.Builder(
  new AddAliasMapping.Builder("employees", "e").build()).build());
jestClient.execute(new ModifyAliases.Builder(
  new RemoveAliasMapping.Builder("employees", "e").build()).build());
```

### 3.2.创建文档

Jest 客户端使用`Index` action 类可以很容易地索引或者创建新文档。**elastic search 中的文档只是 JSON 数据**，有多种方式将 JSON 数据传递给 Jest 客户端进行索引。

对于这个例子，让我们使用一个假想的雇员文档:

```java
{
    "name": "Michael Pratt",
    "title": "Java Developer",
    "skills": ["java", "spring", "elasticsearch"],
    "yearsOfService": 2
}
```

表示 JSON 文档的第一种方式是使用 Java `String`。虽然我们可以手动创建 JSON 字符串，但我们必须注意正确的格式、大括号和转义引号字符。

因此，使用一个 JSON 库，比如[杰克森](/web/20220524005021/https://www.baeldung.com/jackson-object-mapper-tutorial)来构建我们的 JSON 结构，然后转换成一个`String`:

```java
ObjectMapper mapper = new ObjectMapper();
JsonNode employeeJsonNode = mapper.createObjectNode()
  .put("name", "Michael Pratt")
  .put("title", "Java Developer")
  .put("yearsOfService", 2)
  .set("skills", mapper.createArrayNode()
    .add("java")
    .add("spring")
    .add("elasticsearch"));
jestClient.execute(new Index.Builder(employeeJsonNode.toString()).index("employees").build());
```

我们还可以使用 Java `Map`来表示 JSON 数据，并将其传递给`Index`动作:

```java
Map<String, Object> employeeHashMap = new LinkedHashMap<>();
employeeHashMap.put("name", "Michael Pratt");
employeeHashMap.put("title", "Java Developer");
employeeHashMap.put("yearsOfService", 2);
employeeHashMap.put("skills", Arrays.asList("java", "spring", "elasticsearch"));
jestClient.execute(new Index.Builder(employeeHashMap).index("employees").build());
```

最后，Jest 客户机可以接受任何表示要索引的文档的 POJO。假设我们有一个`Employee`类:

```java
public class Employee {
    String name;
    String title;
    List<String> skills;
    int yearsOfService;
}
```

我们可以将这个类的一个实例直接传递给`Index`构建器:

```java
Employee employee = new Employee();
employee.setName("Michael Pratt");
employee.setTitle("Java Developer");
employee.setYearsOfService(2);
employee.setSkills(Arrays.asList("java", "spring", "elasticsearch"));
jestClient.execute(new Index.Builder(employee).index("employees").build());
```

### 3.3.阅读文档

使用 Jest 客户端从 Elasticsearch 访问文档有两种主要方式。首先，如果我们知道文档 ID，我们可以使用`Get`动作直接访问它:

```java
jestClient.execute(new Get.Builder("employees", "17").build());
```

要访问返回的文档，我们必须调用各种`getSource`方法之一。我们可以获得原始 JSON 格式的结果，也可以将其反序列化回 DTO:

```java
Employee getResult = jestClient.execute(new Get.Builder("employees", "1").build())
    .getSourceAsObject(Employee.class);
```

访问文档的另一种方式是使用搜索查询，这是用`Search`动作开玩笑实现的。

**Jest 客户端支持全弹性搜索查询 DSL** 。就像索引操作一样，查询被表示为 JSON 文档，并且有多种方式来执行搜索。

首先，我们可以传递一个表示搜索查询的 JSON 字符串。提醒一下，我们必须注意确保字符串被正确转义并且是有效的 JSON:

```java
String search = "{" +
  "  \"query\": {" +
  "    \"bool\": {" +
  "      \"must\": [" +
  "        { \"match\": { \"name\":   \"Michael Pratt\" }}" +
  "      ]" +
  "    }" +
  "  }" +
  "}";
jestClient.execute(new Search.Builder(search).build());
```

与上面的`Index`动作一样，我们可以使用像 Jackson 这样的库来构建我们的 JSON 查询字符串。

此外，我们还可以使用原生的 Elasticsearch 查询操作 API。这样做的一个缺点是，我们的应用程序必须依赖于完整的 [Elasticsearch 库](https://web.archive.org/web/20220524005021/https://search.maven.org/search?q=g:org.elasticsearch%20a:elasticsearch)。

通过`Search`动作，可以使用`getSource`方法访问匹配的文档。然而， **Jest 也提供了`Hit`类，它包装匹配的文档并提供关于结果的元数据**。使用`Hit`类，我们可以访问每个结果的附加元数据:分数、路由和解释结果，仅举几个例子:

```java
List<SearchResult.Hit<Employee, Void>> searchResults = 
  jestClient.execute(new Search.Builder(search).build())
    .getHits(Employee.class);
searchResults.forEach(hit -> {
    System.out.println(String.format("Document %s has score %s", hit.id, hit.score));
});
```

### 3.4.更新文档

Jest 提供了一个简单的`Update`动作来更新文档:

```java
employee.setYearOfService(3);
jestClient.execute(new Update.Builder(employee).index("employees").id("1").build());
```

它接受与我们前面看到的`Index`动作相同的 JSON 表示，使得在两个操作之间共享代码变得容易。

### 3.5.删除文档

使用`Delete`动作从索引中删除文档。它只需要一个索引名和文档 ID:

```java
jestClient.execute(new Delete.Builder("17")
  .index("employees")
  .build());
```

## 4.批量操作

Jest 客户端也支持批量操作。这意味着我们可以通过同时发送多个操作来节省时间和带宽。

使用`Bulk`动作，我们可以将任意数量的请求组合成一个调用。我们甚至可以将不同类型的请求组合在一起:

```java
jestClient.execute(new Bulk.Builder()
  .defaultIndex("employees")
  .addAction(new Index.Builder(employeeObject1).build())
  .addAction(new Index.Builder(employeeObject2).build())
  .addAction(new Delete.Builder("17").build())
  .build());
```

## 5.异步操作

**Jest 客户端还支持异步操作**，这意味着我们可以使用非阻塞 I/O 执行上述任何操作。

要异步调用操作，只需使用客户端的`executeAsync`方法:

```java
jestClient.executeAsync(
  new Index.Builder(employeeObject1).build(),
  new JestResultHandler<JestResult>() {
      @Override public void completed(JestResult result) {
          // handle result
      }
      @Override public void failed(Exception ex) {
          // handle exception
      }
  });
```

注意，除了动作(在本例中是索引)，异步流还需要一个`JestResultHandler.`Jest 客户机将在动作完成时调用这个对象。该接口有两个方法——`completed`和`failed`——分别允许处理操作的成功或失败。

## 6.结论

在本教程中，我们简要介绍了 Jest 客户端，这是一个用于 Elasticsearch 的 RESTful Java 客户端。

虽然我们只介绍了它的一小部分功能，但是很明显 Jest 是一个健壮的 Elasticsearch 客户端。其流畅的构建器类和 RESTful 接口使其易于学习，其对 Elasticsearch 接口的完全支持使其成为 native client 的有力替代。

和往常一样，教程中的所有代码示例都在 GitHub 上的[中。](https://web.archive.org/web/20220524005021/https://github.com/eugenp/tutorials/tree/master/persistence-modules/elasticsearch)